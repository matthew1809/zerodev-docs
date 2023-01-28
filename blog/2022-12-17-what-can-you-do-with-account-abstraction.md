---
slug: what-can-you-do-with-account-abstraction
title: What Can You Do with Account Abstraction?
authors: derek
---
As the saying goes, the only constant in Web3 is change.  If you are building a Web3 product, you need to stay on top of technological trends, in order to identify new opportunities to grow and improve your product.

One emerging trend that you might have noticed is “account abstraction” (AA).  While there have been [many](https://www.argent.xyz/blog/wtf-is-account-abstraction/) [great](https://www.argent.xyz/blog/part-2-wtf-is-account-abstraction/) [technical](https://www.argent.xyz/blog/part-3-wtf-is-account-abstraction/) [write-ups](https://eips.ethereum.org/EIPS/eip-4337) [on](https://medium.com/infinitism/erc-4337-account-abstraction-without-ethereum-protocol-changes-d75c9d94dc4a) [account](https://notes.ethereum.org/@vbuterin/account_abstraction_roadmap) [abstraction](https://hackmd.io/@s0lness/BJUb16Yo9), I would like to explain in this blog post the practical benefits of account abstraction, or in other words, how you can leverage account abstraction to improve your app.

### Account abstraction in simple terms
First thing first: what does account abstraction actually mean?

On most blockchains today, transactions can only be sent from an "externally owned account" (EOA), which is a fancy term to refer to wallets like MetaMask that are created from a seed phrase.  In other words, EOAs are the wallets everyone has been using so far.

Account abstraction describes the ability for **smart contracts** to send and verify transactions.  The difference between a smart contract and an EOA is like the difference between a computer and an abacus.  Whereas an abacus can only be used in a specific way, a computer can be programmed to perform arbitrary functions.  Similarly, whereas EOAs work in a pre-defined way, smart contract wallets (SCW) can be programmed.  This opens up exciting opportunities for you as a developer to improve user experience (UX) and implement new product features, which we will detail below.

### Side note on smart contract wallets
Some of you might be wondering: how is this different than smart contract wallets like Gnosis Safe?  Hasn't that been around for a while?

Yes, but the key breakthrough with account abstraction is that SCWs can finally originate transactions.  With Gnosis Safe, for example, the user still has to use an EOA like MetaMask to issue transactions, which are then "routed through" Gnosis Safe.  Think controlling a computer with an abacus.  This introduces a significant barrier to using SCWs, which is why to this date SCWs like Gnosis Safe are only used in high-security enterprise settings, and not in day-to-day Web3 usage.

### Gas sucks; AA fixes it
One of the biggest UX issues with EOA wallets is that users have to pay gas in ETH (or whatever the native token is).  This can be a significant hurdle, especially for new users who are not familiar with cryptocurrencies and may not even own any ETH.

As a classic example, let's say you are doing a "free mint" and you would like your users to mint an NFT.  While minting is free, gas is not.  Someone new to crypto would have to first acquire some ETH, which probably involves a KYC process.  More likely than not, they will simply give up, instead of engaging with your app.

Another example is NFT games, where a user may have won or received some NFTs, but can't do anything with them (such as trading or transferring the NFTs) since they don't have the ETH to pay for gas.

Account abstraction addresses these issues by allowing users to skip gas entirely, if a third party is willing to sponsor gas for them.  In these examples, you (the developer) can enable gas-less experiences for your users by sponsoring their gas.

Account abstraction-enabled wallets can also pay gas in any ERC-20 tokens.  For DeFi applications, it's very common for a user to primarily be investing in ERC-20 tokens such as UNI or USDC.  With an EOA, the user would still have to own a little bit of ETH (which they have to top up every now and then) in order to pay gas for transactions.  With account abstraction, the user can pay gas in the tokens they already own!

### Seed phrases are a nightmare
One of the biggest UX issues with EOA wallets is the need to safe-keep a seed phrase.  This is an incredibly difficult task for most people: seed phrases are difficult to remember, and can be easily lost or stolen.

With smart contract wallets, there are many ways to solve this problem.  One idea is social recovery: a user can authorize a list of their friends or family members to recover their account if they lose access.  It's much easier to remember who your best friends are than to remember 12 random words!

The flexibility of smart contract wallets also means that it can work with MPC, so your users can simply login with a social account (e.g. Google) or email/password.

### Batch transactions for fewer confirmations
Another issue with EOA wallets is that each transaction is verified and executed separately, which means long wait times and high gas fees.  We have all had the terrible experience of trying to do something simple, say swapping one token for another, and yet having to confirm and wait for multiple transactions (e.g. an "approve" into a "swap").

With account abstraction, however, multiple transactions can be batched together into a single transaction.  This significantly reduces the cost and wait time associated with interacting with your app — your users can get things done in one click.

Batching also makes your app safer by ensuring “atomicity” — that a multi-step process either finishes or completely reverts, instead of getting stuck in a “half-completed” state, e.g. “approve” succeeding but “swap” failing.  The lack of atomicity can lead to very tricky bugs in DApps, but it won’t be an issue with AA.

### Build interactive apps with session keys
What if you are building a highly interactive application such as a game, where prompting the user for confirmation would really disrupt their flow?  Enter session keys, which are temporary keys that can be used to send transactions for a limited amount of time, with a limited scope of permissions.

Combining session keys with batching, your app can be sending far fewer transactions, while skipping most approvals, making your UX approach that of traditional Web2 applications.

### Transaction Guards
Sometimes you want to protect your wallets against misuse.  With an AA wallet, you can set up "transaction guards" – smart contracts that check transactions that go through your wallet.

One example is spending limits.  You might want to limit your daily spending to $100, and for anything more than that you require a second signer (e.g. ledger).  This not only helps you rein in your NFT impulse buys, but also is a good way to defend against hackers.  With AA, spending limits can be easily implemented as a transaction guard.

### Delegate assets
When you put crypto into a centralized product, you typically have no visibility into what they are doing with your funds, which can result in disastrous consequences (cough FTX cough).  When you hold crypto in a self-custody wallet, there's perfect visibility, but you need to personally initiate and sign every transaction, which limits the potential of your crypto.

What if there's a third way?  What if you can self-custody your funds, while authorizing a third party to perform *limited* actions on your behalf?

A great example is collateralized loans like Compound and Aave.  When you put down collateral and take out a loan from these protocols, you need to continuously monitor your loan and top up your collateral if necessary, in case the price of your collateral token drops and your collateral gets liquidated.  This is a lot of work and stress.

With account abstraction, you can build applications such that your users can *delegate* certain transactions for you to perform.  If you were building a lending app with account abstraction, for instance, your app could automatically close your user's position when their collateral is in danger — all without them having to give away custody of their assets.  This works because the permissions are enforced by smart contract wallets — the third party (e.g. your app) can only perform the delegated transactions, and nothing else (such as stealing user assets).

### Subscriptions
As a particular use case of delegating assets, it's worth mentioning subscriptions.  With AA, your users can easily authorize you (the app) to pull money from their accounts, but only up to a certain amount, at a given frequency.

### State of account abstraction today
By now, hopefully I have convinced you that account abstraction can vastly improve the UX and functionality of your app.  But is it ready today?  Can you actually build real products on account abstraction right now?

The answer is a resounding yes.  New rollups such as [StarkNet](https://docs.openzeppelin.com/contracts-cairo/0.4.0/accounts) and [zksync](https://v2-docs.zksync.io/dev/developer-guides/aa.html) natively support account abstraction, while [ERC-4337](https://eips.ethereum.org/EIPS/eip-4337) has brought account abstraction to all EVM blockchains, including Ethereum, Polygon, Arbitrum, Optimism, Avalanche, and more.

In a future blog post, I will dive deeper into ERC-4337 and explain how account abstraction actually works.  For now, my team at ZeroDev has created an SDK that you can use today to enable account abstraction in your app.  [Give it a try now](https://zerokit.io/), or [join our new Discord](https://discord.gg/KS9MRaTSjx) and say hi!