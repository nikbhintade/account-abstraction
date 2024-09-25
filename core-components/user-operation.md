---
description: >-
  WIP. Following is the draft which might changes after proof reading and fact
  checking.
---

# User Operation

`UserOperation` is a high-level pseudo-transaction object that is sent on behalf of the user. When a user interacts with a dApp that supports account abstraction (AA), the UserOperation is created and signed by the user through some form of authorization, such as using their externally owned account (EOA) or a method like social login, depending on how the dApp is designed.

Now, before diving into what happens next, let's take a look at the structure of a `UserOperation`:

| `sender`                        | `address` | The account making the operation                                                                       |
| ------------------------------- | --------- | ------------------------------------------------------------------------------------------------------ |
| `nonce`                         | `uint256` | Anti-replay parameter (see “Semi-abstracted Nonce Support” )                                           |
| `factory`                       | `address` | account factory, only for new accounts                                                                 |
| `factoryData`                   | `bytes`   | data for account factory (only if account factory exists)                                              |
| `callData`                      | `bytes`   | The data to pass to the `sender` during the main execution call                                        |
| `callGasLimit`                  | `uint256` | The amount of gas to allocate the main execution call                                                  |
| `verificationGasLimit`          | `uint256` | The amount of gas to allocate for the verification step                                                |
| `preVerificationGas`            | `uint256` | Extra gas to pay the bunder                                                                            |
| `maxFeePerGas`                  | `uint256` | Maximum fee per gas (similar to [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) `max_fee_per_gas`) |
| `maxPriorityFeePerGas`          | `uint256` | Maximum priority fee per gas (similar to EIP-1559 `max_priority_fee_per_gas`)                          |
| `paymaster`                     | `address` | Address of paymaster contract, (or empty, if account pays for itself)                                  |
| `paymasterVerificationGasLimit` | `uint256` | The amount of gas to allocate for the paymaster validation code                                        |
| `paymasterPostOpGasLimit`       | `uint256` | The amount of gas to allocate for the paymaster post-operation code                                    |
| `paymasterData`                 | `bytes`   | Data for paymaster (only if paymaster exists)                                                          |
| `signature`                     | `bytes`   | Data passed into the account to verify authorization                                                   |

The dApp constructs the UserOperation based on its specific needs and then sends it to a bundler. The bundler’s job is to gather multiple UserOperations and package them into a batch before sending them to the entry point contract for execution.

This packed version of the UserOperation is consolidated into a new type called `PackedUserOperation`, which looks like this:

```solidity
struct PackedUserOperation {
    address sender;
    uint256 nonce;
    bytes initCode;
    bytes callData;
    bytes32 accountGasLimits;
    uint256 preVerificationGas;
    bytes32 gasFees;
    bytes paymasterAndData;
    bytes signature;
}
```

Here’s a breakdown of the key fields:

* `initCode`: This is the combination of the factory contract’s address and the initialization data (factoryData) required to deploy the user’s account if it doesn’t exist yet.
* `accountGasLimits`: This is a combination of the `verificationGas` (16 bytes) and `maxFeePerGas` (16 bytes). It represents how much gas is needed to verify the operation and the maximum gas price the user is willing to pay.
* `gasFees`: This field combines the `maxPriorityFee` (16 bytes) and `maxFeePerGas` (16 bytes), indicating how much extra gas the user is willing to pay to prioritize the transaction and the maximum overall fee per gas unit.
* `paymasterAndData`: This contains information related to the paymaster, which might sponsor the transaction, and any extra data required by the paymaster.

Once the `UserOperation` is packaged, the bundler submits it to the entry point, where it’s processed on-chain.

