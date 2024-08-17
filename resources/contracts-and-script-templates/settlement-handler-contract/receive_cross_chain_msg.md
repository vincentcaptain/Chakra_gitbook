# Receive\_cross\_chain\_msg

#### Utility

Receive and process the cross-chain message on the destination chain

#### Parameter Description

| Parameter     | Type    | Description                                                                |
| ------------- | ------- | -------------------------------------------------------------------------- |
| txid          | uint256 | The unique transaction identifier of the cross-chain message.              |
| from\_chain   | string  | The identifier of the source chain from which the message originates.      |
| from\_address | uint256 | The sender's address on the source chain.                                  |
| from\_handler | uint256 | The handler or contract address on the source chain that sent the message. |
| payload       | bytes   | The data payload of the cross-chain message.                               |
| sign\_type    | uint8   | The type of signature used to verify the authenticity of the message.      |
| signatures    | bytes   | The cryptographic signatures used to authenticate the message.             |

The `receive_cross_chain_msg` function is likely part of a smart contract that is designed to handle messages coming from different blockchain networks. Here’s a brief explanation of each parameter:

* **`txid`:** An unsigned 256-bit integer representing the unique transaction identifier of the cross-chain message. This helps in tracking and verifying the message across different chains.
* **`from_chain`:** A string that identifies the blockchain network from which the message originates. For example, this could be "Ethereum" or "Binance Smart Chain".
* **`from_address`:** An unsigned 256-bit integer representing the sender's address on the source chain. This could be a standard address format encoded as a number.
* **`from_handler`:** An unsigned 256-bit integer representing the handler or contract address on the source chain that sent the message. This is useful when the message is sent through a specific contract rather than directly from an address.
* **`payload`:** The actual data payload of the message, which could contain any kind of information, such as instructions for the receiving contract or token transfer details.
* **`sign_type`:** An unsigned 8-bit integer indicating the type of cryptographic signature used to verify the authenticity of the message. Different types of signatures (e.g., ECDSA, Schnorr) may require different verification methods.
* **`signatures`:** A byte array containing one or more cryptographic signatures that prove the message was sent by the claimed sender. These signatures are verified using the specified `sign_type`.

#### Code template

```solidity
    function receive_cross_chain_msg(
        uint256 txid,
        string memory from_chain,
        uint256 from_address,
        uint256 from_handler,
        bytes calldata payload,
        PayloadType payload_type,
        uint8 sign_type,
        bytes calldata signatures
    ) external returns (bool) {

        require(
						payload_type == PayloadType.ERC20,
            "Invalid payload type"
        );

        bytes memory erc20_payload = MessageV1Codec.payload(payload);

        // Decode payload method
        ERC20Method method = codec.decode_method(erc20_payload);

        // Cross chain transfer
        {
            if (method == ERC20Method.Transfer) {
                // Decode transfer payload
                ERC20TransferPayload memory transfer_payload = codec
                    .deocde_transfer(erc20_payload);

                if (no_burn) {
                    require(
                        IERC20(token).balanceOf(address(this)) >=
                            transfer_payload.amount,
                        "Insufficient balance"
                    );

                    IERC20(token).transfer(
                        AddressCast.to_address(transfer_payload.to),
                        transfer_payload.amount
                    );
                } else {
                    IERC20Mint(token).mint_to(
                        AddressCast.to_address(transfer_payload.to),
                        transfer_payload.amount
                    );
                }

                return true;
            }
        }

        return false;
    }
```
