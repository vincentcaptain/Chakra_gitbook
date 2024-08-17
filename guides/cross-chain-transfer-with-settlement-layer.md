# Cross-chain Transfer with Settlement Layer

In this tutorial, you'll learn how to use the Chakra Settlement Layer to transfer tokens from Chain A to Chain B. We'll use the example of transferring tokens from Scroll to Arbitrum.

## Before you start

Before you begin, make sure you're familiar with the following:

1. Writing, compiling, and deploying smart contracts. If you need to learn more, check out [Solidity documentation](https://soliditylang.org/).
2. Using cryptocurrency wallets. [MetaMask](https://metamask.io/) is a good starting point.
3. Your account must have some tokens on both Scroll and Arbitrum networks for transaction fees.
   * For Scroll testnet tokens: [Scroll Faucet](https://docs.scroll.io/en/user-guide/faucet/)
   * For Arbitrum testnet tokens:
     * [Chainlink Faucet](https://faucets.chain.link/arbitrum-sepolia)
     * [Alchemy Faucet](https://www.alchemy.com/faucets/arbitrum-sepolia)

## Tutorial:&#x20;

We'll complete the cross-chain functionality in the following steps:

1. Understanding and implementing core contract functions
2. Configuring contract deployment parameters
3. Writing contract deployment scripts
4. Deploying contracts
5. Initiating a cross-chain call

### 1. Core Contract Functions

You need to implement three main functions in your Settlement Handler Contract:

**a. `cross_chain_erc20_settlement`**

This function initiates the cross-chain transfer request:

```solidity
function cross_chain_erc20_settlement(
    string memory to_chain,
    uint256 to_handler,
    uint256 to_token,
    uint256 to,
    uint256 amount
) external {
    // Check requirements
    require(amount > 0, "Amount must be greater than 0");
    require(to != 0, "Invalid to address");
    require(to_handler != 0, "Invalid to handler address");
    require(to_token != 0, "Invalid to token address");
    require(
        IERC20(token).balanceOf(msg.sender) >= amount,
        "Insufficient balance"
    );

    // Transfer tokens to this contract
    IERC20(token).transferFrom(msg.sender, address(this), amount);

    // Create cross-chain message
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

    // Create and encode payload
    ERC20TransferPayload memory payload = ERC20TransferPayload(
        ERC20Method.Transfer,
        AddressCast.to_uint256(msg.sender),
        to,
        AddressCast.to_uint256(token),
        to_token,
        amount
    );
    Message memory cross_chain_msg = Message(
        cross_chain_msg_id,
        PayloadType.ERC20,
        codec.encode_transfer(payload)
    );
    bytes memory cross_chain_msg_bytes = MessageV1Codec.encode(cross_chain_msg);

    // Send cross-chain message
    settlement.send_cross_chain_msg(
        to_chain,
        msg.sender,
        to_handler,
        PayloadType.ERC20,
        cross_chain_msg_bytes
    );
}
```

**b. `receive_cross_chain_msg`**

This function receives the cross-chain message on the target chain:

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
    ERC20Method method = codec.decode_method(erc20_payload);

    if (method == ERC20Method.Transfer) {
        ERC20TransferPayload memory transfer_payload = codec.deocde_transfer(erc20_payload);

        if (no_burn) {
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
```

**c. `receive_cross_chain_callback`**

This function receives the callback on the source chain after the transfer is complete:

```solidity
function receive_cross_chain_callback(
    uint256 txid,
    string memory from_chain,
    uint256 from_handler,
    CrossChainMsgStatus status,
    uint8 sign_type,
    bytes calldata signatures
) external returns (bool) {
    require(
        keccak256(abi.encodePacked(cross_chain_settlement_txs[txid].to_chain)) == keccak256(abi.encodePacked(from_chain)),
        "Invalid from chain"
    );
    require(
        cross_chain_settlement_txs[txid].to_handler == from_handler,
        "Invalid from handler"
    );

    if (no_burn) {
        IERC20Burn(token).burn(cross_chain_settlement_txs[txid].amount);
    }

    cross_chain_settlement_txs[txid].status = CrossChainTxStatus.Settled;

    return true;
}
```

### 2. Configure Deployment Parameters

Update your `hardhat.config.ts` file with the network configurations:

```javascript
networks: {
  scroll: {
    url: "https://scroll-sepolia.public.blastapi.io",
    accounts: ["<Your private key in this chain>"],
    gas: "auto",
    gasPrice: "auto"
  },
  arb: {
    url: "https://arbitrum-sepolia.public.blastapi.io",
    accounts: ["<Your private key in this chain>"],
    gas: "auto",
    gasPrice: "auto"
  }
}
```

### 3. Write Deployment Script

Create a deployment script (`scripts/deploy.js`):

```javascript
const hre = require("hardhat");
const { task } = require('hardhat/config')

async function main() {
    // Deploy Token Contract
    const tokenOwner = "<Your token owner address>";
    const tokenOperator = "<Your token operator address>";
    const decimals = 8;
    const name = "MyToken";
    const symbol = "MKT";
    const MyToken = await hre.ethers.getContractFactory("MyToken");
    const tokenInstance = await hre.upgrades.deployProxy(MyToken, [tokenOwner, tokenOperator, name, symbol, decimals]);
    await tokenInstance.waitForDeployment();
    const tokenAddress = await tokenInstance.getAddress();
    console.log("MyToken contract deployed to: ", tokenAddress);

    // Deploy Codec Contract
    const codecOwner = "<Your codec owner address>";
    const ERC20CodecV1 = await hre.ethers.getContractFactory("ERC20CodecV1");
    const codecInstance = await hre.upgrades.deployProxy(ERC20CodecV1, [codecOwner]);
    await codecInstance.waitForDeployment();
    console.log("Codec contract deployed to: ", await codecInstance.getAddress());

    // Deploy Verify Contract
    const requiredValidators = 2;
    const verifyOwner = "<Your verify owner address>";
    const SettlementSignatureVerifier = await hre.ethers.getContractFactory("SettlementSignatureVerifier");
    const verifyInstance = await hre.upgrades.deployProxy(SettlementSignatureVerifier, [verifyOwner, requiredValidators]);
    await verifyInstance.waitForDeployment();
    console.log("SettlementSignatureVerifier contract deployed to: ", await verifyInstance.getAddress());

    // Deploy SettlementHandler Contract
    const no_burn = true;
    const chain_name = "<Your chain name>";  // "scroll" for Scroll, "arb" for Arbitrum
    const settlementContractAddress = "<Your settlement contract address>";
    const settlementHandlerOwner = "<Your settlement handler owner address>";
    const ERC20SettlementHandler = await hre.ethers.getContractFactory("ERC20SettlementHandler");
    const settlementHandlerInstance = await hre.upgrades.deployProxy(ERC20SettlementHandler, [
        settlementHandlerOwner,
        no_burn,
        chain_name,
        await tokenInstance.getAddress(),
        await codecInstance.getAddress(),
        await verifyInstance.getAddress(),
        settlementContractAddress
    ]);
    await settlementHandlerInstance.waitForDeployment();
    console.log("SettlementHandler contract deployed to: ", await settlementHandlerInstance.getAddress());

    // Add the settlement contract as a manager
    await verifyInstance.add_manager(settlementContractAddress);
    console.log("Settlement contract added as manager to SettlementSignatureVerifier contract: ", settlementContractAddress);
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

### 4. Deploy Contracts

Deploy to Scroll:

```bash
npx hardhat run scripts/deploy.js --network scroll --config hardhat.config.ts
```

Deploy to Arbitrum:

```bash
npx hardhat run scripts/deploy.js --network arb --config hardhat.config.ts
```

### 5. Initiate Cross-chain Transfer

Create a script to initiate the cross-chain transfer (`cross_chain.js`):

```javascript
const { ethers } = require("ethers");

const RPC_PROVIDER_URL = "https://scroll-sepolia.public.blastapi.io";
const provider = new ethers.providers.JsonRpcProvider(RPC_PROVIDER_URL);

const privateKey = "<YOUR_PRIVATE_KEY>";
const wallet = new ethers.Wallet(privateKey, provider);

const contractAddress = "<YOUR_CONTRACT_ADDRESS>";

const handleABI = [
    "function cross_chain_erc20_settlement(string to_chain, uint256 to_handler, uint256 to_token, uint256 to, uint256 amount) external"
];

const contract = new ethers.Contract(contractAddress, handleABI, wallet);

async function callCrossChainSettlement() {
    try {
        const toChain = "Arbitrum";
        const toHandler = "<Your target chain handler contract address>";
        const toToken = "<Your target chain token contract address>";
        const to = "<Your target chain receive address>";
        const amount = ethers.utils.parseUnits("100000000", 'ether');

        const tx = await contract.cross_chain_erc20_settlement(toChain, toHandler, toToken, to, amount);
        const receipt = await tx.wait();
        console.log("Transaction successful with hash:", receipt.transactionHash);
    } catch (error) {
        console.error("Error calling contract method:", error);
    }
}

callCrossChainSettlement();
```

Run the script:

```bash
node cross_chain.js
```

### Conclusion

By following this tutorial, you've learned how to implement cross-chain token transfers using the Chakra Settlement Layer. This process involves deploying contracts on both the source and target chains, implementing key functions for handling cross-chain messages, and initiating the transfer. Before using it in a production environment, remember to thoroughly test your implementation and consider security implications.
