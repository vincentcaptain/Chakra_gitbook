# Settlement Primitive: Cobo

## Overview

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>Cobo Settlement Primitive Workflow</p></figcaption></figure>

The figure above describes how the Cobo Settlement primitive operates, leveraging the integration between the BTC Network and the Chakra Network through the Cobo Custody Wallet service. This process allows users to deposit BTC into a Cobo Wallet and receive a corresponding Cobo Settlement Artifact on the Chakra Network and the reverse operation for withdrawing BTC.

## Workflow

### **1. BTC Deposit**

1. **User Wallet to Cobo Wallet (BTC Network)**: The user initiates a transfer of BTC from their personal wallet to a designated Cobo Wallet within the BTC Network.
2. **Cobo Wallet to Cobo Service**: Once the BTC is received in the Cobo Wallet, the Cobo Service indexes this transaction.
3. **Cobo Service Notification**: The Cobo Service sends a notification to the Cobo Settlement Service on the Chakra Network, indicating that a deposit has been made.
4. **Cobo Settlement Service (Chakra Network)**: Upon receiving the notification, the Cobo Settlement Service creates a mint request.
5. **Cobo Settlement Artifact Contract**: The Cobo Settlement Artifact Contract processes the mint request and mints a new Cobo Settlement Artifact.
6. **User Address (Chakra Network)**: The newly minted Cobo Settlement Artifact is transferred to the user's address on the Chakra Network.

### **2. BTC Withdrawal Workflow**

1. **User Address to Cobo Settlement Artifact Contract (Chakra Network)**: The user initiates a withdrawal request by sending the Cobo Settlement Artifact to the Cobo Settlement Artifact Contract, which burns the artifact.
2. **Cobo Settlement Service**: The Cobo Settlement Artifact Contract notifies the Cobo Settlement Service of the burn event.
3. **Withdraw Request to Cobo Service**: The Cobo Settlement Service sends a withdraw request to the Cobo Service.
4. **Cobo Service to Cobo Wallet**: The Cobo Service processes the withdrawal request and transfers the corresponding amount of BTC from the Cobo Wallet back to the user's wallet within the BTC Network.
5. **Cobo Wallet to User Wallet (BTC Network)**: The user receives the BTC in their personal wallet, completing the withdrawal process.

On top of this workflow, we are able to establish [settlement primitive with Babylon](babylon-+-cobo-settlement-primitive.md) to foster native BTC staking.&#x20;
