---
slug: kernel-v2-and-the-lessons-we-learned
title: Kernel v2 and the Lessons We Learned
authors: derek
hide_table_of_contents: true
---

Ever since releasing [Kernel v1](https://twitter.com/zerodev_app/status/1650936162436128769), we have seen a flurry of activities from developers building novel plugins on Kernel.  However, developers soon ran into significant limitations that exposed some of the shortcomings of Kernel, which prompted us to start working on Kernel v2.

In this blog post, we will dive into some of the issues with Kernel v1 and how we addressed them in v2.

A fair warning: this blog is written for a technical audience who want to understand the inner workings of Kernel, especially plugin developers.  Most users of ZeroDev do not need to understand what’s described in this blog.

## Issues with Kernel v1

### Validation and execution are closely coupled

In Kernel v1, plugins modify how transactions are validated.  Once validated, the transactions are executed through a hardcoded `execute` function.

However, certain use cases turned out to require not just custom validation, but also custom execution.  For example, the default `execution` function allows both `call` and `delegatecall`, and the latter makes it very hard to reason about the security of a plugin.  Therefore, some plugin developers wanted to disable `delegatecall` altogether, but that was hard to do with Kernel v1.

### Inability to add custom functions

While the default `execute` function is meant to be flexible enough to execute arbitrary transactions, sometimes there are legit needs to implement custom functions.  For example, let’s say there is a new ERC like ERC-1271 that requires the implementation of new functions.  With Kernel v1, there’s no way to dynamically extend the contract to implement the new interface.

### Inability to change the default validation function

Plugins in Kernel v1 introduce new “paths” for transactions to be validated, but there wasn’t a way to update the “default path” — which validates ECDSA signatures from the wallet owner.  In other words, the wallet owner can always execute transactions, regardless of plugins.

While this is normally desired, there are cases where the “default path” needs to be modified or even outright blocked.  For example, if you want to build a 2FA account, it’s not enough to build a 2FA plugin — you also need to make sure that the default ECDSA validation function is no longer effective, or it would defeat the point of 2FA.

### Overlapping storage between Kernel and plugins

In Kernel v1, plugins are invoked through `delegatecall`.  To prevent storage collision between plugins and the kernel, plugins are required to use unstructured storage, sometimes known as “diamond storage.”

However, this requirement cannot be enforced, and a plugin needs to be carefully audited to ensure that it’s in fact not using any storage outside of its area.  This places heavy burden on the user of the plugin as well as auditors.

## Design Decisions for Kernel v2

Kernel v2 draws on the lessons we learnt from real-world applications building on Kernel v1.  At the core of Kernel v2’s architecture is two key design decisions:

- Separation of plugin storage from kernel storage.
- Separation of validation from execution.

### Separation of Plugin Storage from Kernel Storage

In Kernel v1, plugins are invoked through `delegatecall`, which means plugins and Kernel ultimately share the same storage.  Therefore, plugin authors need to take care to not “touch”  the storage area of the Kernel, by using “[diamond storage](https://dev.to/mudgen/how-diamond-storage-works-90e).”  This places burden on the plugin author, the plugin auditor, as well as the user to ensure that the plugin correctly handles storage.

In Kernel v2, validator plugins are invoked through `call`.  Therefore, validator plugins have no access to the Kernel’s storage, vastly reducing the surface of attack.

### Separation of Validation from Execution

Whereas there are only “validation plugins” in Kernel v1, there are now two classes of plugins in Kernel v2: `validators` and `executors`.

#### Validators

Validators are plugins that modify how transactions are validated.  These plugins are akin to the plugins in Kernel v1.

One notable difference is that in v2, it’s possible to replace the “default” validator.  For example, if you want to set up an account as 2FA, you would set the default validator to the 2FA plugin, therefore replacing the default ECDSA plugin.  This makes it impossible to send transactions without going through 2FA.

#### Executors

Executors are plugins that add custom functions to Kernel.  In particular, each custom function is tied to a validator, meaning that a call to a custom function is “routed” to a particular validator.

The ability to route each function to a different validator makes it possible to implement ultra-fine-grained security policy.  For example, you might want to add a custom function to Kernel, but you ONLY want that function to be called if the user goes through 2FA.  With Kernel, you can set up routing so that the custom function (executor) is routed through 2FA (validator).

## Kernel v2 Architecture Overview

In ERC-4337, a transaction (aka “UserOp”) is processed in two phases: a validation phase and an execution phase.  To understand how Kernel v2 works, let’s walk through the lifecycle of a UserOp as processed by Kernel.

### Validation Phase

In the validation phase, the `EntryPoint` calls the `validateUserOp` function on Kernel. Transactions to Kernel can be executed in one of three "modes," as indicated by the first few bytes of the UserOp's `signature` field.

- Sudo mode (0x0)
    
    In sudo mode, Kernel's "default validator" is invoked. The default validator is a plugin that determines how transactions are validated by default (that is, if the transaction is not handled by another plugin). In ZeroDev, the default validator is normally set to the [ECDSA validator](https://github.com/zerodevapp/kernel/blob/main/src/validator/ECDSAValidator.sol), which approves a transaction if it's signed by the owner through ECDSA -- just like a regular transaction.
    
- Plugin mode (0x1)
    
    In plugin mode, Kernel "looks up" the validator to use by the function selector from the `calldata`. The mapping between function selectors and validators are set through the "enable mode," which will be explained later.
    
    In any case, once a validator has been looked up, it's used to validate the transaction.
    
- Enable mode (0x2)
    
    In enable mode, Kernel "enables" a validator, and it does so by associating the current function selector with the validator. The validator's address (keep in mind that plugins are smart contracts) is encoded inside the `signature` itself.
    
    Once enabled, the validator will be used to validate this and every subsequent invocation of the same function in plugin mode.
    

### Execution Phase

In enable mode, Kernel actually associates with the function selector not just the validator, but also the executor. Executors are smart contracts that actually implement the function that corresponds to the selector. That is, when you call the function `kernel.someFunction()`, the `someFunction` is actually implemented in an executor, not the `kernel` itself.

When EntryPoint calls the function, Kernel uses a [fallback function](https://docs.soliditylang.org/en/v0.8.20/contracts.html#fallback-function) to look up the executor associated with the function selector, then `delegatecall`s the executor to execute the function. If you are familiar with EIP-2535 aka "Diamond Proxies," you can think of executors as "facets."

## Next Steps

Today we are happy to announce that Kernel v2 has passed the initial audit and therefore entered public beta.  You can read more about Kernel and how to build plugins on [our docs](https://docs.zerodev.app/extend-wallets/overview), or read the [raw notes](https://hackmd.io/joe9mwzPRCCA5Mw0JVWzBw) by Taek which this blog is based off of.  If you want to build some plugins, join our [Discord](https://discord.gg/KS9MRaTSjx) and head to #plugin-devs where our community can help!