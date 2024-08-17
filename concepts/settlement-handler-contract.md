# Settlement Handler Contract

## Introduction

The Settlement Handler Contract is an essential component in the Chakra settlement layer network, responsible for managing BTC settlement artifacts and initiating BTC settlements on the Bitcoin mainnet. It acts as the intermediary that processes specific cross-chain messages and interacts with the Settlement Contract to ensure secure and efficient cross-chain communication.

The primary purpose of the Settlement Handler Contract is to facilitate the settlement of BTC artifacts by handling the necessary logic to lock or burn tokens on the source chain and trigger the settlement process. It ensures that cross-chain transactions are secure, verified, and efficiently processed, thereby maintaining the integrity and security of the settlement process.

## **SettlementHandler Interface**

Like the Settlement Interface, DApp Developers only need to care about the SettlementHandler Interface instead. The `ISettlementHandler` interface defines the methods required for handling cross-chain messages and callbacks. Below is the Solidity interface for `ISettlementHandler`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {CrossChainMsgStatus, PayloadType} from "contracts/libraries/Message.sol";

interface ISettlementHandler {
    /**
     * @notice Receives a cross-chain callback message and processes it.
     * @param txid The unique identifier of the cross-chain transaction.
     * @param from_chain The name or identifier of the chain the message is coming from.
     * @param from_handler The ID or address of the handler on the source chain.
     * @param status The status of the cross-chain message, such as success or failure.
     * @param sign_type The type of signature used to verify the message, such as a multisig or BLS (Boneh-Lynn-Shacham) signature.
     * @param signatures The actual signatures used to verify the message.
     * @return True if the callback message was successfully processed, false otherwise.
     */
    function receive_cross_chain_callback(
        uint256 txid,
        string memory from_chain,
        uint256 from_handler,
        CrossChainMsgStatus status,
        uint8 sign_type, // validators signature type /  multisig or bls sr25519
        bytes calldata signatures
    ) external returns (bool);

    /**
     * @notice Receives a cross-chain message and processes it.
     * @param txid The unique identifier of the cross-chain transaction.
     * @param from_chain The name or identifier of the chain the message is coming from.
     * @param from_address The address of the sender on the source chain.
     * @param from_handler The ID or address of the handler on the source chain.
     * @param payload The data payload received from the cross-chain transaction.
     * @param sign_type The type of signature used to verify the message, such as a multisig or BLS (Boneh-Lynn-Shacham) signature.
     * @param signatures The actual signatures used to verify the message.
     * @return True if the message was successfully processed, false otherwise.
     */
    function receive_cross_chain_msg(
        uint256 txid,
        string memory from_chain,
        uint256 from_address,
        uint256 from_handler,
        bytes calldata payload,
        uint8 sign_type,
        bytes calldata signatures
    ) external returns (bool);
}

```

You can find the step-by-step guide [here](../guides/cross-chain-transfer-with-settlement-layer.md) with an example code snippet on how to implement the contract.&#x20;
