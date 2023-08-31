---
slug: towards-the-most-optimized-aa-wallet
title: Towards the most optimized AA wallet
authors:
    - derek
    - taek
hide_table_of_contents: true
---

At ZeroDev, we are obsessed with smart contract wallets (SCW), as we believe they are [necessary for realizing the full potential of Web3](https://vitalik.eth.limo/general/2023/06/09/three_transitions.html).  One practical roadblock towards a wider adoption of SCW and AA, however, is the fact that SCWs cost gas to deploy.  In contrast, to create an EOA, it costs nothing — the account simply *exists* once you generate a private key.

The deployment cost of SCWs is why AA is unlikely to be widely used on L1s in the near term.  Luckily, more and more traffic are moving towards L2 anyways, but even then we have come across high-volume use cases where the small costs of creating SCWs add up over time.

Beyond deployment, SCWs also necessarily introduce some overhead for each transaction, since the validation logic is implemented with smart contracts (as opposed to being a part of the protocol itself, which is the case for EOAs).  While the overhead is small, they can still add up to a meaningful amount over time.

At the moment, [Kernel](https://github.com/zerodevapp/kernel) is the [most widely deployed SCW for AA](https://twitter.com/0xKofi/status/1686618729247834112), so we consider it a serious responsibility for us to optimize the gas efficiency of Kernel.  But how?

# Optimizing Kernel

When we first measured the gas efficiency of Kernel (v2), we were disappointed to find that it lagged behind some other implementations out there.  The main reason, we quickly realized, was that there was a tension between performance and modularity.  By [supporting plugins in Kernel](https://docs.zerodev.app/extend-wallets/overview), we also sacrificed some gas efficiency since the code that dispatches to plugins necessarily introduces some gas overhead.  That’s why Kernel used more gas than simpler SCW implementations that don’t support plugins.

But we didn’t want to make any excuses, so we went ahead and squeezed more performance out of Kernel.  **We are now happy to share that Kernel v2.1, the latest version, is now the most optimized AA wallets out there!** . This is despite the fact that **we are also the most modular**.  Here are the numbers:

|  | Creation | Native transfer | ERC20 transfer | Total |
| --- | --- | --- | --- | --- |
| SimpleAccount | 410061 | 97690 | 86754 | 594505 |
| Biconomy | 296892 | 100780 | 89577 | 487249 |
| Etherspot | 305769 | 100091 | 89172 | 495032 |
| Kernel v2.0 | 366662 | 106800 | 95877 | 569339 |
| Kernel v2.1 | 291413 | 103240 | 92289 | 486942 |
| Kernel v2.1-lite | 256965 | 97331 | 86121 | 440417 |

So how did we do it?  It would be too long to go over every little optimization we did, so let’s highlight the main ones.

### Optimizing the Proxy

You might know that most SCWs are actually [proxies](https://medium.com/coinmonks/proxy-pattern-and-upgradeable-smart-contracts-45d68d6f15da) — lightweight smart contracts that point to an underlying “implementation contract” which contains the actual SCW logic.  This is for two reasons:

- Proxies are cheaper to deploy, since the proxy contract itself contains minimal logic.
- Proxies are typically upgradable, so the user can switch from one SCW implementation to another.

Since Kernel is also deployed as a proxy, there are two things we can optimize:

- The bytecode size of the proxy itself.
- The “dispatch function” — how the proxy dispatches to the underlying implementation contract.

While Kernel originally used OpenZeppelin’s standard ERC-1967 proxy contract (and so did most other SCWs we looked at), we switched to the proxy offered by [Solady](https://github.com/Vectorized/solady), an audited set of hyper-optimized contracts.  That brought us 118 less gas per UserOp and 82 less bytes per deployment.

However, we realized that we could further optimize on Solady because Solady implements an “admin” feature that allows the user to upgrade the implementation contract for the proxy.  However, in the context of Kernel, this is actually unnecessary, because Kernel itself already knows how to handle proxy upgrades.  Therefore, we were able to remove the admin feature, which removed another SSTORE that costs around 20000 gas on deployment!

### Optimizing ECDSA Signature Validation

Kernel, being a modular wallet, can handle different kinds of signature schemes, but the most commonly used is still ECDSA which replicates the signing behavior of EOA wallets.  Therefore, optimizing the ECDSA verification flow can be very impactful.

The first thing we did was to leverage Solady’s ECDSA recovery procedure, which saves 282 gas comparing to OpenZeppelin.  But we realized that there’s a much bigger gain — again, since Kernel is so modular, even ECDSA verification is implemented as a module (that way, it can be easily swapped for another signature scheme such as multisig or RSA).  However, by far the majority of ZeroDev users use ECDSA for signatures, so they are paying for the plugin dispatch cost for no reason.

That’s why we built Kernel Lite — a version of Kernel that hardcodes the ECDSA validation logic.  If you look at the benchmark numbers again, you will see that Kernel Lite has by far the best numbers.  We are currently auditing Kernel Lite and will be releasing it to our SDK soon.

# Open-sourcing the Benchmark

As much as we love Kernel, we want all SCWs to improve in gas efficiency so users can enjoy cheaper transactions no matter which SCW they use.  Therefore, we are happy to [open-source our benchmark](https://github.com/zerodevapp/aa-benchmark) in the hopes that it will help other SCW implementations improve their performance.

We have taken great care to make the benchmark easy to use.  To benchmark a SCW implementation, you just need to create a test class that inherits from `TestBase.sol` and implement a few functions.  If you do so, please open a PR so we can add your SCW to the list.  Together, let’s make SCWs more efficient and save gas for everyone!

----------------------

*[ZeroDev is hiring](https://www.notion.so/Senior-Engineer-ZeroDev-729ff99d05854a93924ce6414bf08951?pvs=21)!  If what we wrote here sounds interesting to you, drop us a note :)*