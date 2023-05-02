---
slug: session-keys-are-the-jwts-of-web3
title: Session Keys are the JWTs of Web3
authors: derek
---

Web3 UX today faces many challenges.  High gas costs, long transaction times, and difficulties managing seed phrases are some of the most common issues that many projects, including ZeroDev, strive to fix.

However, there’s one problem that lies at the heart of why Web3 just “doesn’t feel right” to regular users, and yet is rarely discussed.  The problem is that **authorization is broken on Web3**.

# Authentication vs Authorization

Before we delve deeper, let's clarify the difference between authentication and authorization.

- Authentication is the process of proving *who you are*.  For instance, if you arrive at an NFT drop that you've been whitelisted for, how do you prove that you have indeed been whitelisted?  Typically, the NFT drop will request you to sign a message.  By cryptographically signing this message, you are *authenticating* to prove ownership of the whitelisted wallet.

- Authorization, on the other hand, is proving *what you can do*.  When you swap tokens on Uniswap, for example, it asks for your "approval" of the tokens you are swapping.  By doing so, you are *authorizing* Uniswap to swap tokens on your behalf.

# Authorization on Web2

In Web2, authorization is usually managed with JSON Web Tokens (JWTs), primarily in the context of OAuth.

Consider when we log into Zoom using Google.  An OAuth sequence prompts you to grant certain permissions, such as creating calendar events.  Once approved, Google generates a JWT containing these specific authorizations.

The JWT, bearing Google's digital signature for verification, then allows Zoom to request corresponding services from Google's API on your behalf.  As a result, Zoom can automate actions for you like calendar event creation, but it can't do anything else, like accessing your emails.

This process is so commonplace that people seldom pause to think about it.  However, when newcomers to Web3 start using DApps, they quickly realize a daunting issue — authorization in Web3 is fundamentally flawed.

# Authorization is broken on Web3

The central idea of the JWT experience is that on Web2, it's possible to *authorize*
 a third-party app (e.g., Zoom) to interact with a service (e.g., Google) on your behalf.

In Web3, however, there’s no *common standard* for authorization.  As a result, each application (or ERC) has to implement its own method for authorization.

For example, for ERC20 tokens, the `approve(spender, amount)` function authorizes a `spender` to spend up to `amount` of your tokens.  On the other hand, ERC721 tokens use the `setApprovalForAll(operator, approved)` function to authorize an `operator` to transfer all your NFTs in this collection.

More importantly, handling authorization at the contract level is deeply limiting.  For example, in the case of an NFT (ERC721) contract, what if you want to authorize a third party to mint NFTs for you too?  That would be helpful if you wanna set up a bot that mints when an NFT collection is dropped.  Since the `setApprovalForAll` function only concerns with transferring NFTs, however, you are out of luck.

What we need is a Web3 equivalent of JWTs — a universal standard for authorizing third parties to perform actions on your behalf.  This standard should be flexible and widely interoperable, enabling DApps to "speak" this authorization language with minimal modifications.

However, expecting all contracts to conform to the same authorization standard is a significant challenge.  As already mentioned, ERC20 and ERC721 contracts handle authorizations differently, and most other contracts don't handle authorizations at all.

# Use the wallet, duh

Turns out the best way to make authorization works for *all* contracts it to not worry about contracts at all — rather, we do it with the *wallet*.

After all, the most obvious way to authorize someone to do something for you is to, well, give them your seed phrase.  That way, they can interact with *any* contract on your behalf.  Problem solved?

Of course, we all know that’s a terrible idea, since there’s no limit to what the third party can do with your seed phrase.  This is the equivalent of giving someone key to your house when they ask to use your bathroom — it’s overkill and unsafe.

So what if there was a way to give someone access to your wallet, but in such a way that they could only send a *limited set* of transactions for you?

# Session Keys are the JWTs of Web3

Enter session keys — a feature of ZeroDev AA wallets wherein you can create keys that are scoped to only certain transactions, with an expiration time.

**Session keys are the JWTs of Web3**.  Much like JWTs, session keys are cryptographically signed — in this case, with your master key.  Like JWTs, session keys encode "scopes" within themselves that specify the actions they can perform.  Also, both JWTs and session keys can be created with an expiration time to limit the consequences of keys being leaked.

While JWTs and session keys share many similarities, session keys are fundamentally more powerful, because they are *programmable*, whereas JWTs are defined by a standard that by definition doesn’t change.  In that sense, session keys can be thought of as programmable JWTs.

The table below summarizes how JWTs and session keys compare across key dimensions.

| Feature | JWTs (Web2) | Session Keys (Web3) |
| --- | --- | --- |
| Granular Permissions | Provides permissions for specific actions on the platform. | Provides permissions for specific actions across different contracts, while setting parameters for actions, like a maximum gas amount, maximum transaction volume, etc. |
| Expiration Time | JWTs have an expiration time, requiring token refresh or user re-authentication. | Session keys also have an expiration time, ensuring temporary permissions. |
| User Experience | Allows users to stay logged in across sessions and share sessions across devices. | Enables users to interact with a DApp within pre-set rules without the need to sign every transaction. |
| Security | If someone gains access to a JWT, they would potentially have the same permissions as the user until the token expires. The impact depends on the scope of the permissions granted by the JWT, and could include unauthorized access to user data or actions performed on their behalf. | If a session key is compromised, the potential damage is limited by the specific rules and parameters set for that key. For instance, a malicious actor might be limited in the amount of tokens they can transact, the gas they can spend, or the duration they can interact with a DApp. |
| Adoption | Widely adopted in Web2. | A relatively new concept in Web3, but already used (with mostly proprietary implementations) by several gaming projects including Loot Realms, Briq, Topology, Cartridge, MatchboxDAO, and more. |
| Decentralization | JWTs are issued and verified by a centralized server. | Session keys are issued and verified by the user's wallet, and are entirely decentralized. |
| Interoperability | Limited to platforms that support JWTs. | Universally compatible with DApps on any blockchain that supports AA wallets |
| Trust | Trust in the central server is required for issuance and verification. | Trust is not required as rules are set by the user and signed with the user’s master key |

# Empowering DApps with session keys

It’s hard to overstate the impact of a flexible and interoperable means of authorization.  By taking advantage of session keys, DApps can create experiences that are simply impossible to create otherwise.

In essence, session keys allow transactions that seem "automatic" from the user's perspective. This can occur in two forms:

- **Skipping confirmations while the user is online**.  This is most useful in social and gaming applications, where frequent, small transactions are common.  Each time a user has to sign a transaction, it disrupts the gaming experience.  Using session keys, the user can pre-authorize a range of transactions, allowing them to enjoy the game uninterrupted.
- **Executing transactions when the user is offline**.  Consider a user who wants to ensure that they don't miss out on an NFT drop while they're asleep.  The user can create a session key authorizing the purchase of the NFT at the drop time, even if they're not online.  As another example, DeFi applications can use session keys to automatically exit risky positions for users so they don’t get liquidated.

# Use session keys with ZeroDev today

At ZeroDev, we recognized the potential of session keys early on.  After many iterations, we now have what we believe to be the most advanced implementation of session keys in the ecosystem.

We'll save the implementation details for a future blog post, but here's a brief overview of what makes ZeroDev's session keys special:

- ZeroDev session keys can be created *off-chain*.  This means that creating a new session key does NOT require an on-chain transaction, enabling your applications to generate a large number of session keys without paying any gas.
- ZeroDev session keys can define the scope of transactions based on various parameters such as contract addresses, function names, and ERC-165 interfaces (e.g., ERC20/ERC721), among others.
- ZeroDev session keys can be authorized using only a public key.  Thus, a client can create a public-private key pair and send just the public key to the master signer for authorization.  This approach ensures that the private part of the session key never needs to be shared, minimizing the risk of leakage.

Session keys are available with ZeroDev today.  **[Get started here](https://docs.zerodev.app/use-wallets/use-session-keys)** and build some groundbreaking DApps!

# **The Road Ahead**

The quest to fix Web3 UX is a long one, but session keys are a significant step towards this goal, offering a practical and secure solution to a major issue — the lack of authorizations — that has been largely overlooked until now.

The widespread adoption of session keys will require collaboration across the ecosystem. Wallet providers will need to support account abstraction (AA), and DApps will need to be built to take advantage of session keys.  However, the benefits of dramatically improved user experience and security make the effort worthwhile.

As more projects adopt session keys and AA in general, we will see the gap between Web2 and Web3 experiences begin to close.  [ZeroDev](https://docs.zerodev.app/) is proud to be a part of this journey, and we hope our contributions will make your users smile!