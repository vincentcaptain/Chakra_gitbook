---
hidden: true
---

# Scenario 3: Settlement into LST/LRT Token

In this scenario, a Settlement Artifact is deposited into the LST project, generating the corresponding LST Token. Users can also burn LST Tokens to receive Settlement Artifacts, which can then be settled into BTC using the Settlement Primitive, as shown below:

&#x20;

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

## **Workflow:**

1. **Deposit BTC to Chakra Settlement Layer**: The user deposits BTC into the Chakra Settlement Layer through the DApp.
2. **Minting Settlement Artifacts**: The Settlement Artifact Contract within the Chakra Settlement Layer mints new Settlement Artifacts corresponding to the deposited BTC and credits them to the user's account.
3. **Generating LST Tokens**: The user deposits the Settlement Artifacts into the LST project. The LST Settlement Artifact Contract processes this deposit and mints the corresponding LST Tokens, which are credited to the user's account.
4. **Burning LST Tokens**: To convert LST Tokens back into Settlement Artifacts, the user initiates a burn transaction for their LST Tokens.
5. **Returning Settlement Artifacts**: The LST Settlement Artifact Contract burns the specified LST Tokens and credits the corresponding Settlement Artifacts back to the user's account.
6. **Settlement into BTC**: The user can then use the Settlement Primitive to convert these Settlement Artifacts back into BTC. The Settlement Primitive facilitates the withdrawal of BTC from the Chakra Settlement Layer to the BTC Network, where it is credited to the user's BTC wallet.

For the implementation of [Settlement Handler,](../../concepts/settlement-handler-contract.md) please see [here](../../resources/contracts-and-script-templates/settlement-handler-contract/)
