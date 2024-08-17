---
hidden: true
---

# \[ARCHIVED] LST Settlement Tutorial #1: BTC to stBTC Conversion

This tutorial demonstrates how to use Settlement Primitives and Settlement Artifacts to convert BTC into stBTC (staked BTC) on a target chain using the Chakra Network as an intermediary.

### Overview

We'll implement two main processes:

1. Locking Settlement Artifact on Chakra Network and Minting stBTC on Target Chain
2. Burning stBTC on Target Chain and Unlocking Settlement Artifact on Chakra Network

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Workflow

**Process 1: Locking Settlement Artifact on Chakra Network and Minting stBTC on Target Chain:**

1. **User Request**: The user initiates a request from their account on the Chakra Network to lock a Cobo Settlement Artifact into the LST Settlement Handler.
2. **Locking the Artifact**: The Cobo Settlement Artifact Contract locks the artifact into the LST Settlement Handler Contract on the Chakra Network.
3. **Settlement Message**: The LST Settlement Handler Contract sends a Settlement Message to the Settlement Contract on the target chain.
4. **Minting stBTC**: The Settlement Contract on the target chain receives the Settlement Message. The LST Settlement Handler Contract on the target chain then mints the corresponding amount of stBTC to the user’s account on the target chain.

**Process 2: Burning stBTC on Target Chain and Unlocking Settlement Artifact on Chakra Network:**

1. **User Request**: The user initiates a request from their account on the target chain to burn stBTC and withdraw the corresponding Settlement Artifact.
2. **Burning stBTC**: The LST Settlement Handler Contract on the target chain burns the stBTC and sends a Settlement Message to the Settlement Contract on the Chakra Network.
3. **Settlement Message**: The Settlement Contract on the Chakra Network receives the Settlement Message.
4. **Unlocking the Artifact**: The LST Settlement Handler Contract on the Chakra Network processes the Settlement Message and unlocks the corresponding Settlement Artifact. The Cobo Settlement Artifact Contract then unlocks the Settlement Artifact and returns it to the user’s account on the Chakra Network.

For the example implementation of a [smart contract with BTC settlement capability](../staking-settlement-artifacts-with-native-btc-withdraw.md), please see [here](../staking-settlement-artifacts-with-native-btc-withdraw.md)
