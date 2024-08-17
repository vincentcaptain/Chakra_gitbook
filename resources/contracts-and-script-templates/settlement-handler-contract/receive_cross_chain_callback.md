# Receive\_cross\_chain\_callback

#### Utility

Receive the callback event on the chain and burn the corresponding token

#### Parameter Description

| Parameter      | Type                  | Description                                                                                           |
| -------------- | --------------------- | ----------------------------------------------------------------------------------------------------- |
| `txid`         | `uint256`             | Unique identifier for the transaction or message being sent across chains.                            |
| `from_chain`   | `string`              | Identifier of the chain where the message originated.                                                 |
| `from_handler` | `uint256`             | Identifier for the entity (e.g., contract address) on the originating chain that sent the message.    |
| `status`       | `CrossChainMsgStatus` | Enum representing the status of the cross-chain message (e.g., success, failure).                     |
| `sign_type`    | `uint8`               | The type of signature used by the validators to sign the message (e.g., single validator, multi-sig). |
| `signatures`   | `bytes`               | The actual signature data used to verify the authenticity of the message.                             |

The `receive_cross_chain_msg` function is likely part of a smart contract that is designed to handle messages coming from different blockchain networks. Here’s a brief explanation of each parameter:

* **`txid`:** An unsigned 256-bit integer representing the unique transaction identifier of the cross-chain message. This helps in tracking and verifying the message across different chains.
* **`from_chain`:** A string that identifies the blockchain network from which the message originates. For example, this could be "Ethereum" or "Binance Smart Chain".
* **`from_handler`:** An unsigned 256-bit integer representing the handler or contract address on the source chain that sent the message. This is useful when the message is sent through a specific contract rather than directly from an address.
* **`status`**: An Enum that represents the status of the cross-chain message. For example, this could be "Success" or "Failure".&#x20;
* **`sign_type`:** An unsigned 8-bit integer indicating the type of cryptographic signature used to verify the authenticity of the message. Different types of signatures (e.g., ECDSA, Schnorr) may require different verification methods.
* **`signatures`:** A byte array containing one or more cryptographic signatures that prove the message was sent by the claimed sender. These signatures are verified using the specified `sign_type`.

#### Code template

```solidity
 function receive_cross_chain_callback(
        uint256 txid,
        string memory from_chain,
        uint256 from_handler,
        CrossChainMsgStatus status,
        uint8 /*sign_type*/, // validators signature type /  multisig
        bytes /*calldata signatures */
    ) external returns (bool) {
        require(
            keccak256(
                abi.encodePacked(cross_chain_settlement_txs[txid].to_chain)
            ) == keccak256(abi.encodePacked(from_chain)),
            "Invalid from chain"
        );

        require(
            cross_chain_settlement_txs[txid].to_handler == from_handler,
            "Invalid from handler"
        );

        bytes32 message_hash = keccak256(
            abi.encodePacked(txid, from_handler, address(this), status)
        );

        if (no_burn) {
            IERC20Burn(token).burn(cross_chain_settlement_txs[txid].amount);
        }

        cross_chain_settlement_txs[txid].status = CrossChainTxStatus.Settled;

        return true;
    }
```
