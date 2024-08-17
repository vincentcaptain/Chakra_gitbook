# Standard EVM Requirements

Before an EVM blockchain network can integrate with the Chakra Settlement Layer, it must meet certain technical requirements. These requirements are critical for Chakra Settlement Layer validator nodes and Chakra settlement services to function correctly on a given network.

## Solidity Global Variables and Opcode Implementation

Solidity global variables and opcode implementation constructs must meet the following requirements for Chakra services to operate correctly:

* **Global variables:** Support all global variables as specified in the Solidity [Block and Transaction Properties](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#block-and-transaction-properties) documentation. Examples include:
  * **`block.blockhash`** must return the hash of the requested block for the last 256 blocks.
  * **`block.number`** must return the respective chain's block number.
  * **`block.chainID`** must return the current chain ID.
  * **`block.timestamp`** must return the current block timestamp in seconds since the Unix epoch.
* **Precompiles:** Must support all precompile contracts and expected behaviors listed in the Solidity [Mathematical and Cryptographic Functions](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#mathematical-and-cryptographic-functions) documentation.
* **Contract size:** Must support the maximum contract size defined in [EIP-170](https://eips.ethereum.org/EIPS/eip-170).
* **Nonce:** The transaction nonce must increase as transactions are confirmed and propagated to all nodes in the network.

## Finality

Blockchain development teams must ensure that blocks with a commitment level of _finalized_ are genuinely final. The properties of the finality mechanism, including underlying assumptions and conditions under which finality violations could occur, must be clearly documented and communicated to application developers in the blockchain ecosystem.

Furthermore, this information should be accessible through RPC API tags _finalized_ from the [JSON-RPC specification tags](https://www.notion.so/Settlement-Layer-2-0670828876f6419fa32f864f95a917f0?pvs=21) described later in this document.

## Standardized RPCs with SLAs

Chakra nodes use RPCs to communicate with the chain and perform soak testing. Ensuring the functionality of Chakra services requires stable and performant RPCs. The requirements are:

**Dedicated RPC Node:**

* The chain must provide instructions and hardware requirements to set up and run a full node.
* The archive node setup must also be provided and allow queries of blocks from genesis with transaction history and logs.
* The RPC node must enable and allow configurable settings for batch calls, log lookbacks, HTTPS, and WSS connections.

**RPC Providers:**

* Three separate independent RPC providers must be available.
* RPC providers must ensure there is no rate limit.
* RPC providers must have a valid SSL certificate.
* During the trailing 30 days, the RPC providers must meet the following RPC performance requirements:
  * Uptime: At least 99.9%
  * Throughput: Support at least 300 calls per second
  * Latency: Less than 250ms
  * Support SLA: For SEV1 issues, provide a Time to Answer (TTA) of at most 1 hour

## Support the Ethereum JSON-RPC Specification

The chain must support the [Ethereum JSON-RPC Specification](https://ethereum.org/en/developers/docs/apis/json-rpc). Chakra services use several methods to operate on the chain and require a specific response format to those calls. The following methods are required and must adhere to the [Ethereum RPC API specification](https://ethereum.github.io/execution-apis/api-documentation/):

* [GetCode](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_getcode)
* [Call](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_call)
* [ChainID](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_chainid)
* [SendTransaction](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_sendtransaction)
* [SendRawTransaction](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_sendrawtransaction)
* [GetTransactionReceipt](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_gettransactionreceipt)
* [GetTransactionByHash](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_gettransactionbyhash)
* [EstimateGas](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_estimategas)
* [GasPrice](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_gasprice)
* [GetTransactionCount](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_gettransactioncount)
* [GetLogs](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_getlogs)
  * Must follow the spec as defined in [EIP-1474](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1474.md?plain=1#L856). The "latest" block number returned by **`GetBlockByNumber`** must also be served by **`GetLogs`** with logs.
  * Must accept the **`blockhash`** param as defined in [EIP-234](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-234.md)
* [GetBalance](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_getbalance)
* [GetBlockByNumber](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_getblockbynumber)
* [GetBlockByHash](https://ethereum.org/en/developers/docs/apis/json-rpc#eth\_getblockbyhash)

The network must also support:

* **Subscription Methods:** [Websocket JSON-RPC subscription methods](https://geth.ethereum.org/docs/interacting-with-geth/rpc/pubsub)
  * **`eth_subscribe`** with support for subscription to **`newHeads`** and **`logs`**
* **Tags:** The RPC methods must support the **`finalized`**, **`latest`**, and **`pending`** tags where applicable. They must also support natural numbers for blocks.
* **Batch Requests:** Must support batching of requests for the **`GetLogs`** and **`GetBlockByNumber`** methods.
* **Response size:** Any RPC request, including the batch requests, must be within the allowed size limit of around 173MB.

## Multi-signature Wallet Support

The chain must provide a supported and audited multi-signature wallet implementation with a UI.

## Block Explorer Support

The chain must provide a block explorer and support for contract and verification APIs.
