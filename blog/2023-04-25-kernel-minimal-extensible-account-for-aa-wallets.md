---
slug: kernel-minimal-extensible-account-for-aa-wallets
title: Introducing Kernel — Minimal & Extensible Smart Contract Account for ERC-4337 Wallets
authors: derek
---

# Introducing *Kernel* — Minimal & Extensible Smart Contract Account for ERC-4337 Wallets

With the launch of ERC-4337, we are seeing tremendous excitement from Web3 developers to build the next generation of crypto wallets using account abstraction.

Whereas traditional wallets like MetaMask are powered by *externally owned accounts* (EOA), account abstraction wallets are powered by *smart contract accounts* (CA).  These wallets will be able to sponsor gas for users, batch transactions, support automatic payments (subscriptions)… overall enabling a Web3 experience hitherto unimaginable.

While some wallet developers want to control the entire tech stack end-to-end, most wallet developers we’ve met would rather focus on the end-user experience, by building *product features* such as DeFi integrations, cross-chain transfers, etc.  Actually coding a smart contract account in Solidity, and making sure it’s compatible with ERC-4337 and supports all the essential functionalities such as validating signatures (ERC-1271) and bundling transactions, is time-consuming and hard to get right for most wallet developers.

# Introducing *Kernel*, a minimal smart contract account designed to be extended

Seeing this need, ZeroDev has developed an *account abstraction wallet kernel*.  The term *kernel* comes from the lingo of operating systems.  The Linux kernel, for example, is used by a wide range of operating systems such as Android, Raspberry Pi, etc.  The reason why the Linux kernel exists is so that different operations systems do not have to build the basic OS functionalities (e.g. file systems and networking) from scratch.  Rather, operating systems builders can focus on building the OS *features* that make the OS unique, whether it’s great UI, integration with popular apps, or whatnot.

Similarly, the goal of the ZeroDev Kernel is so wallet developers do not have to build the basic wallet functionalities from scratch.  Specifically, the kernel includes the following basic features that we consider essential to any AA wallet:

- Compatibility with ERC-4337
- Validating signatures with ERC-1271
- Batching transactions
- Delegating calls

However, we recognize that wallet developers may also want to build additional on-chain functionalities, and oftentimes these needs cannot even be anticipated when the wallet was first built.  For example, you might decide, after launching your wallet, that a lot of your users are using the wallet with Web3 games, so you’d like to support session keys (temporary keys with restricted permissions).  It would be very painful if you had to ask your users to *upgrade* their on-chain smart contract accounts in order to support the new use case.

Therefore, we have built a *plugin framework* for developers to add functionalities on top of the kernel, without needing to upgrade the account itself.

# Kernel *plugins* — ERC-4337-native Solidity modules that modify validation logic

So what exactly is a plugin?  It’s a Solidity contract that the kernel can *delegate* to to modify the validation logic of the account.

That was a mouthful, so let’s look at an example.  Let’s say, as a trivial example, that you want to allow someone to manage your USDC and DAI balances.  You can create a contract like this (in pseudo-solidity):

```solidity
contract StableCoinPlugin {
    function validateUserOp(UserOp op) returns bool {
        return op.to == USDC_CONTRACT || op.to == DAI_CONTRACT;
    }
}
```

Essentially, this plugin authorizes transactions that interact with the USDC and DAI contracts.

Once this plugin has been deployed, you can sign an off-chain message *authorizing* this plugin.  You can then share the signed message with the person or app that wants to manage your USDC/DAI.  They will be able to send transactions *on your behalf*, but only if those transactions interact with the USDC and DAI contracts.

We will be diving deep into the plugin framework in a future blog post.

# Kernel makes it easy for users to *migrate* between wallets

Most smart contract wallets are deployed as [proxies](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies), since proxies are a lot cheaper to deploy than the underlying contract, but also because proxies allow smart contract wallets to be upgraded.

One consequence of smart contract wallets being upgradable is that users are free to switch between *account implementations*.  For example, a user might be onboarded to Web3 with a simple in-game wallet.  The user might have accumulated some valuable NFTs in the game, and instead of transferring the NFTs to a real wallet, the user can simply *upgrade* its wallet implementation to a real wallet, while keeping the same address.

While the idea is very appealing, in practice migrating between wallets can be hard and unsafe.  This is due to the issue with storage layouts.  Consider these two wallet implementations (in pseudo-solidity):

```solidity
contract WalletA {
  address owner;
  uint256 nonce;
  // ...more code
}

contract WalletB {
  uint256 nonce;
  address owner;
  // ...more code
}
```

If a user starts with WalletA, then migrates to WalletB, its storage will be corrupted because the original `owner` now sits on the slot of the `nonce`, and vice versa.  So in reality, before the migration, the user would need to wipe its own storage, which is tricky and hard to get right.

ZeroDev Kernel is designed with migration in mind.  To that end, Kernel uses *[diamond storage](https://dev.to/mudgen/how-diamond-storage-works-90e)* — a technique that ensures that one wallet’s data storage won’t collide with another wallet’s.  Therefore, it’s perfectly safe to migrate either *from* or *to* a wallet built on the kernel.

Note that Diamond storage is different than Diamond proxies (ERC-2535), which is a much more ambitious and complex design. Here, we simply ensure that storage layouts between different wallets don’t collide.

The fact that ZeroDev is perfectly migrate-able makes it perfect for building *onboarding wallets*.  Your users can onboard with ZeroDev and rest assured that when they find an AA wallet they like, they can seamlessly migrate to that wallet without having to change address.  This is a 100x improvement over the status quo of “exporting seed phrases,” which involves trusting some centralized server to NOT store a copy of your seed phrase and hoping that the seed phrase isn’t leaked somewhere along the process.

# Kernel vs Safe

During beta testing, the most common question we got was understandably this: why are you building a new smart contract account when you can just use Gnosis Safe?

In fact, ZeroDev *started* with Safe.  We contributed heavily to the [reference Safe 4337 implementation](https://github.com/eth-infinitism/account-abstraction/tree/develop/contracts/samples/gnosis), and used it all throughout our beta.  However, we ran into some major issues with Safe that blocked us from achieving our product objectives:

- Safe is complex.  By all accounts, Safe is one of the largest smart contract codebases ever.  Many features have been added over the years to satisfy a variety of organizational needs, and truthfully most of them are completely irrelevant for the kind of single-user AA use case that ZeroDev sets out to address.
- Safe is inefficient as an ERC-4337 wallet.  As the reference implementation shows, the only way to make Safe compatible with ERC-4337 was to do it through Safe’s “fallbacks” and “modules” mechanisms.  This leads to a large amount of context switching and therefore high gas costs for even the most simple operations.

Ultimately, Safe was designed for a different use case — organizational multisig.  This is about as far from the single-user, single-sig use case that ZeroDev is designed for.  Therefore, we ultimate decided to bite the bullet and implement a smart contract account optimized for retail AA users.

With Kernel, we now have a much simpler, much more efficient, and highly extensible smart contract account, and our users couldn’t be happier.

# Kernel, ERC-6900, and Interoperability

One main goal with Kernel was to foster a thriving plugin ecosystem, but some may be concerned that a plugin developed for Kernel will *only* work with Kernel.

As if anticipating that concern, our friends at Alchemy recently drafted [ERC-6900](https://github.com/ethereum/EIPs/pull/6900/files), titled “Modular Smart Contract Accounts and Plugins.”  The goal of the EIP is to define a common interface between smart contract accounts (e.g. Kernel) and plugins.

We are very happy to see this development and we will be contributing to the ERC.  We are also glad to see that we’ve made many of the same design decisions that the ERC authors did.  As of today, Kernel is the *closest* thing to an implementation of ERC-6900 that we know of, and we will be making Kernel fully compatible with ERC-6900 once it’s finalized.  That way, plugin and wallet developers building on ZeroDev can rest assured that they are building on top of an open standard and will enjoy great interoperability for their products.

# Start building on Kernel now

Today, we are excited to announce that [Kernel](https://github.com/zerodevapp/zerodev-wallet-kernel) has been open-sourced and audited, and it’s now available for anyone to use.  Being an open-source project, Kernel is free for anyone to fork and extend.  You can use [ZeroDev](https://docs.zerodev.app) to quickly spin up Kernel-based AA wallets, and then extend the wallet’s functionalities using our [plugin framework](https://docs.zerodev.app/extend-wallets/build-a-plugin).  We are already building some of the most commonly asked-for plugins including [session keys](https://docs.zerodev.app/use-wallets/use-session-keys), which we will dive into in a future blog post.

We are confident that Kernel will dramatically lower the barrier for building wallets powered by account abstraction.  We can’t wait to see what you build with Kernel!
