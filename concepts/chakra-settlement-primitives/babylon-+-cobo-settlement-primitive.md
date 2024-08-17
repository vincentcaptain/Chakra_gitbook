---
hidden: true
---

# Babylon + Cobo Settlement Primitive

## Overview

The Babylon + Cobo Settlement Primitive integrates the BTC Network, Babylon Network, and Chakra Network to facilitate secure and efficient staking and settlement processes. This primitive allows users to stake their BTC and receive a corresponding Babylon + Cobo Settlement Artifact on the Chakra Network, which can later be redeemed for BTC through a series of coordinated actions across the networks.

A similar workflow applies to other MPC solutions integrated with native staking protocols like Babylon, and the artifacts generated are [interchangeable](../../guides/btc-collateral-transformation.md). You can see the currently supported solutions [here](./).&#x20;

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>Babylon + Cobo Settlement Primitive Workflow</p></figcaption></figure>

## Workflow

### **Generating Babylon Cobo Settlement Artifact**

1. **BTC Deposit (on BTC Network)**: When a user deposits BTC into the Cobo Babylon Staking Custody Wallet, the Babylon Network records this staking transaction and creates a Delegation record.
2. **IBC Channel Notification (through IBC Channel)**: The Babylon Network notifies the Chakra Network of the new Delegation record via the IBC channel.
3. **Minting Babylon Cobo Settlement Artifact (on Chakra Network)**: Upon receiving the notification, the Chakra Network mints a corresponding Babylon Cobo Settlement Artifact through the Babylon Cobo Settlement Artifact Contract and assigns it to the user’s address.

### **Redeeming Babylon Cobo Settlement Artifact for BTC**

1. **Burn Artifact (on Chakra Network)**: The user initiates the redemption process by burning the Babylon Cobo Settlement Artifact on the Chakra Network.
2. **Sending Undelegate Transaction (through IBC Channel)**: The Chakra Network sends an Undelegate transaction request to the Babylon Network via the IBC channel.
3. **Processing Withdraw Transaction (on Babylon)**: The Babylon Network processes the Undelegate request and generates a corresponding Withdraw transaction, which is then recorded on the blockchain.
4. **Unlocking BTC (to BTC Network)**: After the Babylon Unbonding Period, the Cobo Babylon Staking Custody Wallet unlocks the BTC from the delegation transaction and transfers the unlocked BTC to the user’s wallet.
