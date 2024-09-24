---
description: Understanding the Concept.
---

# What is Account Abstraction?

Account abstraction is a new approach to how users manage their accounts. It allows users to use smart contracts as wallets, reducing the need for EOAs (externally owned accounts) like MetaMask, Keplr, etc.

In the EVM, there are two types of accounts:

* **EOA (Externally Owned Account):** Controlled by a private key.
* **Contract Account:** Controlled by code. These accounts can’t initiate transactions on their own; they only act when called.

Both types of accounts share common features: they can hold assets, transfer funds, and interact with other accounts. The key difference is that only EOAs can create transactions, making them essential for interacting with contracts.

However, EOAs have limitations:

* **Limited security:** EOAs use public-key cryptography, where control of the wallet and its assets depends on the private key. If this key is leaked or shared, whoever holds it gains complete access to the assets without any safeguards.
* **Lack of customization:** EOAs lack flexibility, as they don’t let users assign partial access to funds or set custom spending limits, forcing reliance on external solutions for these needs.
* **Lack of future-proofing:** As quantum computing advances, EOAs provide no built-in protection against potential future vulnerabilities.

Account abstraction addresses these issues by decoupling transaction verification from how transactions are processed on the blockchain. Let's explore how this works.
