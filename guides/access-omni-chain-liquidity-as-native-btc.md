# Access Omni-chain Liquidity as Native BTC

## Overview

This tutorial guides you through implementing a seamless user flow from Bitcoin (BTC) to ERC-20 tokens using the Chakra Settlement Layer. Protocols can securely and transparently access real and native Cross-Chain Interoperability and expand their DeFi Access with Enhanced Liquidity**.**

This tutorial will walk you through each implementation step, from setting up the necessary services to deploying smart contracts and initiating cross-chain settlements. By following this guide, you'll understand cross-chain interactions deeply and be well-equipped to develop your own cross-chain applications.

## Before you Start

Before beginning this tutorial, ensure you have the following:

* **Go Programming Environment**: You should have Go version 1.21 or later installed. Go is used to implement the backend services that handle BTC transactions and interact with the Chakra Settlement Layer.
* **Solidity Development Environment**: Familiarity with Solidity and a development framework like Hardhat is required. You'll be writing and deploying smart contracts for the Chakra Settlement Layer and the target chain.
* **Access to Blockchain Nodes**:
  * A Bitcoin node or access to a Bitcoin RPC endpoint for monitoring BTC transactions.
  * Access to a Chakra Settlement Layer RPC endpoint.
  * Access the target chain's RPC endpoint (e.g., Scroll Sepolia in this tutorial).
* **Basic Blockchain Knowledge**: Fundamental understanding of how Bitcoin and Ethereum-compatible blockchains work, including concepts like transactions, smart contracts, and cross-chain communication.
* **Wallet and Funds**:
  * A Bitcoin wallet with some testnet BTC for testing.
  * An Ethereum-compatible wallet (like MetaMask) with some test ETH on the Chakra network and the target chain.
* **Development Tools**:
  * A code editor (e.g., Visual Studio Code, Sublime Text)
  * Git for version control
  * Node.js and npm for managing JavaScript dependencies
* **API Keys and Credentials**: If you're using third-party services for blockchain interactions (like Infura for Ethereum), ensure you have the necessary API keys and credentials.

## Step-by-step Tutorial

### Step 1: Set Up BTC Backend Service

The BTC Backend Service is responsible for monitoring Bitcoin transactions and handling withdrawals. We'll implement two main components: the BTC Deposit Indexer and the BTC Withdraw Service.

#### 1.1 Create the project structure

```bash
mkdir BTCBackendService
cd BTCBackendService
go mod init btcbackendservice
```

#### 1.2 Implement BTC Deposit Indexer

Create a file named `btc_deposit_indexer.go` with the following content:

```go
package main

import (
    "encoding/hex"
    "fmt"
    "log"
    "github.com/btcsuite/btcd/btcec"
    "github.com/btcsuite/btcd/btcjson"
    "github.com/btcsuite/btcd/rpcclient"
    "github.com/btcsuite/btcd/wire"
    "github.com/btcsuite/btcutil"
)

const btcAddress = "your_btc_address_here"

var connCfg = &rpcclient.ConnConfig{
    Host:         "your_btc_node_rpc_url",
    User:         "your_rpc_username",
    Pass:         "your_rpc_password",
    HTTPPostMode: true,
    DisableTLS:   true,
}

func main() {
    client, err := rpcclient.New(connCfg, nil)
    if err != nil {
        log.Fatalf("Error creating new BTC RPC client: %v", err)
    }
    defer client.Shutdown()

    client.NotifyNewTransactions(true)

    log.Println("Listening for transactions...")
    for {
        select {
        case tx := <-client.NotifyTx():
            processTransaction(client, tx)
        }
    }
}

func processTransaction(client *rpcclient.Client, tx *btcutil.Tx) {
    txHash := tx.Hash()
    rawTx, err := client.GetRawTransactionVerbose(txHash)
    if err != nil {
        log.Printf("Failed to get raw transaction: %v", err)
        return
    }

    for _, vout := range rawTx.Vout {
        for _, addr := range vout.ScriptPubKey.Addresses {
            if addr == btcAddress {
                log.Printf("Transaction %s sent to monitored address", txHash)
                checkOpReturn(tx)
            }
        }
    }
}

func checkOpReturn(tx *btcutil.Tx) {
    for _, txOut := range tx.MsgTx().TxOut {
        script := txOut.PkScript
        if script[0] == 0x6a {
            opReturnData := script[2:]
            data, err := hex.DecodeString(hex.EncodeToString(opReturnData))
            if err != nil {
                log.Printf("Failed to decode OP_RETURN data: %v", err)
                return
            }
            parseOpReturnData(data)
        }
    }
}

func parseOpReturnData(data []byte) {
    if len(data) < 20 {
        log.Println("Invalid OP_RETURN data")
        return
    }
    ethAddress := data[:20]
    amount := string(data[20:])
    log.Printf("Extracted ETH Address: %s", hex.EncodeToString(ethAddress))
    log.Printf("Extracted Amount: %s", amount)
    // Here you would typically call a function to mint the Settlement Artifact
    // mintSettlementArtifact(ethAddress, amount)
}
```

#### 1.3 Implement BTC Withdraw Service

Create a file named `btc_withdraw_service.go` with the following content:

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/big"
    "github.com/btcsuite/btcd/btcec"
    "github.com/btcsuite/btcd/btcjson"
    "github.com/btcsuite/btcd/chaincfg"
    "github.com/btcsuite/btcd/rpcclient"
    "github.com/btcsuite/btcd/wire"
    "github.com/btcsuite/btcutil"
)

type WithdrawService struct {
    client *rpcclient.Client
}

func NewWithdrawService(client *rpcclient.Client) *WithdrawService {
    return &WithdrawService{client: client}
}

func (s *WithdrawService) Withdraw(userAddress string, amount btcutil.Amount) (string, error) {
    addr, err := btcutil.DecodeAddress(userAddress, &chaincfg.MainNetParams)
    if err != nil {
        return "", fmt.Errorf("invalid BTC address: %v", err)
    }

    tx, err := s.createWithdrawTransaction(addr, amount)
    if err != nil {
        return "", fmt.Errorf("failed to create transaction: %v", err)
    }

    signedTx, err := s.signTransaction(tx)
    if err != nil {
        return "", fmt.Errorf("failed to sign transaction: %v", err)
    }

    txHash, err := s.client.SendRawTransaction(signedTx, false)
    if err != nil {
        return "", fmt.Errorf("failed to broadcast transaction: %v", err)
    }

    return txHash.String(), nil
}

func (s *WithdrawService) createWithdrawTransaction(addr btcutil.Address, amount btcutil.Amount) (*wire.MsgTx, error) {
    tx := wire.NewMsgTx(wire.TxVersion)

    utxoHash, err := chainhash.NewHashFromStr("your-utxo-hash")
    if err != nil {
        return nil, fmt.Errorf("invalid UTXO hash: %v", err)
    }
    outPoint := wire.NewOutPoint(utxoHash, 0)

    txIn := wire.NewTxIn(outPoint, nil, nil)
    tx.AddTxIn(txIn)

    txOut := wire.NewTxOut(int64(amount), addr.ScriptAddress())
    tx.AddTxOut(txOut)

    return tx, nil
}

func (s *WithdrawService) signTransaction(tx *wire.MsgTx) (*wire.MsgTx, error) {
    privKey, _ := btcec.PrivKeyFromBytes(btcec.S256(), []byte("your-private-key"))

    for i, txIn := range tx.TxIn {
        signature, err := txscript.SignatureScript(tx, i, txIn.SignatureScript, txscript.SigHashAll, privKey, true)
        if err != nil {
            return nil, fmt.Errorf("failed to sign input: %v", err)
        }
        txIn.SignatureScript = signature
    }

    return tx, nil
}

func main() {
    connCfg := &rpcclient.ConnConfig{
        Host:         "your_btc_node_rpc_url",
        User:         "yourrpcuser",
        Pass:         "yourrpcpassword",
        HTTPPostMode: true,
        DisableTLS:   true,
    }
    client, err := rpcclient.New(connCfg, nil)
    if err != nil {
        log.Fatalf("Failed to connect to Bitcoin Core: %v", err)
    }
    defer client.Shutdown()

    service := NewWithdrawService(client)

    txHash, err := service.Withdraw("user-btc-address", btcutil.Amount(1000000))
    if err != nil {
        log.Fatalf("Failed to withdraw BTC: %v", err)
    }
    log.Printf("Transaction sent successfully! TX Hash: %s\n", txHash)
}
```

### Step 2: Develop Chakra Backend Service

The Chakra Backend Service will handle interactions with the Chakra Settlement Layer, including monitoring for events and minting Settlement Artifacts.

#### 2.1 Create the project structure

```bash
mkdir ChakraBackendService
cd ChakraBackendService
go mod init chakrabackendservice
```

#### 2.2 Implement Chakra Settlement Artifact Indexer

Create a file named `chakra_artifact_indexer.go` with the following content:

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/big"
    "strings"
    "github.com/ethereum/go-ethereum/accounts/abi"
    "github.com/ethereum/go-ethereum/common"
    "github.com/ethereum/go-ethereum/ethclient"
    "github.com/ethereum/go-ethereum/core/types"
)

const ContractABI = `[{"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"account","type":"address"},{"indexed":false,"internalType":"uint256","name":"amount","type":"uint256"}],"name":"Withdraw","type":"event"}]`

var ContractAddress = common.HexToAddress("0xYourContractAddressHere")

type EventWithdraw struct {
    BTCAddress string
    Amount     *big.Int
}

func main() {
    client, err := ethclient.Dial("Chakra Settlement Layer Node URL")
    if err != nil {
        log.Fatalf("Failed to connect to the Ethereum client: %v", err)
    }

    contractAbi, err := abi.JSON(strings.NewReader(ContractABI))
    if err != nil {
        log.Fatalf("Failed to parse contract ABI: %v", err)
    }

    query := ethereum.FilterQuery{
        Addresses: []common.Address{ContractAddress},
        Topics:    [][]common.Hash{{contractAbi.Events["Withdraw"].ID}},
    }

    logs := make(chan types.Log)
    sub, err := client.SubscribeFilterLogs(context.Background(), query, logs)
    if err != nil {
        log.Fatalf("Failed to subscribe to logs: %v", err)
    }

    log.Println("Listening for Withdraw events...")
    for {
        select {
        case err := <-sub.Err():
            log.Fatalf("Error received from subscription: %v", err)
        case vLog := <-logs:
            var event EventWithdraw
            err := contractAbi.UnpackIntoInterface(&event, "Withdraw", vLog.Data)
            if err != nil {
                log.Printf("Failed to unpack event: %v", err)
                continue
            }
            fmt.Printf("Withdraw Event - BTC Address: %s, Amount: %s\n", event.BTCAddress, event.Amount.String())
            // Here you would typically call a function to initiate the BTC withdrawal
            // initiateWithdrawal(event.BTCAddress, event.Amount)
        }
    }
}
```

#### 2.3 Implement Chakra Settlement Artifact Mint Service

Create a file named `chakra_artifact_mint_service.go` with the following content:

```go
package main

import (
    "context"
    "crypto/ecdsa"
    "fmt"
    "log"
    "math/big"
    "strings"
    "github.com/ethereum/go-ethereum"
    "github.com/ethereum/go-ethereum/accounts/abi"
    "github.com/ethereum/go-ethereum/accounts/abi/bind"
    "github.com/ethereum/go-ethereum/common"
    "github.com/ethereum/go-ethereum/crypto"
    "github.com/ethereum/go-ethereum/ethclient"
)

func main() {
    client, err := ethclient.Dial("Chakra Settlement Layer Node URL")
    if err != nil {
        log.Fatalf("Failed to connect to the client: %v", err)
    }

    contractAbi, err := abi.JSON(strings.NewReader(`[{"inputs":[{"internalType":"address","name":"to","type":"address"},{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"mint","outputs":[],"stateMutability":"nonpayable","type":"function"}]`))
    if err != nil {
        log.Fatalf("Failed to parse the contract ABI: %v", err)
    }

    contractAddress := common.HexToAddress("0xYourContractAddressHere")

    privateKey, err := crypto.HexToECDSA("YOUR_PRIVATE_KEY")
    if err != nil {
        log.Fatalf("Failed to load private key: %v", err)
    }

    publicKey := privateKey.Public()
    publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
    if !ok {
        log.Fatalf("Failed to get public key from private key")
    }
    fromAddress := crypto.PubkeyToAddress(*publicKeyECDSA)

    nonce, err := client.PendingNonceAt(context.Background(), fromAddress)
    if err != nil {
        log.Fatalf("Failed to get nonce: %v", err)
    }

    gasPrice, err := client.SuggestGasPrice(context.Background())
    if err != nil {
        log.Fatalf("Failed to get gas price: %v", err)
    }

    auth, err := bind.NewKeyedTransactorWithChainID(privateKey, big.NewInt(1) /* replace with actual chain ID */)
    if err != nil {
        log.Fatalf("Failed to create transaction auth: %v", err)
    }
    auth.Nonce = big.NewInt(int64(nonce))
    auth.Value = big.NewInt(0)
    auth.GasLimit = uint64(300000)
    auth.GasPrice = gasPrice

    toAddress := common.HexToAddress("0xRecipientAddressHere")
    amount := big.NewInt(1000000000000000000) // 1 token (assuming 18 decimals)

    data, err := contractAbi.Pack("mint", toAddress, amount)
    if err != nil {
        log.Fatalf("Failed to pack function call: %v", err)
    }

    tx := ethereum.CallMsg{
        To:   &contractAddress,
        Data: data,
    }

    gasLimit, err := client.EstimateGas(context.Background(), tx)
    if err != nil {
        log.Fatalf("Failed to estimate gas: %v", err)
    }
    auth.GasLimit = gasLimit

    txHash, err := client.SendTransaction(context.Background(), auth)
    if err != nil {
        log.Fatalf("Failed to send transaction: %v", err)
    }
    fmt.Printf("Transaction sent: %s\n", txHash.Hex())
}
```

### Step 3: Implement Settlement Artifact Contract

The Settlement Artifact Contract is a crucial component that represents the tokenized BTC on the Chakra Settlement Layer.

#### 3.1 Set up the Solidity project

If you haven't already set up your Solidity project, do so now:

```bash
mkdir SettlementArtifact
cd SettlementArtifact
npm init -y
npm install --save-dev hardhat @openzeppelin/contracts-upgradeable
npx hardhat init
```

Choose "Create a JavaScript project" when prompted.

#### 3.2 Create the Settlement Artifact Contract

Create a file named `BTCSettlementArtifact.sol` in the `contracts` folder with the following content:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

interface ISettlementArtifact {
    function artifact_metadata() external view returns (bytes memory);
    function mint_to(address account, uint256 value) external;
    function burn_from(address account, uint256 value) external;
}

contract BTCSettlementArtifact is Initializable, ERC20Upgradeable, OwnableUpgradeable, UUPSUpgradeable, ISettlementArtifact {
    bytes private _metadata;
    
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(address initialOwner, string memory name, string memory symbol, bytes memory metadata) initializer public {
        __ERC20_init(name, symbol);
        __Ownable_init(initialOwner);
        __UUPSUpgradeable_init();
        _metadata = metadata;
    }

    function artifact_metadata() external view override returns (bytes memory) {
        return _metadata;
    }

    function mint_to(address account, uint256 value) external override onlyOwner {
        _mint(account, value);
    }

    function burn_from(address account, uint256 value) external override onlyOwner {
        _burn(account, value);
    }

    function _authorizeUpgrade(address newImplementation)
        internal
        onlyOwner
        override
    {}
}
```

This contract implements the `ISettlementArtifact` interface and includes the required functionality for minting, burning, and storing metadata.

#### 3.3 Create a deployment script

Create a file named `deploy_settlement_artifact.js` in the `scripts` folder:

```javascript
const { ethers, upgrades } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const BTCSettlementArtifact = await ethers.getContractFactory("BTCSettlementArtifact");
  
  const metadata = ethers.utils.defaultAbiCoder.encode(
    ["string", "string", "string"],
    ["SecurityMeasureMents", "OperationalStatus", "Interchangeability"],
    ["MultiSigWallet(3/5)", "AverageLatency(200ms)", "Protocolstandards(ERC-20)"]
  );

  const artifact = await upgrades.deployProxy(BTCSettlementArtifact, [
    deployer.address,
    "BTC Settlement Artifact",
    "BTCSA",
    metadata
  ]);

  await artifact.deployed();

  console.log("BTCSettlementArtifact deployed to:", artifact.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### Step 4: Deploy Settlement Artifact Contract

Now that we have implemented our Settlement Artifact Contract, let's deploy it to the Chakra network.

#### 4.1 Configure Hardhat for Chakra network

Update your `hardhat.config.js` file to include the Chakra network configuration:

```javascript
require("@nomiclabs/hardhat-waffle");
require('@openzeppelin/hardhat-upgrades');
require("dotenv").config();

module.exports = {
  solidity: "0.8.24",
  networks: {
    chakra: {
      url: "https://rpcv1-dn-1.chakrachain.io",
      accounts: [process.env.PRIVATE_KEY],
      gas: "auto",
      gasPrice: "auto"
    }
  }
};
```

Make sure to create a `.env` file in your project root and add your private key:

```
PRIVATE_KEY=your_private_key_here
```

#### 4.2 Deploy the contract

Run the deployment script:

```bash
npx hardhat run scripts/deploy_settlement_artifact.js --network chakra
```

This will deploy your BTCSettlementArtifact contract to the Chakra network.

#### 4.3 Verify the contract (optional)

If the Chakra network supports contract verification (like Etherscan for Ethereum), you can verify your contract for transparency. The exact process may vary depending on the explorer used by Chakra, but typically it involves flattening your contract and submitting the source code to the explorer.

#### 4.4 Test the deployed contract

After deployment, it's crucial to test your contract to ensure it's working as expected. You can create a simple script to interact with your deployed contract:

```javascript
const { ethers } = require("hardhat");

async function main() {
  const [signer] = await ethers.getSigners();
  
  const BTCSettlementArtifact = await ethers.getContractFactory("BTCSettlementArtifact");
  const artifact = BTCSettlementArtifact.attach("YOUR_DEPLOYED_CONTRACT_ADDRESS");

  // Test metadata
  const metadata = await artifact.artifact_metadata();
  console.log("Metadata:", metadata);

  // Test minting
  const mintTx = await artifact.mint_to(signer.address, ethers.utils.parseEther("1"));
  await mintTx.wait();
  console.log("Minted 1 token to", signer.address);

  // Check balance
  const balance = await artifact.balanceOf(signer.address);
  console.log("Balance:", ethers.utils.formatEther(balance));
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Run this script with:

```bash
npx hardhat run scripts/test_settlement_artifact.js --network chakra
```

This completes the implementation and deployment of the Settlement Artifact Contract on the Chakra network. The contract is now ready to be used in the cross-chain settlement process.

### Step 5: Register Settlement Artifact

After deploying the Settlement Artifact Contract, we need to register it with the Chakra Registry. This step is crucial for the Chakra network to recognize and interact with our Settlement Artifact.

#### 5.1 Create a registration script

Create a new file named `register_artifact.js` in your project's `scripts` folder:

```javascript
const { ethers } = require("hardhat");
require("dotenv").config();

const CHAKRA_REGISTRY_ADDRESS = "YOUR_CHAKRA_REGISTRY_ADDRESS"; // Replace with the actual Chakra Registry address
const SETTLEMENT_ARTIFACT_ADDRESS = "YOUR_SETTLEMENT_ARTIFACT_ADDRESS"; // Replace with your deployed Settlement Artifact address

async function main() {
  const [signer] = await ethers.getSigners();

  const chakraRegistryABI = [
    "function registrySettlementArtifact(address artifact) external"
  ];

  const chakraRegistry = new ethers.Contract(CHAKRA_REGISTRY_ADDRESS, chakraRegistryABI, signer);

  console.log("Registering Settlement Artifact...");
  
  try {
    const tx = await chakraRegistry.registrySettlementArtifact(SETTLEMENT_ARTIFACT_ADDRESS);
    await tx.wait();
    console.log("Settlement Artifact registered successfully!");
    console.log("Transaction Hash:", tx.hash);
  } catch (error) {
    console.error("Error registering Settlement Artifact:", error);
  }
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 5.2 Run the registration script

Execute the registration script using Hardhat:

```bash
npx hardhat run scripts/register_artifact.js --network chakra
```

This will register your Settlement Artifact with the Chakra Registry, allowing it to be recognized and used within the Chakra ecosystem.

### Step 6: Implement Settled Token Contract

The Settled Token Contract represents the BTC on the target chain (in this case, Scroll Sepolia). This ERC-20 token will be minted when BTC is locked in the Settlement Artifact and burned when BTC is released.

#### 6.1 Create the Settled Token Contract

Create a new file named `SettledBTC.sol` in your `contracts` folder:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

interface ISettled {
    function mint_to(address account, uint256 value) external;
    function burn_from(address account, uint256 value) external;
}

contract SettledBTC is Initializable, ERC20Upgradeable, AccessControlUpgradeable, UUPSUpgradeable, ISettled {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");
    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(address defaultAdmin, address minter, address burner) initializer public {
        __ERC20_init("Settled BTC", "sBTC");
        __AccessControl_init();
        __UUPSUpgradeable_init();

        _grantRole(DEFAULT_ADMIN_ROLE, defaultAdmin);
        _grantRole(MINTER_ROLE, minter);
        _grantRole(BURNER_ROLE, burner);
        _grantRole(UPGRADER_ROLE, defaultAdmin);
    }

    function mint_to(address account, uint256 value) external onlyRole(MINTER_ROLE) {
        _mint(account, value);
    }

    function burn_from(address account, uint256 value) external onlyRole(BURNER_ROLE) {
        _burn(account, value);
    }

    function _authorizeUpgrade(address newImplementation)
        internal
        onlyRole(UPGRADER_ROLE)
        override
    {}
}
```

#### 6.2 Create a deployment script for Settled Token

Create a new file named `deploy_settled_token.js` in your `scripts` folder:

```javascript
const { ethers, upgrades } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const SettledBTC = await ethers.getContractFactory("SettledBTC");
  
  const settledBTC = await upgrades.deployProxy(SettledBTC, [
    deployer.address, // defaultAdmin
    deployer.address, // minter (you may want to set this to your Settlement Handler address later)
    deployer.address  // burner (you may want to set this to your Settlement Handler address later)
  ]);

  await settledBTC.deployed();

  console.log("SettledBTC deployed to:", settledBTC.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 6.3 Configure Hardhat for Scroll Sepolia

Update your `hardhat.config.js` to include the Scroll Sepolia network:

```javascript
require("@nomiclabs/hardhat-waffle");
require('@openzeppelin/hardhat-upgrades');
require("dotenv").config();

module.exports = {
  solidity: "0.8.24",
  networks: {
    chakra: {
      url: "https://rpcv1-dn-1.chakrachain.io",
      accounts: [process.env.PRIVATE_KEY],
      gas: "auto",
      gasPrice: "auto"
    },
    scrollSepolia: {
      url: "https://sepolia-rpc.scroll.io/",
      accounts: [process.env.PRIVATE_KEY],
      gas: "auto",
      gasPrice: "auto"
    }
  }
};
```

#### 6.4 Deploy the Settled Token Contract

Run the deployment script:

```bash
npx hardhat run scripts/deploy_settled_token.js --network scrollSepolia
```

This will deploy your SettledBTC contract to the Scroll Sepolia network.

#### 6.5 Verify the contract (optional)

If Scroll Sepolia has a block explorer that supports contract verification, you can verify your contract for transparency. The process would be similar to verifying on Etherscan.

#### 6.6 Test the deployed Settled Token Contract

Create a test script named `test_settled_token.js` in your `scripts` folder:

```javascript
const { ethers } = require("hardhat");

async function main() {
  const [signer] = await ethers.getSigners();
  
  const SettledBTC = await ethers.getContractFactory("SettledBTC");
  const settledBTC = SettledBTC.attach("YOUR_DEPLOYED_SETTLED_BTC_ADDRESS");

  // Test minting
  const mintTx = await settledBTC.mint_to(signer.address, ethers.utils.parseEther("1"));
  await mintTx.wait();
  console.log("Minted 1 sBTC to", signer.address);

  // Check balance
  const balance = await settledBTC.balanceOf(signer.address);
  console.log("Balance:", ethers.utils.formatEther(balance), "sBTC");

  // Test burning
  const burnTx = await settledBTC.burn_from(signer.address, ethers.utils.parseEther("0.5"));
  await burnTx.wait();
  console.log("Burned 0.5 sBTC from", signer.address);

  // Check updated balance
  const updatedBalance = await settledBTC.balanceOf(signer.address);
  console.log("Updated Balance:", ethers.utils.formatEther(updatedBalance), "sBTC");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Run the test script:

```bash
npx hardhat run scripts/test_settled_token.js --network scrollSepolia
```

This completes the implementation and deployment of the Settled Token Contract on the Scroll Sepolia network. The contract is now ready to be used in the cross-chain settlement process.

### Step 7: Implement Settlement Handler Contracts

The Settlement Handler contracts are responsible for managing the cross-chain communication and token minting/burning on both the Chakra and target chains.

#### 7.1 Clone the Settlement Handler template

First, clone the template repository:

```bash
git clone https://github.com/Generative-Labs/chakra-evm-settlement-handler-template.git
cd chakra-evm-settlement-handler-template
```

#### 7.2 Implement the Settlement Handler contract

Open the `contracts/ERC20SettlementHandler.sol` file and implement the following key functions:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "./interfaces/ISettlement.sol";
import "./interfaces/ISettlementSignatureVerifier.sol";
import "./interfaces/ICodec.sol";

contract ERC20SettlementHandler is Initializable, OwnableUpgradeable, UUPSUpgradeable {
    ISettlement public settlement;
    ISettlementSignatureVerifier public verifier;
    ICodec public codec;
    address public token;
    string public chainName;
    bool public noBurn;

    function initialize(
        address _owner,
        bool _noBurn,
        string memory _chainName,
        address _token,
        address _codec,
        address _verifier,
        address _settlement
    ) public initializer {
        __Ownable_init(_owner);
        __UUPSUpgradeable_init();
        noBurn = _noBurn;
        chainName = _chainName;
        token = _token;
        codec = ICodec(_codec);
        verifier = ISettlementSignatureVerifier(_verifier);
        settlement = ISettlement(_settlement);
    }

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

        IERC20(token).transferFrom(msg.sender, address(this), amount);

        ERC20TransferPayload memory payload = ERC20TransferPayload(
            ERC20Method.Transfer,
            AddressCast.to_uint256(msg.sender),
            to,
            AddressCast.to_uint256(token),
            to_token,
            amount
        );

        bytes memory cross_chain_msg_bytes = MessageV1Codec.encode(
            Message(
                uint256(keccak256(abi.encodePacked(block.number, address(this), msg.sender))),
                PayloadType.ERC20,
                codec.encode_transfer(payload)
            )
        );

        settlement.send_cross_chain_msg(
            to_chain,
            msg.sender,
            to_handler,
            PayloadType.ERC20,
            cross_chain_msg_bytes
        );
    }

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
        require(payload_type == PayloadType.ERC20, "Invalid payload type");

        bytes memory erc20_payload = MessageV1Codec.payload(payload);
        ERC20Method method = codec.decode_method(erc20_payload);

        if (method == ERC20Method.Transfer) {
            ERC20TransferPayload memory transfer_payload = codec.deocde_transfer(erc20_payload);
            
            if (noBurn) {
                require(
                    IERC20(token).balanceOf(address(this)) >= transfer_payload.amount,
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

        return false;
    }

    function receive_cross_chain_callback(
        uint256 txid,
        string memory from_chain,
        uint256 from_handler,
        CrossChainMsgStatus status,
        uint8 sign_type,
        bytes calldata signatures
    ) external returns (bool) {
        // Implement callback logic here
        return true;
    }

    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}
```

#### 7.3 Create deployment scripts

Create two deployment scripts, one for Chakra and one for Scroll Sepolia.

For Chakra (`scripts/deploy_chakra_handler.js`):

```javascript
const { ethers, upgrades } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const ERC20SettlementHandler = await ethers.getContractFactory("ERC20SettlementHandler");
  
  const handler = await upgrades.deployProxy(ERC20SettlementHandler, [
    deployer.address,
    false, // noBurn
    "Chakra",
    "YOUR_SETTLEMENT_ARTIFACT_ADDRESS",
    "YOUR_CODEC_ADDRESS",
    "YOUR_VERIFIER_ADDRESS",
    "YOUR_SETTLEMENT_CONTRACT_ADDRESS"
  ]);

  await handler.deployed();

  console.log("ERC20SettlementHandler deployed to:", handler.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

For Scroll Sepolia (`scripts/deploy_scroll_handler.js`):

```javascript
const { ethers, upgrades } = require("hardhat");

async function main() {
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const ERC20SettlementHandler = await ethers.getContractFactory("ERC20SettlementHandler");
  
  const handler = await upgrades.deployProxy(ERC20SettlementHandler, [
    deployer.address,
    false, // noBurn
    "Scroll",
    "YOUR_SETTLED_BTC_ADDRESS",
    "YOUR_CODEC_ADDRESS",
    "YOUR_VERIFIER_ADDRESS",
    "YOUR_SETTLEMENT_CONTRACT_ADDRESS"
  ]);

  await handler.deployed();

  console.log("ERC20SettlementHandler deployed to:", handler.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 7.4 Deploy the Settlement Handler contracts

Deploy to Chakra:

```bash
npx hardhat run scripts/deploy_chakra_handler.js --network chakra
```

Deploy to Scroll Sepolia:

```bash
npx hardhat run scripts/deploy_scroll_handler.js --network scrollSepolia
```

### Step 8: Initiate Cross-Chain Settlement

Now that all components are in place, we can initiate a cross-chain settlement.

#### 8.1 Create a script to initiate settlement

Create a file named `initiate_settlement.js` in the `scripts` folder:

```javascript
const { ethers } = require("hardhat");

async function main() {
  const [signer] = await ethers.getSigners();

  const ChakraHandlerAddress = "YOUR_CHAKRA_HANDLER_ADDRESS";
  const ScrollHandlerAddress = "YOUR_SCROLL_HANDLER_ADDRESS";
  const SettledBTCAddress = "YOUR_SETTLED_BTC_ADDRESS";

  const ChakraHandler = await ethers.getContractFactory("ERC20SettlementHandler");
  const chakraHandler = ChakraHandler.attach(ChakraHandlerAddress);

  const amount = ethers.utils.parseEther("1"); // 1 BTC

  console.log("Initiating cross-chain settlement...");

  const tx = await chakraHandler.cross_chain_erc20_settlement(
    "Scroll",
    ScrollHandlerAddress,
    SettledBTCAddress,
    signer.address,
    amount
  );

  await tx.wait();

  console.log("Cross-chain settlement initiated. Transaction hash:", tx.hash);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### 8.2 Run the settlement script

Execute the script to initiate a cross-chain settlement:

```bash
npx hardhat run scripts/initiate_settlement.js --network chakra
```

This will initiate the cross-chain settlement process, locking the BTC in the Settlement Artifact on Chakra and minting the corresponding amount of SettledBTC on Scroll Sepolia.

## Summary

Congratulations! You have now implemented a complete BTC to ERC-20 cross-chain settlement system using the Chakra Settlement Layer. This system allows users to lock their BTC and receive a corresponding ERC-20 token on the Scroll Sepolia network.

Here's a summary of what we've accomplished:

1. Set up BTC Backend Services for monitoring deposits and handling withdrawals.
2. Developed Chakra Backend Services for interacting with the Settlement Layer.
3. Implemented and deployed the Settlement Artifact Contract on Chakra.
4. Registered the Settlement Artifact with the Chakra Registry.
5. Implemented and deployed the Settled Token Contract on Scroll Sepolia.
6. Implemented and deployed Settlement Handler Contracts on both Chakra and Scroll Sepolia.
7. Initiated a cross-chain settlement.

Remember to thoroughly test your implementation in a controlled environment before deploying to production. Additionally, ensure that you have proper security measures in place, especially for handling private keys and managing the custodial wallet for BTC.

This implementation provides a foundation that can be extended and optimized based on specific requirements and use cases. Always stay updated with the latest security practices and blockchain developments to ensure the safety and efficiency of your cross-chain settlement system.

