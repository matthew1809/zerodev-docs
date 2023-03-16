---
slug: erc-6492-and-why-its-important-for-aa
title: What is ERC-6492 and why it’s important for Account Abstraction
authors: derek
---

# What is ERC-6492 and why it’s important for Account Abstraction

Unless you check new ERCs everyday (in which case, good for you), you probably haven’t noticed this new ERC known as [ERC-6492](https://eips.ethereum.org/EIPS/eip-6492), innocuously named "Signature Validation for Predeploy Contracts.”  As this post is going to argue, ERC-6492 is critical to the wide adoption of account abstraction and smart contract wallets in general.

We will now explain the issue that ERC-6492 addresses, briefly touch on the technicalities of how ERC-6492 solves the problem, and end by explaining why it’s critical for key ecosystem projects such as Ethers and SIWE to adopt this new standard.

# Terminology

For brevity, I will be using the following pairs of terms interchangeably, even though it’s not technically accurate:

- “AA” and “ERC-4337”
- “AA wallets” and “smart contract wallets”
- “Wallets” and “accounts”

# The Context

As I explained in [a previous post](https://docs.zerodev.app/blog#aa-is-not-compatible-with-existing-dapps), AA wallets are mostly compatible with existing DApps since AA transactions look no different than normal transactions from the perspective of the DApps, except for some edge cases.

Signature validation is a different story, however.  As you know, many DApps require some form of signing; OpenSea for instance requires the user to sign a message before they can “log into” the DApp.

Since smart contract wallets have the flexibility to support different signing schemes, there isn’t a universal way to validate signatures by a smart contract wallet.  Instead, there’s a standard [ERC-1271](https://eips.ethereum.org/EIPS/eip-1271) which defines a standard function `isValidSignature` on a smart contract wallet so that the verifier (e.g. OpenSea) can call the function to validate the signature, without needing to know specifically what signing scheme the wallet uses.

This is all fine and good, and in fact ERC-1271 as a standard enjoys wide adoption.  Most popular DApps today, including OpenSea, already support it.

# The Issue

With the rise of ERC-4337, smart contract wallets are becoming increasingly commonplace.  One key optimization that ERC-4337 implements is *counterfactual deployment* — namely, that we can compute the address of the account before the underlying smart contract is actually deployed.  As a result, a user can “create” a ERC-4337 wallet without paying the deployment cost, so they can start receiving assets, signing into DApps, etc.  Only when the user sends their first transaction that the contract is actually deployed.

While counterfactual deployment is normally very desirable, it becomes an issue when the user needs to sign messages.  To understand why, recall that in order to validate a signature from a smart contract wallet, the verifier needs to call `isValidSignature` on the wallet contract.  However, since the wallet contract is not actually deployed, it’s impossible to call that function!  As a result, an attempt to validate that signature will fail.

# Consequences

So what does this mean for the users?  It means that it’s impossible to validate signatures from ERC-4337 wallets until they are deployed.  Therefore, for a new ERC-4337 wallet that has not sent any transactions, it’s impossible to, say, sign into OpenSea or any DApp that uses [SIWE](https://login.xyz/).

This is very bad because users who are new to Web3 want to sign into DApps and look around before they spend any money on gas.  Forcing a user to pay some gas to deploy their wallets before they can see a DApp would be a major step backwards comparing to the EOA experience today, where you can sign into DApps even from an empty account.

# Solution

ZeroDev first encountered this problem when we were developing our WalletConnect integration and realized that we couldn’t sign into OpenSea until we deployed the wallet, which led to [a lengthy discussion](https://github.com/eth-infinitism/account-abstraction/issues/188) with many smart people in the 4337 ecosystem.  Eventually, [Ivo from Ambire](https://twitter.com/Ivshti) came up with a great solution that turned into ERC-6492.

On a high level, ERC-6492 works by using a `[UniversalSigValidator` contract](https://eips.ethereum.org/EIPS/eip-6492#reference-implementation) that validates a signature as such:

- Check if the signature ends with a sequence of *magic bytes*, which indicate that the signature is for a not-yet-deployed contract.
    - If so, the signature itself contains all the data necessary for deploying the contract, which comes down to an *account factory* address and the *calldata* for the factory.
    - `UniversalSigValidator` would then proceed to deploy the contract and calls `isValidSignature` on it to validate the signature.
- If the magic bytes are not detected, then proceed as normal, which means:
    - Check if there’s contract code at the address.  If so, proceed with ERC-1271.
    - Otherwise, assume that the account is an EOA and perform a `ecrecover`.

But wait!  You might say.  The signature verifier has to *deploy* the contract if it doesn’t exist?  Isn’t that incurring a lot of cost for the verifier?

The answer is no because the verifier will be using `[eth_call](https://docs.alchemy.com/reference/eth-call)`, which essentially simulates the transaction without actually executing it on-chain.

# Next Steps

So who needs to implement ERC-6492?  In short, it’s whoever that needs to verify signatures, which is mostly DApps.

However, DApps don’t write everything from scratch.  In fact, there are a few libraries that most DApps use for handling signatures, so if these libraries adopt ERC-6492, DApps would get to support ERC-6492 “for free.”  Some of these key libraries are:

- Ethers: Ivo from Ambire has helpfully created [a PR for supporting ERC-1271 validation](https://github.com/ethers-io/ethers.js/pull/3904), which will be a stepping stone for ERC-6492.
- SIWE: SIWE already supports ERC-1271, so we have created [a PR that extends the support to ERC-6492](https://github.com/spruceid/siwe/pull/153), using [Ivo’s library](https://github.com/AmbireTech/signature-validator).

If you want to see the space move towards AA and smart contract wallets, there are a few things you can do:

- Upvote these PRs
- Make your own PRs to libraries that validate signatures
- And of course, if you are building a DApp, make sure that it can handle SCW signatures!  DApps that work seamlessly with SCW will have an inherent advantage comparing to those that don’t, since more and more traffic are moving to SCW everyday.