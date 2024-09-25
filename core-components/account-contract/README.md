---
description: >-
  WIP. Following is the draft which might changes after proof reading and fact
  checking.
---

# Account Contract

An **Account Contract** is a smart contract wallet where users hold their funds instead of using traditional externally owned accounts (EOAs). The validation of a UserOperation happens within these account contracts. The developer of the account contract defines the validation logic, ensuring that the UserOperation is properly signed by the user and authorized.

The **ERC-4337 specification** outlines the minimum requirements for an Account Contract. You can find the full specification [here](https://eips.ethereum.org/EIPS/eip-4337#account-contract-interface). Below is the interface for an Account Contract:

```solidity
interface IAccount {
  function validateUserOp
      (PackedUserOperation calldata userOp, bytes32 userOpHash, uint256 missingAccountFunds)
      external returns (uint256 validationData);
}
```

All Account Contracts must follow this interface. When a UserOperation is sent to the entry point with the address of the target contract (i.e., the Account Contract), the entry point will call the `validateUserOp` function with the required arguments. Let's take a look at arguments of this function:

* userOp: This is PackedUserOperation that bundler created whic is sent to entrypoint and then to account contract.
* userOpHash: This is generated using UserOp without signature, address of entrypoint, and chainId of the network.
* missingAccountFunds: This is the amount that account needs to compensate to bundler for sending it on chain.

Here’s what happens in the `validateUserOp` function:

* After the validateUserOp is called. It verifies the UserOperation based on the verification logic developer has implemented. Validating the UserOperation happens by checking signature is signed by the sender or the owner of the account.&#x20;
  * Let's go on a tangent here and understand some caveats. Even though with AA is used to remove EOA, most of AA solutions right now use MPC wallets to abstract way visbile interacttion with EOAs like MetaMask, Keplr, etc. but there is still EOA that signs this user operation.
  * MPC (multi party computation) wallet allows users to split private key and store that with different parties which can be combined to use for any operation.
  * In future, the can be other signature algorithms used to verify the signature like P256 (RIP-7212) but for now most of the solution use ECDSA signature to verify.
* Additionally, developer can also verify the userOpHash to make sure signature from one chain is not passed other to prevent replay attack. &#x20;
* After validation, missingAccountFund is sent to entrypoint via \_payPrefund function which refunds to bundler.

Once the UserOperation is validated, the entry point needs to execute it. There are two possible functions the entry point might call: `execute` and `executeUserOp`.

* If the calldata sent with the UserOperation contains the function signature for `executeUserOp`, the entry point will send the entire UserOperation to that function.
* Otherwise, it will send only the calldata, along with the destination address and the value, to the `execute` function.

Here’s the definition of the `execute` function:

```solidity
function execute(address dst, uint256 value, bytes calldata functionData)
```

The `executeUserOp` function comes from the `IAccountExecute` interface, which developers need to implement if they want to use this functionality. Here’s the `IAccountExecute` interface

```solidity
interface IAccountExecute {
  function executeUserOp(PackedUserOperation calldata userOp, bytes32 userOpHash) external;
}
```

