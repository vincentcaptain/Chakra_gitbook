---
hidden: true
---

# Copy of BTC Collateral Transformation

## Overview

The Chakra Settlement Layer enables the interchangeability between [settlement artifacts](../concepts/chakra-settlement-artifacts/), a proof of asset in Chakra, facilitating the transformation of BTC. This setup can ensure that BTC is securely managed throughout its lifecycle, whether in a custody or staking state. The BTC is not just represented differently on the blockchain but is **actually moved and managed** in a way that optimizes security and staking rewards.&#x20;

In this tutorial, you will implement a contract on Chakra to convert a Cobo BTC Settlement Artifact into a Cobo + Babylon Settlement Artifact.

## **Before you begin:**

1.  The Chakra Settlement Layer is a fully EVM-compatible chain, so most of your development journey will revolve around EVM and Solidity. You should know how to [develop](https://soliditylang.org/) and [deploy](https://remix.ethereum.org/) Solidity smart contracts. You should also know how to use a wallet; [MetaMask](https://metamask.io/) is a good starting point.

    As you will be developing on the Chakra Settlement Layer, you need to prepare some Chakra Settlement Layer Chain native tokens. Here's how to obtain them from the Chakra Settlement Layer faucet: \[Faucet information to be added]

## Workflow

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>Interchange between Cobo and Babylon + Cobo Settlement Artifact</p></figcaption></figure>

The figure above shows the Interchange workflow between Cobo and Babylon + Cobo Settlement Artifact. The breakdown of the workflow is attached:&#x20;

### **Cobo Settlement Artifact -> Cobo+Babylon Settlement Artifact:**

1. **Conversion Request**: The user initiates a conversion request from their account, requesting to convert a Cobo Settlement Artifact to a Cobo+Babylon Settlement Artifact.
2. **Event Listening**: The Chakra Settlement Service listens for this conversion event.
3. **BTC Withdrawal and Staking Request**: The Chakra Settlement Service sends a request to the Cobo Service to withdraw BTC and stake it in the Babylon Network.
4. **Asset Locking**: The Chakra Cobo Wallet locks the assets in the Cobo Babylon Staking Wallet.
5. **Delegation Record Generation**: The Babylon Network indexes the staking event and generates a Delegation Record within the network.
6. **IBC Channel Notification**: The Babylon Network notifies the Chakra Network of the Delegation Record via the IBC channel.
7. **Artifact Generation**: The Chakra Network generates a Cobo+Babylon Settlement Artifact for the user.

### **Cobo Settlement Artifact <- Cobo+Babylon Settlement Artifact:**

1. **Conversion Request**: The user initiates a conversion request to convert a Cobo+Babylon Settlement Artifact back to a Cobo Settlement Artifact.
2. **Undelegation Transaction Request**: The Chakra Network sends an Undelegation transaction request to the Babylon Network via the IBC channel.
3. **Unbonding Transaction**: The Babylon Network processes the Undelegation request and sends an unbonding transaction to the BTC Network. The Chakra Settlement Service indexes this event locally.
4. **Unbonding Period**: After the unbonding period, the Chakra Settlement Service requests the transfer of assets from the Babylon Staking Wallet to the Chakra Custody Wallet.
5. **BTC Reception**: The Chakra Custody Wallet receives the BTC and generates a Cobo Settlement Artifact for the user.

From the process above, we can tell that:

* For **Cobo Settlement Artifact -> Cobo+Babylon Settlement Artifact**, the actual BTC is moved from a simple custody setup to a staking setup within the Babylon Network. This involves withdrawing BTC from the Cobo Custody Wallet and staking it in the Cobo Babylon Staking Custody Wallet.
* Conversely, when converting back from a Cobo+Babylon Settlement Artifact to a Cobo Settlement Artifact, the staked BTC is unbonded and transferred back to a custody setup. This process requires coordination across multiple networks and includes staking, delegation, undelegation, and unbonding operations, which are far more complex than a simple token swap.&#x20;

This setup can ensure that BTC is securely managed throughout its lifecycle, whether it is in a custody or staking state. The BTC is not just represented differently on the blockchain but is **actually moved and managed** in a way that optimizes security and staking rewards.
