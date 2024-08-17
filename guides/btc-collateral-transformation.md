# BTC Collateral Transformation

## Overview

The Chakra Settlement Layer enables the interchangeability between [settlement artifacts](../concepts/chakra-settlement-artifacts/), a proof of asset in Chakra, facilitating the transformation of BTC. This setup can ensure that BTC is securely managed throughout its lifecycle, whether in a custody or staking state. The BTC is not just represented differently on the blockchain but is **actually moved and managed** in a way that optimizes security and staking rewards.&#x20;

In this tutorial, you will implement a contract on Chakra to convert a Cobo BTC Settlement Artifact into a Cobo + Babylon Settlement Artifact.

## **Before you begin:**

1. The Chakra Settlement Layer is a fully EVM-compatible chain, so most of your development journey will revolve around EVM and Solidity. You should know how to [develop](https://soliditylang.org/) and [deploy](https://remix.ethereum.org/) Solidity smart contracts. You should also know how to use a wallet; [MetaMask](https://metamask.io/) is a good starting point.
2. As you will be developing on the Chakra Settlement Layer, you need to prepare some Chakra Settlement Layer Chain native tokens. Here's how to obtain them from the Chakra Settlement Layer faucet: \[Faucet to be added]

## Example: Converting Cobo BTC Settlement Artifacts to Cobo + Babylon Settlement Artifacts

Users can directly call the `convert_to_babylon_liquid_btc` function of the settlement contract through a frontend web page.

### Method 1: Through Interface

The contract on the Chakra Settlement Layer provides the interface.

**Function name**: `convert_to_babylon_liquid_btc`

| Parameter         | Type    | Remark                                                         | Example                                    |
| ----------------- | ------- | -------------------------------------------------------------- | ------------------------------------------ |
| amount            | uint256 | Token amount                                                   | 5000                                       |
| receiver\_address | address | Babylon Liquid BTC receiver address in Chakra Settlement Layer | 0x940D583861e57ab1c7F83D5a9450323CAe38402b |

#### **Step 1: Adding Chakra network to your wallet**

```javascript
const CHAKRA_CHAIN = {
  chain_id: '8545',
  chain_name: 'ChakraDevnet',
  rpc_url: 'https://rpcv1-dn-1.chakrachain.io',
  currency: 'CKA',
  blockExplorerUrls: ['https://explorer-dn.chakrachain.io/'],
  nativeCurrency: {
    name: 'CKA',
    symbol: 'CKA',
    decimals: 8,
  },
}

// Add Chakra network
await window.ethereum.request({
  method: 'wallet_addEthereumChain',
  params: [
    {
      chainId: hexChainId,
      chainName: CHAKRA_CHAIN.chain_name,
      blockExplorerUrls: CHAKRA_CHAIN.blockExplorerUrls,
      rpcUrls: [CHAKRA_CHAIN.rpc_url],
      nativeCurrency: CHAKRA_CHAIN.nativeCurrency,
    },
  ],
})
```

#### **Step 2: Calling contract method on Chakra network to convert ckrBTC to ckrFBTC**

```javascript
const receiveAddress = "<Chakra user address>"
const fromAddress = "<Chakra user address>"
const amountSat = 10000 
const handlerContractAddress = "<Settlement handler contract address>"
const ckrBTCTokenAddress = "<ckrBTC token address>"

try {
  const withdrawResult = await convert_to_babylon_liquid_btc(
    amountSat,
    receiveAddress
  )
  // show success dialog
} catch (error) {
  console.log(`convert Error`, error)
}
```

### Method 2: Through Smart Contract

You can call the `convert_to_babylon_liquid_btc` function of the settlement contract in your smart contract to convert Cobo BTC Settlement Artifacts to Cobo + Babylon Settlement Artifacts.

#### **Contract Code Example:**

> Note: This code is for demonstration purposes only. It has not been audited and should not be used directly.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract Deposit {
    address settlement_address;
    address cobo_btc_settlement_artiface;
    interface ISettlement{
        function convert_to_babylon_liquid_btc(uint256 amount, address rceiver_address);
    }

    event ConvertedToBabylonLiquidBTC(uint256 amount, address rceiver_address);

    constructor(
        address memory _settlement_address,
        address memory _cobo_btc_settlement_artiface
    ) {
        settlement_address = _settlement_address;
        cobo_btc_settlement_artiface = _cobo_btc_settlement_artiface;
    }

    function deposit(uint256 amount) external payable{
        IERC20(cobo_btc_settlement_artiface).transferFrom(msg.sender, address(this), amount);
        ISettlement(settlement_address).convert_to_babylon_liquid_btc(amount, address(this));
        emit ConvertedToBabylonLiquidBTC(amount, address(this));
    }
}
```

#### **Contract Deployment**

1. Configure the Chakra testnet chain in your project's hardhat.config.ts:

```javascript
networks: {
  chakra: {
    url: "https://rpcv1-dn-1.chakrachain.io",
    accounts: ["<Your private key in this chain>"],
  }
}
```

2. Write a deployment script:

```bash
mkdir scripts
touch scripts/deploy.js
```

Example of scripts/deploy.js:

```javascript
const {ethers} = require("hardhat");
const settlement_address = "0xdAC17F958D2ee523a2206206994597C13D831ec7";
async function main() {
    const Deposit = await ethers.getContractFactory("Deposit");
    await Deposit.deploy(settlement_address);
}

main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });
```

3. Deploy the contract to the Chakra network using Hardhat:

```bash
npx hardhat run scripts/deploy.js --network chakra
```

#### **Contract Interaction**

We'll use JavaScript as an example of interaction.

Install dependencies:

```bash
yarn add ethers
```

Write the interaction script:

```bash
touch deposit.js
```

Example of deposit.js (calling the `deposit` method):

```javascript
const { ethers } = require("ethers");

const RPC_PROVIDER_URL = "https://rpcv1-dn-1.chakrachain.io";

const provider = new ethers.providers.JsonRpcProvider(RPC_PROVIDER_URL);

const privateKey = "<YOUR_PRIVATE_KEY>";
const wallet = new ethers.Wallet(privateKey, provider);

const contractAddress = "<YOUR_CONTRACT_ADDRESS>";

const contractABI = [
    "function deposit() external payable"
];

const contract = new ethers.Contract(contractAddress, contractABI, wallet);

async function deposit() {
    try {
        const amount = 10000000000;

        const tx = await contract.deposit(amount);

        const receipt = await tx.wait();

        console.log("Transaction successful with hash:", receipt.transactionHash);
    } catch (error) {
        console.error("Error calling contract method:", error);
    }
}

deposit();
```

Execute the script:

```bash
node deposit.js
```

This concludes the tutorial on using the Chakra Settlement Layer to convert Cobo BTC Settlement Artifacts to Cobo + Babylon Settlement Artifacts.
