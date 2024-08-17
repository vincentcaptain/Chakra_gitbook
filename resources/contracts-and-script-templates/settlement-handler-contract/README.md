# Settlement Handler Contract

## Cross\_Chain\_erc20\_settlement

Utility: receive and process cross-chain transfer request from caller

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
