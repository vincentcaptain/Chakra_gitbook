# Settlement Contract

## Introduction

The Settlement Contract is a standardized interface implemented for the Chakra Settlement Layer. Its primary purpose is to manage secure configurations and facilitate the sending and receiving of messages across different blockchain networks. Typically, the Settlement Contract is deployed by the Chakra Settlement Layer on various chains and is governed by the overarching Chakra Settlement management system.

For DApp developers, the key functionalities of the Settlement Contract to focus on are the contract interfaces for sending and receiving messages. These interfaces are designed to abstract the complexities of underlying processes, enabling developers to integrate and utilize the Chakra Settlement Layer with ease and efficiency.

The Settlement Contract ensures seamless communication between different blockchain networks, providing a reliable mechanism for cross-chain transactions. Handling the secure configuration and message transmission maintains the integrity and efficiency of the settlement process.

## `Settlement` Interface:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {ISettlementHandler} from "contracts/interfaces/ISettlementHandler.sol";
import {PayloadType} from "contracts/libraries/Message.sol";

interface ISettlement {
    /**
     * @notice Sends a cross-chain message to the specified chain.
     * @param to_chain The name or identifier of the destination chain.
     * @param from_address The address of the sender on the current chain.
     * @param to_handler The ID or address of the handler on the destination chain.
     * @param payload_type The type of data payload.
     * @param payload The data payload to be sent across the chain.
     */
    function send_cross_chain_msg(
        string memory to_chain,
        address from_address,
        uint256 to_handler,
        PayloadType payload_type,
        bytes calldata payload
    ) external;

    /**
     * @notice Receives a cross-chain message from the specified chain.
     * @param txid The unique identifier of the cross-chain transaction.
     * @param from_chain The name or identifier of the chain the message is coming from.
     * @param from_address The address of the sender on the source chain.
     * @param from_handler The ID or address of the handler on the source chain.
     * @param to_handler The recipient handler on the current chain.
     * @param payload_type The type of data payload.
     * @param payload The data payload received from the cross-chain transaction.
     * @param sign_type The type of signature used to verify the message, such as a multisig or BLS (Boneh-Lynn-Shacham) signature.
     * @param signatures The actual signatures used to verify the message.
     */
    function receive_cross_chain_msg(
        uint256 txid,
        string memory from_chain,
        address from_address,
        uint256 from_handler,
        ISettlementHandler to_handler,
        PayloadType payload_type,
        bytes calldata payload,
        uint8 sign_type, // validators signature type /  multisig or bls sr25519
        bytes calldata signatures
    ) external;
}
```

## Note

The Settlement Contract includes several security measures to ensure the integrity and security of cross-chain transactions:

* **Authorization**: Only authorized addresses can execute certain functions, ensuring only trusted entities can send or receive messages.
* **Signature Verification**: Messages are verified using multisig or BLS signatures, providing robust security against unauthorized tampering.
