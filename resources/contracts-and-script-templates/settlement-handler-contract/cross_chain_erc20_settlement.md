# Cross\_chain\_erc20\_settlement

#### Utility

Receive and process cross-chain transfer request from caller

#### Parameter Description

| Parameter    | Type      | Description                                                                |
| ------------ | --------- | -------------------------------------------------------------------------- |
| `to_chain`   | `string`  | The identifier of the target chain where the settlement will take place.   |
| `to_handler` | `uint256` | The handler or contract address on the target chain to receive the tokens. |
| `to_token`   | `uint256` | The identifier of the ERC-20 token being transferred.                      |
| `to`         | `uint256` | The recipient address on the target chain.                                 |
| `amount`     | `uint256` | The amount of the ERC-20 token to be transferred.                          |

The `cross_chain_erc20_settlement` function is likely part of a smart contract that facilitates cross-chain transfers of ERC-20 tokens. Here’s a brief explanation of each parameter:

* **`to_chain`:** A string representing the destination blockchain network. For example, if you are transferring from Ethereum to Binance Smart Chain, this would be the identifier for Binance Smart Chain.
* **`to_handler`:** An unsigned 256-bit integer (usually representing an address in some blockchains) that indicates the specific contract or entity on the destination chain responsible for handling the incoming tokens.
* **`to_token`:** An unsigned 256-bit integer that identifies the ERC-20 token being transferred. This could be a token ID or a contract address on the source chain.
* **`to`:** The recipient's address on the destination chain. This is typically another unsigned 256-bit integer that represents an address.

#### Code Template

```solidity

    function cross_chain_erc20_settlement(
        string memory to_chain,
        uint256 to_handler,
        uint256 to_token,
        uint256 to,
        uint256 amount
    ) external {
        require(amount > 0, "Amount must be greater than 0");
        require(to != 0, "Invalid to address");
        require(to_handler != 0, "Invalid to handler address");
        require(to_token != 0, "Invalid to token address");

        require(
            IERC20(token).balanceOf(msg.sender) >= amount,
            "Insufficient balance"
        );

        // transfer tokens
        IERC20(token).transferFrom(msg.sender, address(this), amount);

        uint256 cross_chain_msg_id = uint256(
            keccak256(
                abi.encodePacked(
                    cross_chain_msg_id_counter,
                    address(this),
                    msg.sender,
                    nonce_manager[msg.sender]
                )
            )
        );

        // Create a erc20 transfer payload
        ERC20TransferPayload memory payload = ERC20TransferPayload(
            ERC20Method.Transfer,
            AddressCast.to_uint256(msg.sender),
            to,
            AddressCast.to_uint256(token),
            to_token,
            amount
        );
        // Create a cross chain msg
        Message memory cross_chain_msg = Message(
            cross_chain_msg_id,
            PayloadType.ERC20,
            codec.encode_transfer(payload)
        );

        // Encode the cross chain msg
        bytes memory cross_chain_msg_bytes = MessageV1Codec.encode(
            cross_chain_msg
        );

				// Send the cross chain msg
        settlement.send_cross_chain_msg(
            to_chain,
            msg.sender,
            to_handler,
            PayloadType.ERC20,
            cross_chain_msg_bytes
        );
     }
```
