# LST/LRT Integration

## Overview

Integrating LRT (Liquid Reserves Token) and LST (Liquid Staking Token) into the Chakra Settlement Layer enables secure and efficient settlement processes for BTC and other assets. This integration leverages Settlement Primitives and Settlement Artifacts to maintain balance and security within the system, even under market volatility.

We provide four typical scenarios that LRT and LST protocols can utilize Chakra Settlement Layer to boost their liquidity:&#x20;

* [Scenario 1: Settle Native BTC to Target Chain Through Chakra](lst-settlement-tutorial-1-btc-to-stbtc-conversion.md)

The process starts with the user locking their Cobo Settlement Artifact on the Chakra Network. This action triggers a minting process on the target chain where an equivalent amount of stBTC is minted to the user's account. Conversely, the user can burn their stBTC on the target chain, which triggers the unlocking of the original Settlement Artifact on the Chakra Network.

* [Scenario 2: Injecting Liquidity to Chakra to Access Omni-chain BTC Settlement Channel ](lst-settlement-tutorial-2-injecting-btc-liquidity-to-chakra.md)

The Chakra Settlement Layer integrates with external BTC settlement channels and encapsulates them into its Settlement Primitives and Artifacts, providing a seamless user experience. When users deposit BTC, new Settlement Artifacts are minted and recorded by the Settlement Artifact Registry Contract. This ensures transparency and traceability of assets. Users can easily withdraw their assets by burning the appropriate Settlement Artifacts, which triggers the corresponding BTC transfer from the LST Project Settlement Channel back to their wallet.

* [Scenario 3: Settlement into LST/LRT Token](scenario-3-settlement-into-lst-lrt-token.md)

Users deposit BTC into the Chakra Settlement Layer, resulting in the minting of Settlement Artifacts. These Settlement Artifacts can be deposited into the LST project to mint LST Tokens. Users can also burn LST Tokens to retrieve Settlement Artifacts. Finally, Settlement Artifacts can be converted back into BTC through the Settlement Primitive, completing the cycle of liquidity composability.

* [Scenario 4: Composability of LST and LRT on Chakra](scenario-4-composability-of-lst-and-lrt-with-chakra.md)

The Chakra Settlement Layer enables the seamless integration of liquidity provided by LRT and LST projects through the use of Settlement Artifacts. These artifacts act as standard ERC20 tokens, allowing developers to wrap them into new tokens, create liquidity pools, and build versatile decentralized applications (DApps). This composability is achieved by leveraging EVM-compatible smart contracts, which can mint, deposit, and withdraw these artifacts.
