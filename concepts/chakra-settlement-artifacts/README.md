# Chakra Settlement Artifacts

## **Introduction**

Settlement Artifacts in the Chakra ecosystem serve as intermediary products for settlement purposes and are not intended for market circulation. Their primary role is to act as underlying proof of assets for LST and LRT. Settlement Artifacts play a crucial role in maintaining the security of BTC settlements alongside Settlement Primitives. Even when there is a significant imbalance between LST/LRT and the underlying BTC assets, the presence of Settlement Artifacts ensures the continued operation of [Settlement Primitives](../chakra-settlement-primitives/).

Due to the varying attributes of different Settlement Primitives, such as security, operational status, and capital scale, the value of each Settlement Artifact differs and is determined by the market.

## **Key Features**

1. **Non-Marketable Asset**: Settlement Artifacts are designed specifically for settlement purposes and cannot participate in market circulation. They serve as the underlying proof of assets for LRT and LST, providing stability and security to the settlement process.
2. **Security Assurance**: The primary function of Settlement Artifacts is to work in tandem with Settlement Primitives to ensure the security of BTC settlements. They help maintain the integrity and functionality of the settlement system even during periods of significant asset imbalance.
3. **Market-Driven Value**: The market determines the value of each Settlement Artifact based on the specific attributes of the corresponding Settlement Primitive. These attributes include security, operational status, and capital scale.
4. **Extended ERC20 Implementation**: Settlement Artifacts are implemented as smart contracts on the Chakra Chain, extending the ERC20 standard. This extension includes additional functionality to reveal the attributes of the Settlement Primitive that generated the artifact.

## **Benefits**

1. **Enhanced Stability**: Settlement Artifacts enhance the overall stability of the settlement process by acting as underlying proof of assets for LRT and LST. They provide a robust foundation that supports the secure and reliable execution of cross-chain transactions.
2. **Resilience to Imbalance**: Settlement Artifacts help maintain the functionality of Settlement Primitives even when there is a significant imbalance between LST/LRT and the underlying BTC assets. This resilience ensures that the settlement system can continue to operate smoothly under various market conditions.
3. **Transparency and Accountability**: The extended ERC20 implementation of Settlement Artifacts includes metadata that reveals the attributes of the Settlement Primitive that created them. This transparency allows users and developers to understand the underlying mechanisms and attributes of the artifacts, fostering trust and accountability within the system.
4. **Market-Driven Adaptability**: The market-driven value of Settlement Artifacts allows them to adapt to changing market conditions. This adaptability ensures that the artifacts remain relevant and valuable, providing ongoing support for the stability and security of the settlement process.
5. **Integration and Interoperability**: Settlement Artifacts can be easily integrated into various projects and applications developed on the Chakra Chain. The Settlement Artifacts Registry Contract provides a centralized source of information about registered artifacts, facilitating seamless integration and interoperability across different projects.
