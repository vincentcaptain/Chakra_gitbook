---
coverY: 0
---

# Layer 2's and Execution Layer Integration

## Overview

Integrating Layer 2 (L2) solutions and execution layers with the Chakra Settlement Layer involves meeting specific technical requirements to ensure seamless functionality. This integration enables L2 solutions to leverage the extensive liquidity provided by LST and LRT projects, enhancing interoperability, security, and operational efficiency. The following sections outline the standard EVM requirements, finality considerations, RPC specifications, multi-signature wallet support, block explorer support, integration procedure, and the benefits for L2 solutions.

### Increased Liquidity

By integrating with the Chakra Settlement Layer, Layer 2 solutions can tap into the extensive liquidity provided by LST and LRT projects, enhancing their ecosystem’s liquidity.

### Enhanced Interoperability

Chakra’s compatibility with EVM and standardized RPCs ensures seamless integration, making it easier for Layer 2 solutions to interact with other blockchains and decentralized applications.

### Improved Security and Finality

The rigorous finality requirements and multi-signature wallet support enhance the security and reliability of transactions, ensuring that assets and data are securely managed.

### Access to a Robust Development Ecosystem

Layer 2 solutions benefit from Chakra’s extensive support for Solidity global variables, opcode implementation, and JSON-RPC specifications, facilitating efficient development and deployment of smart contracts and decentralized applications.

### Operational Efficiency

The requirement for high-performance, standardized RPCs ensures that Layer 2 solutions can maintain high uptime, low latency, and robust support, thereby improving operational efficiency and user experience.

### Comprehensive Support and Documentation

Chakra provides detailed documentation and support throughout the integration process, ensuring that Layer 2 solutions can seamlessly onboard and benefit from the enhanced functionality offered by the Chakra Settlement Layer.

## Requirements

For technical requirements before the integration, please see [here](standard-evm-requirements.md)

## How it works

The following outlines the collaboration process, including the time required for each step:

1. Project team sends an email to Chakra.
2. Chakra begins the review process.
3. After the review, a conference call is scheduled to discuss further.
4. Post-discussion, the testnet setup begins:
   1. Deployment of Settlement Contract.
   2. Deployment of test Settlement Handler Contract.
   3. Integration with Chakra Settlement Layer.
   4. Cross-chain token/message testing.
5. Upon successful testnet operation, the mainnet deployment begins.
