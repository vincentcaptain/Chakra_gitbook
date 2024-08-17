# Estimate and pay gas

## **Prepaid Gas**

Unlike other chains, Chakra enhances user experience by using multi-chain gas estimation. The Chakra network routes each cross-chain settlement message to its target chain, and at the time of transaction initiation, users obtain the required gas for multiple chains through the Chakra gas service. This includes:

* Gas to be prepaid on the source chain
* Gas to be prepaid on the destination chain

## **Gas Refunds**

After the cross-chain settlement message execution is completed, Chakra calculates the actual gas used and refunds the excess to the payer's account.

## **Two-Way Calls**

Chakra supports a callback mode where cross-chain messages can travel from the source chain to the destination chain and then back to the source chain. For two-way calls, users need to prepay gas for both directions on the source chain. Once the transaction is executed on the destination chain, Chakra will callback the result to the source chain using the fees paid on the source chain.

## Fee Estimates

The `GasPriceService` contract maintains the gas prices for each chain, providing stable exchange rates for tokens used as gas across different chains. This service is based on oracle price feeds.

The `GasEstimationService` is a service provided by Chakra that estimates the gas fees for multiple chains and returns the gas amount required for each chain. When initiating a cross-chain transaction, users can use the gas amounts returned by `GasEstimationService` as the gas limit and calculate the required gas fee using the following formula:

$$
\text{totalGasFee} = \text{gasLimit} \times (\text{Base Fee} + \text{Priority Fee})
$$

Users pay the gas fee through the `SettlementHandler` contract, which then pays the gas fee to the `Settlement` contract.

### Base Fee & Priority Fee

Base Fee and Priority Fee are important concepts introduced in Ethereum's EIP 1559 upgrade, where:&#x20;

* **Base Fee**:
  * Automatically set by the network as the minimum gas fee.
  * Dynamically adjusts based on network congestion.
  * Must be paid by all transactions.
  * This portion of the fee is burned and does not go to miners.
* **Priority Fee**:
  * Optional additional fee users can pay.
  * Acts as an incentive for miners to prioritize processing the transaction.
  * Also known as a "tip".

Any EVM-compatible chain implementing EIP 1559 can obtain the base fee and maximum priority fee (max priority fee). Below is the process for obtaining and calculating these fees:

#### Getting and Calculating Base Fee:

Ethereum node RPC does not provide a direct API to get the base fee since it dynamically adjusts based on blocks. By using the `eth_getBlockByNumber` RPC to get the block header, you can calculate the base fee according to EIP 1559 specifications. Here is a `Go` implementation for calculating the base fee:

```go
package main

import (
    "context"
    "fmt"
    "math/big"

    "github.com/ethereum/go-ethereum/consensus/misc"
    "github.com/ethereum/go-ethereum/ethclient"
    "github.com/ethereum/go-ethereum/params"
)

const ethJSONRPCEndpointAddress = "http://path-to-json-rpc:8545"

func main() {
    config := params.MainnetChainConfig
    ethClient, _ := ethclient.DialContext(context.Background(), ethJSONRPCEndpointAddress)
    bn, _ := ethClient.BlockNumber(context.Background())

    bignumBn := big.NewInt(0).SetUint64(bn)
    blk, _ := ethClient.BlockByNumber(context.Background(), bignumBn)
    baseFee := misc.CalcBaseFee(config, blk.Header())
    fmt.Printf("Base fee for block %d is %s\n", bn+1, baseFee.String())
}

```

#### **Getting Max Priority Fee:**

Use the `eth_maxPriorityFeePerGas` RPC to get the max priority fee in wei:

```bash
# WIP for Chakra integration with Infura, DeFillama and others
# curl https://celo-mainnet.infura.io/v3/YOUR-API-KEY \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "eth_maxPriorityFeePerGas", "params": [], "id": 1}'

```

Users pay the gas fee by initiating the cross-chain settlement transaction through the `SettlementHandler` contract, which then pays the gas fee to the `Settlement` contract.

## **Gas Service Workflow**

The figure below illustrates the process of Gas Service Workflow on Chakra. The detail process is listed below:&#x20;

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

1. **Gas Estimation**:
   * Users interact with the `GasEstimationService` to obtain the gas amount required for each chain involved in the transaction.
   * The `GasPriceService` uses oracle feeds to provide up-to-date gas prices, ensuring stable exchange rates for gas tokens.
2. **Transaction Initiation**:
   * Users initiate the cross-chain settlement transaction through the `SettlementHandler` contract.
   * The required gas fee, as estimated earlier, is paid to the `Settlement` contract.
3. **Message Routing and Execution**:
   * The Chakra settlement layer routes the cross-chain settlement message to the target chain.
   * Upon successful execution, the actual gas used is calculated, and any excess gas is refunded to the user's account.
4. **Two-Way Transactions**:
   * Users prepay the gas for both directions on the source chain for two-way transactions.
   * After execution on the destination chain, the result is callbacked to the source chain, utilizing the prepaid fees.
