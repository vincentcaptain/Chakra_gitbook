---
hidden: true
---

# Copy of LST Settlement Tutorial #2: Injecting BTC Liquidity to Chakra

In the previous sections, we described the [Settlement Primitives](../../concepts/chakra-settlement-primitives/) provided by Chakra and their corresponding [Settlement Artifacts](../../concepts/chakra-settlement-artifacts/). We also introduced the Settlement Artifacts [Registry Contract](../../concepts/chakra-settlement-artifacts/interfaces-of-artifacts-and-its-registry.md). The Chakra Settlement Layer can integrate with BTC settlement channels of other projects and encapsulate them into Settlement Primitives and Settlement Artifacts. Users can settle their BTC through the LST Project Settlement Primitive, which will be managed by the Settlement Artifact Contract on the Chakra Settlement Layer.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## **Workflow:**

1. **User Deposit to LST Project Settlement Channel**: The user deposits their BTC into the BTC LST Project Settlement Channel.
2. **Deposit to Chakra Settlement Layer**: The BTC deposited into the LST Project Settlement Channel is then deposited into the LST Settlement Artifact Contract within the Chakra Settlement Layer.
3. **Minting Settlement Artifacts**: The LST Settlement Artifact Contract mints new Settlement Artifacts corresponding to the deposited BTC and credits them to the user's account on the Chakra Network.
4. **Maintaining Records**: The Settlement Artifact Registry Contract maintains a record of all minted Settlement Artifacts, ensuring a consistent and auditable log of all transactions.
5. **User Withdraw**: When a user wishes to withdraw, they initiate a request to the LST Settlement Artifact Contract.
6. **Withdrawal from Chakra Settlement Layer**: The LST Settlement Artifact Contract processes the withdrawal, burning the necessary Settlement Artifacts.
7. **Withdrawal from LST Project Settlement Channel**: The BTC corresponding to the burned Settlement Artifacts is then withdrawn from the BTC LST Project Settlement Channel and transferred back to the user's BTC wallet.

For the step-by-step instruction on [Token Transfer Settlement](../cross-chain-transfer-with-settlement-layer.md), please see [here](../cross-chain-transfer-with-settlement-layer.md).&#x20;
