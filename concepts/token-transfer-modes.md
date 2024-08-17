# Token Transfer Modes

Chakra supports two primary modes for cross-chain token transfer:

|      | Lock/Unlock                                                                 | Burn/Mint                                                    |
| ---- | --------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Pro  | Flexible (used with any token)                                              | No need to manage liquidity                                  |
| Cons | <ul><li>Liquidity on both ends</li><li>Locked token can be stolen</li></ul> | <ul><li>Token contracts need burn/mint permissions</li></ul> |

## Lock/Unlock Mode:

Tokens are locked on the source blockchain, and an equivalent amount of tokens are released on the target blockchain.

Key Implementation Snippets:

1. Implement `cross_chain_erc20_settlement` interface on the cross-chain initiation side, and transfer ERC20 from the caller to this contract

```solidity
IERC20(token).transferFrom(msg.sender, address(this), amount);
```

2. Implement `receive_cross_chain_msg` interface on the cross-chain receiving side. The receiver transfers from the handler contract to the target address

```solidity
IERC20(token).transfer(AddressCast.to_address(transfer_payload.to), transfer_payload.amount);

```

## Burn/Mint Mode:

Tokens are burned on the source blockchain, and an equivalent amount of tokens are minted on the target blockchain. The initiating side transfers the user's tokens to the handler contract, and the counterpart contract mints the tokens. Upon receiving the callback, the initiating side burns the tokens.

Key Implementation Snippets:

1. Implement `cross_chain_erc20_settlement` interface on the cross-chain initiation side to transfer ERC20 from the caller to this contract

```solidity
IERC20(token).transferFrom(msg.sender, address(this), amount);
```

2. Implement `receive_cross_chain_msg` interface on the cross-chain receiving side. The receiver has mint permissions, directly mint to the target address

```solidity
IERC20Mint(token).mint_to(AddressCast.to_address(transfer_payload.to), transfer_payload.amount);
```

3. Implement `receive_cross_chain_callback` interface on the cross-chain initiation side. The initiator has burn permissions, receive the processed callback, and directly burn the token

```solidity
IERC20Burn(token).burn(cross_chain_settlement_txs[txid].amount);
```
