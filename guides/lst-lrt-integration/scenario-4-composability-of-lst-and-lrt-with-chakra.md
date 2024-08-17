---
hidden: true
---

# Scenario 4: Composability of LST and LRT with Chakra

Settlement Artifacts serve as the liquidity foundation for LST/LRT projects within the Chakra Settlement Layer. Through Chakra, users can obtain a Hybrid LST Token through the LST DApp. The LST project utilizes two types of Settlement Artifact Contracts to derive a mixed LST Token, which increase the composability of LST and LRT.

The diagram below illustrate a simple example involves the LST Project using Cobo Settlement Artifacts and Babylon + Cobo Settlement Artifacts to validate the Hybrid LST Token. Due to the different underlying assets represented by these two types of Settlement Artifacts, they exhibit distinct characteristics:

1. **Cobo Settlement Artifact (from its** [**primitive**](../../concepts/chakra-settlement-primitives/settlement-primitive-cobo.md)**)**:
   * Represents spot BTC, which can be immediately redeemed without any loss.
2. **Babylon + Cobo Settlement Artifact (from its** [**primitive**](../../concepts/chakra-settlement-primitives/babylon-+-cobo-settlement-primitive.md)**)**:
   * Represents staked BTC, which might be subject to slashing and requires waiting for an unbonding period to redeem.

An LST project might create a Hybrid LST Token using a combination of 0.4 Cobo Settlement Artifact and 0.6 Babylon + Cobo Settlement Artifacts to form one Hybrid LST Token.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## **Workflow:**

1. **User Interaction with DApp**: The user interacts with the DApp to initiate the process of obtaining a Hybrid LST Token.
2. **Deposit BTC**: The user deposits BTC into the DApp, which is then processed through the BTC Network.
3. **Generating Settlement Artifacts**: The deposited BTC is split between the two Settlement Artifact Contracts:
   * Project1 Settlement Artifact Contract (handling Cobo Settlement Artifacts).
   * Project2 Settlement Artifact Contract (handling Babylon + Cobo Settlement Artifacts).
4. **Minting Hybrid LST Tokens**: The corresponding Settlement Artifacts are deposited into the Hybrid LST Token contract. The contract mints Hybrid LST Tokens, combining 0.4 Cobo Settlement Artifact and 0.6 Babylon + Cobo Settlement Artifact for each token.
5. **User Receives Hybrid LST Tokens**: The minted Hybrid LST Tokens are credited to the user’s account.
6. **Burning Hybrid LST Tokens**: Users who wish to redeem their Hybrid LST Tokens initiate a burn transaction. The Hybrid LST Token contract burns the tokens and converts them back into the respective proportions of Cobo and Babylon + Cobo Settlement Artifacts.
7. **Withdraw BTC**: The user can use the Settlement Primitives to convert these Settlement Artifacts back into BTC. The BTC is then withdrawn from the Chakra Settlement Layer to the BTC Network, and finally credited back to the user’s BTC wallet.

The detail instruction on two examples on liquidity composability on Chakra can be found here.&#x20;

