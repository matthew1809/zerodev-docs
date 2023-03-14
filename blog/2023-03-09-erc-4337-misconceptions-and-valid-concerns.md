---
slug: erc-4337-misconceptions-and-valid-concerns
title: ERC-4337 — Misconceptions and Valid Concerns
authors: derek
---

At ZeroDev, it’s our job to help devs learn and adopt AA, so naturally we have come across a lot of questions, concerns, and objections.

In this post, I’d like to summarize some common pushbacks against ERC-4337 and AA in general, and I will group them into three categories:

- **Misconceptions**: things that are just not true.
- **Yes and no**: somewhat true, but the reality is nuanced.
- **Valid concerns**: real issues that need to be addressed.

Let’s dive in!

# Misconceptions

### AA is no big deal because SCW has been around for years

Without AA, smart contract wallets like Safe/Argent are not “first class citizens” on the blockchain, meaning that you cannot initiate transactions directly from them.  Rather, you have to do one of the following:

- Call the SCW from a EOA, so you STILL have to own a dumb wallet before you can own a smart one.
- Rely on a centralized relaying service to relay your transactions, which exposes you to risks like censorship, downtime, etc.

With AA however, you can directly send transactions from a SCW, the same way you can directly send transactions from MetaMask.  This makes it possible to use a SCW as your only wallet, which is precisely what the next billion Web3 users will do.

### You still need a EOA to own an AA wallet

This common point of confusion stems from the fact that most AA wallets today are owned by a private key (just like EOA wallets), but it’s misguided because:

- Private key ≠ EOA.  While it’s true that each private key has a corresponding EOA, the key itself is just that — a key that can sign things.  A typical AA wallet will store and safeguard a private key just like MetaMask does, and use the key to sign transactions for the smart contract account.  The corresponding EOA, to the extent that it exists, is only used as a public key for validating signatures.
- Since AA enables transactions to be validated with a smart contract, the validation logic can be arbitrary, so you don’t technically even need a private key.  [Here’s a proof of concept using fingerprints instead of private keys.](https://twitter.com/plusminushalf/status/1631821839466123272)

### We don’t need AA if we have MPC

The best way to think about MPC vs AA is:

- MPC improves the key management experience.
- AA improves the transaction experience.

With MPC, you effectively have a “virtual private key” without ever having to store it somewhere, which is a huge improvement over the status quo of having to write down a 12-word seed phrase.

AA is about what happens when you send a transaction — who pays gas?  What tokens are used to pay for gas?  Who signs the transaction?  All of these can be abstracted away with AA.

As you can see, MPC and AA actually complement each other nicely — MPC saves the user from having to deal with keys, while AA makes transactions smooth.  In fact, it’s precisely by combining MPC with AA that we are able to offer [social AA wallets](https://docs.zerodev.app/create-wallets/social/overview).

# Yes and No

### AA transactions are more expensive

Since AA uses smart contract wallets, each transaction necessarily has some overheads comparing to the equivalent EOA transaction.  There’s also the cost of deploying the smart contract wallet on-chain.

However, multiple factors lower the transaction cost in AA’s favor:

- SCW has the ability to batch transactions, so what normally takes multiple transactions with EOA, may only take one transaction with a SCW.  A classic example is when you interact with a DeFi protocol, where each action typically involves multiple transactions (e.g. approve → swap → deposit).  In AA, all these can be done in one atomic transaction, thus saving gas.
- ERC-4337 supports signature aggregation, so that multiple AA transactions can effectively “share” a signature, thus lowering the cost for each transaction.  [Here are some numbers from Vitalik.](https://twitter.com/VitalikButerin/status/1554983955182809088)
- ERC-4337 does not deploy the smart contract account until the user’s first transaction.  Before then, the account exists “counterfactually” — it has an address even though it’s not really deployed.  So your users can receive assets even without paying any deployment cost.

As a result, whether AA transactions or normal transactions cost more gas actually depend on the workload.  For some applications (notably DeFi), AA transactions might wound up being cheaper!

### AA is not ready for production

There’s no doubt that anyone building something on AA/ERC-4337 today is a trailblazer — there are not many prior examples to look to or patterns to borrow from.  In that sense, building something on AA certainly involves more technical risks than building a classic DApp.

However, everything you need to build a full AA application, notably ERC-4337 itself, is already running in production/mainnet.  **We are at an inflection point where you are either building one of the last non-AA applications, or one of the first AA applications.**  The choice is yours.

### AA is not compatible with existing DApps

Before AA, there was “meta transactions” that could remove gas (or pay gas in ERC20s) by using relayers that submit transactions on users’ behalf.  The main problem, however, was that DApp contracts had to [use a helper function like `_msgSender()`](https://docs.opengsn.org/#recipient-contract-sees-the-original-sender-and-executes-the-original-transaction) instead of the more intuitive `msg.sender` to get the address of the transaction sender.  Needless to say, most DApps did not do that, so the compatibility of meta transactions were severely limited.

AA does not have this problem, however, which makes it compatible with the vast majority of DApps.  Where the compatibility breaks down, however, is when the DApp asks the wallet to sign a message.  It turns out that EOA signatures and smart contract wallet signatures cannot be verified the same way, so there’s a standard [ERC-1271](https://eips.ethereum.org/EIPS/eip-1271) that DApps are supposed to implement to be compatible with smart contract wallets.  [Here’s an incomplete and likely outdated list of DApps grouped by whether they support ERC-1271.](https://eip1271.io/)

If a DApp requires message signing but doesn’t support ERC-1271, then AA indeed won’t work with the DApp.  Fortunately, the space is [completely aligned](https://twitter.com/VitalikButerin/status/1576267880542633984?lang=en) that ERC-1271 needs to be supported, and new DApps being written today typically support ERC-1271 by default if they use libraries like OpenZeppelin.

### ERC-4337 is not real AA.  We should wait for real AA

When someone says that ERC-4337 is not “real” AA, they are typically referring to the fact that ERC-4337 is NOT integrated into the blockchain protocol itself.  In contrast, new networks like zksync and StarkNet have “enshrined” AA as a part of their protocols.

The reason why Ethereum and most other EVM chains have not enshrined AA is two-fold:

- Enshrining AA will be protocol-breaking, and therefore require a hard fork.
- There’s no consensus over the best approach to implement AA, so it’s not even clear what we should be enshrining.

Enshrining AA into the protocol itself also means that **every** EVM chain has to implement this breaking change, which can take a very very long time.  In contrast, since ERC-4337 is implemented as smart contracts, deploying to a new chain is literally a matter of deploying a few smart contracts.  That’s why ERC-4337 is already running on all EVM chains today.

In any case, the distinction between “real AA” and “ERC-4337 AA” matters little to the end users.  From their perspective, their transactions “just work” either way.   Therefore, given the level of community buy-in for ERC-4337, it’s our best hope for achieving AA on EVM blockchains in the near term.

# Valid Concerns

### ERC-4337 is fairly centralized right now

In theory, ERC-4337 is designed such that anyone can spin up relayers (aka “bundlers”), unlike previous relayer networks that are typically run by a single entity.

In practice, however, most bundler implementations except for StackUp are not production-ready, so most ERC-4337 traffic is going through StackUp today.  This is not unlike how [most Ethereum traffic is going through Geth](https://clientdiversity.org/).  Hopefully, this will change as other bundlers go into production.

### ERC-4337 may still change

While ERC-4337 has been deployed on mainnet, it’s not technically finalized.  [The EIP is still in draft status](https://eips.ethereum.org/EIPS/eip-4337), and the core team has acknowledged that the EIP and the smart contracts could still change.

Fortunately, it’s expected that any changes to the EIP and core smart contracts won’t affect the core account interface, so wallets that are compatible with ERC-4337 today will most likely still be compatible with ERC-4337 in the future.

### ERC-4337 has not been formally verified

While ERC-4337 [has been audited](https://blog.openzeppelin.com/eip-4337-ethereum-account-abstraction-incremental-audit/), it has NOT been [formally verified](https://ethereum.org/en/developers/docs/smart-contracts/formal-verification/), so one cannot completely rule out the possibility that there are some critical security issues.

Fortunately, there are teams working on the formal verification of ERC-4337 (with our very own [taek](https://twitter.com/leekt216) being a major contributor).  When ERC-4337 has been formally verified, that’s when we expect to see it finalized.

# The Bottom Line

Anyone building on AA today is a trailblazer taking on technical risks, no doubt about it.  But with risk comes reward — if properly executed, your project will dwarf your competition in terms of usability.

At ZeroDev, we’ve developed an AA framework that dramatically shortens the time — and reduces the risks — for devs to build wallets and DApps on AA.  [Check out ZeroDev](https://docs.zerodev.app/) and start building the future of Web3 today!
