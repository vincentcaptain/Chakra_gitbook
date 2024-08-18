# Getting Started

Chakra's Modular Settlement Layer enables a developer building on one chain to transfer between any supported chain. We provide a secure and efficient way to move assets across different blockchain networks. This guide will walk you through setting up and executing cross-chain token transfers using Chakra’s infrastructure.

With Chakra, you can:&#x20;

* Transfer tokens between any supported chain
* Incorporate BTC Liquidity through Artifacts

## Before you start

Before you begin the token transfer process, ensure the following prerequisites are met:

1. **Knowledge and Tools:**
   * Understand how to write, compile, and deploy smart contracts.
   * Familiarize yourself with using a wallet, such as MetaMask.
2. **Network Tokens:**
   * Ensure your account has sufficient tokens on the source network to cover transaction fees.
3. Make sure that both the source and destination chains [must be supported by the Chakra Settlement Layer](../resources/testnet-and-contract-references.md).
4. Clone the Settlement Handler contract template as shown below:

```bash
git clone git@github.com:Generative-Labs/chakra-evm-settlement-handler-template.git
mv chakra-evm-settlement-handler-template <your project name>
```

5. Set Up the Token Contract:&#x20;

If you don't already have an ERC20 token contract, you can use the provided template to create and deploy one.

```solidity
contract MyToken is ERC20 {
    constructor() ERC20("MyToken", "MTK") {
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }
}
```

## Create the Settlement Handler Contract on the Source Chain

We can create the handler contract **on the source chain** to initiate the transfer. **Example code snippet to initiate the transfer is shown below:**

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
    require(IERC20(token).balanceOf(msg.sender) >= amount, "Insufficient balance");

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

For [a detailed description ](../resources/contracts-and-script-templates/settlement-handler-contract/cross\_chain\_erc20\_settlement.md)of each parameter and its functionality, please see [here](../resources/contracts-and-script-templates/settlement-handler-contract/cross\_chain\_erc20\_settlement.md)

## **Chakra Network Validation**

Chakra validators confirm and process the transfer.

* If your source chain is BTC network, Chakra handles the Artifact to your account through [Settlement Primitives](../concepts/chakra-settlement-primitives/).&#x20;
* If your source chain is one of the other networks, Settlement Messages will be sent and received by the corresponding handler contract on the destination chain.&#x20;

## Receive Cross-chain Messages on the Destination Chain

On the **destination chain**, we implement a receipt settlement handler contract to receive and confirm the message from the source chain. **Here is an example code snippet to receive cross-chain messages:**&#x20;

```solidity
function receive_cross_chain_msg(
    uint256 txid,
    string memory from_chain,
    uint256 from_address,
    uint256 from_handler,
    bytes calldata payload,
    uint8 sign_type,
    bytes calldata signatures
) external returns (bool) {
    require(
        MessageV1Codec.payload_type(payload) == PayloadType.ERC20,
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

For [a detailed description ](../resources/contracts-and-script-templates/settlement-handler-contract/receive\_cross\_chain\_msg.md)of each parameter and its functionality, please see [here](../resources/contracts-and-script-templates/settlement-handler-contract/receive\_cross\_chain\_msg.md)

## \[OPTIONAL] Receive Cross-chain Callback

If a callback is initiated, please follow the **example code snippet to handle the source chain callback**

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

For [a detailed description ](../resources/contracts-and-script-templates/settlement-handler-contract/receive\_cross\_chain\_callback.md)of each parameter and its functionality, please see [here](../resources/contracts-and-script-templates/settlement-handler-contract/receive\_cross\_chain\_callback.md).

## Deployment and Configuration

Successfully deploying and configuring your cross-chain token transfer setup involves deploying the necessary smart contracts and configuring the network parameters to ensure smooth operation. We have provided the template script to guide you through the deployment to the supported chain. Please ensure that your network parameters are correctly set up. This configuration enables deployment to the appropriate blockchain networks.&#x20;

### Example script to deploy to Scroll:&#x20;

You can use the [deployment script template](../resources/contracts-and-script-templates/deployment-script-template.md) below:&#x20;

```javascript
// scripts/deploy.js
const hre = require("hardhat");

async function main() {
    // 1. Deploy Token Contract
    const tokenOwner = "<Your token owner address>"
    const tokenOperator = "<Your token operator address>"
    const decimals = 8
    const name = "MyToken"
    const symbol = "MKT"

    const MyToken = await hre.ethers.getContractFactory("MyToken")
    const tokenInstance = await hre.upgrades.deployProxy(MyToken, [tokenOwner, tokenOperator, name, symbol, decimals])
    await tokenInstance.waitForDeployment()
    const tokenAddress = await tokenInstance.getAddress()
    console.log("Token contract deployed to:", tokenAddress)

    // 2. Deploy Codec Contract
    const codecOwner = "<Your codec owner address>"
    const ERC20CodecV1 = await hre.ethers.getContractFactory("ERC20CodecV1")
    const codecInstance = await hre.upgrades.deployProxy(ERC20CodecV1, [codecOwner])
    await codecInstance.waitForDeployment()
    console.log("Codec contract deployed to:", await codecInstance.getAddress())

    // 3. Deploy Verify Contract
    const requiredValidators = 2
    const verifyOwner = "<Your verify owner address>"
    const SettlementSignatureVerifier = await hre.ethers.getContractFactory("SettlementSignatureVerifier")
    const verifyInstance = await hre.upgrades.deployProxy(SettlementSignatureVerifier, [verifyOwner, requiredValidators])
    await verifyInstance.waitForDeployment()
    console.log("Verify contract deployed to:", await verifyInstance.getAddress())

    // 4. Deploy SettlementHandler Contract
    const no_burn = true
    const chain_name = "<Your chain name>"
    const settlementContractAddress = "<Your settlement contract address>"
    const settlementHandlerOwner = "<Your settlement handler owner address>"
    const ERC20SettlementHandler = await hre.ethers.getContractFactory("ERC20SettlementHandler")
    const settlementHandlerInstance = await hre.upgrades.deployProxy(ERC20SettlementHandler, [
        settlementHandlerOwner,
        no_burn,
        chain_name,
        await tokenInstance.getAddress(),
        await codecInstance.getAddress(),
        await verifyInstance.getAddress(),
        settlementContractAddress
    ])
    await settlementHandlerInstance.waitForDeployment()
    console.log("SettlementHandler contract deployed to:", await settlementHandlerInstance.getAddress())

    // Add the settlement contract as a manager
    await verifyInstance.add_manager(settlementContractAddress)
    console.log("Settlement contract added as manager to Verify contract:", settlementContractAddress)
}

main().catch((error) => {
    console.error(error)
    process.exitCode = 1
})

```

and the [Configuration script template](../resources/contracts-and-script-templates/configuration-script-template.md) below:&#x20;

```json
networks: {
    scroll: {
        url: "https://scroll-sepolia.public.blastapi.io",
        accounts: ["<Your private key in this chain>"],
        gas: "auto",
        gasPrice: "auto"
    },
    chakra: {
        url: "https://rpcv1-dn-1.chakrachain.io",
        accounts: ["<Your private key in this chain>"],
        gas: "auto",
        gasPrice: "auto"
    }
}

```

Execute the following commands to deploy the contracts to the respective chains

```
npx hardhat run scripts/deploy.js --network scroll --config hardhat.config
```

For more examples of deploying to other chains, please see [here](../resources/contracts-and-script-templates/deployment-script-template.md).

## Examine the Sample Code

Using the last script, you can now deploy your settlement handler contract on Scroll (source chain) to initiate the transfer, and another settlement handler contract on Arbitrum (destination chain) to receive the transfer. Now we start utilizing the deployed contract to initiate the cross-chain settlement. The below example illustrate how to call `cross_chain_erc20_settlement` on your source chain (Scroll sepolia in this case) to send your token to the destination chain (Arbitrum sepolia in this case).

The example execution script is shown here:&#x20;

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
        const toChain = "Arbitrum"; // target chain name

        const toHandler = "<Your target chain handler contract address>"; // uint256 format
        const toToken = "<Your target chain token contract address>"; // uint256 format
        const to = "<Your target chain receive address>"; // uint256 format

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

And you can execute the script by doing

```
node cross_chain.js
```

You can verify the results via:&#x20;

1. Scroll sepolia: 100,000,000 `MyToken` was transferred from the initializer to the handler contract

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

2. Arbitrum sepolia: 100,000,000 `MyToken` was minted to the receiver's address

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

3. Scroll sepolia: 100,000,000 `MyToken` was burned

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

