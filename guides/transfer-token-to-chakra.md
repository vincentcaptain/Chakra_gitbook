# Transfer Token to Chakra

In this tutorial, you will learn how to transfer tokens to Chakra using the Chakra Settlement Layer. First, you'll need to implement a Settlement Handler Contract for your token to create a transfer interface. Then, you'll send a cross-chain message through the Chakra Settlement contract and mint your token on Chakra. For example, you'll inject tUSDC from your current chain and settle it as cUSDC on Chakra.

### Before you begin

1. You should know how to write, compile, [deploy](https://remix.ethereum.org/), and [develop smart contracts](https://soliditylang.org/). You should also be familiar with using wallets; [MetaMask](https://metamask.io/) is a good starting point.
2. Your account must have some tokens on the source chain to cover transaction fees (e.g., ETH).

### Download the Settlement Handler Contract Template

Before you start developing the Settlement Handler contract, we provide a template to get you started. You can obtain the template and begin developing your contract as follows:

> Note: This project is built using Hardhat, an integrated development environment for smart contracts. You can learn more about it [here](https://hardhat.org/docs).

```bash
git clone git@github.com:Generative-Labs/chakra-evm-settlement-handler-template.git
mv chakra-evm-settlement-handler-template <your project name>
```

### Deploy and Configure the Token Contract

In our example, we're using tUSDC as a sample token. Therefore, we need to deploy a token contract. In most cases, you might already have an ERC20 token contract on a specific chain. If so, you can skip this part and follow the same steps to integrate with the Settlement Handler contract later.

To set up and deploy the tUSDC contract, you can use `contracts/MyToken.sol`:

```bash
cp contracts/MyToken.sol contracts/tUSDC.sol
```

Then, modify:

```solidity
contract MyToken {
```

to:

```solidity
contract TestUSDC {
```

Open the `contracts/tUSDC.sol` contract code. The `initialize` function is where you set the contract's parameters:

```solidity
function initialize(
    address _owner,
    address _operator,
    string memory _name,
    string memory _symbol,
    uint8 _decimals
) public initializer {}
```

* `owner` is the address of the contract owner
* `operator` is the address that has permission to mint and burn tokens
* `name`, `symbol`, and `decimals` are common ERC20 parameters

To deploy this contract, you can use one of two methods:

1. Write a deployment script
2. Use Remix

### Develop the Settlement Handler Contract

Developing the Settlement Handler contract consists of the following parts:

1. Set up and deploy the tUSDC ERC20 contract (optional)
2. Implement the `cross_chain_usdc_settlement` method to initiate cross-chain token transfers. In this example, we'll transfer USDC to Chakra.
3. Implement the `receive_cross_chain_msg` method to receive cross-chain messages from the Chakra chain. In this example, we'll receive USDC cross-chain transfer messages from Chakra.

#### Implement `cross_chain_usdc_settlement`

* Open the `contracts/ERC20SettlementHandler.sol` file
* Change the `cross_chain_erc20_settlement` method to `cross_chain_usdc_settlement`
* Here's an example implementation of `cross_chain_usdc_settlement`:

```solidity
function cross_chain_usdc_settlement(
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

    // Create an erc20 transfer payload
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

#### Implement `receive_cross_chain_msg`

* The `receive_cross_chain_msg` method primarily receives cross-chain messages from Chakra. In this example, the message content is an ERC20 transfer.
* Open the `contracts/ERC20SettlementHandler.sol` file and locate the `receive_cross_chain_msg` method. You need to implement this method.
* Here's an example that receives a cross-chain message from Chakra, decodes it as an ERC20 Transfer, and then mints an equivalent amount of the corresponding token:

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

### Deploy the Contract

You'll need to deploy the contract to two chains. Let's assume your current chain is Scroll. Modify the `networks` parameter in your project's `hardhat.config.ts`:

```javascript
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

#### Write a Deployment Script

```bash
mkdir scripts
touch scripts/deploy.js
```

Here's an example of `scripts/deploy.js`. Modify the `tokenOwner` and other custom parameters to your own configuration:

```javascript
// scripts/deploy.js
const hre = require("hardhat");
const { task } = require('hardhat/config')

async function main() {
    // 1. Deploy Token Contract
    const tokenOwner = "<Your token owner address>"
    const tokenOperator = "<Your token operator address>"
    const decimals = 8
    const name = "Test USDC"
    const symbol = "tUSDC"
    const TestUSDC = await hre.ethers.getContractFactory("TestUSDC")
    const tokenInstance = await hre.upgrades.deployProxy(TestUSDC, [tokenOwner, tokenOperator, name, symbol, decimals]);
    await tokenInstance.waitForDeployment();
    const tokenAddress = await tokenInstance.getAddress();
    console.log("tUSDC contract deployed to: ", tokenAddress)

    // 2. Deploy Codec Contract
    const codecOwner = "<Your codec owner address>"
    const ERC20CodecV1 = await hre.ethers.getContractFactory("ERC20CodecV1");
    const codecInstance = await hre.upgrades.deployProxy(ERC20CodecV1, [codecOwner]);
    await codecInstance.waitForDeployment();
    console.log("Codec contract deployed to: ", await codecInstance.getAddress())

    // 3. Deploy Verify Contract
    const requiredValidators = 2;
    const verifyOwner = "<Your verify owner address>"
    const SettlementSignatureVerifie = await hre.ethers.getContractFactory("SettlementSignatureVerifier");

    const verifyInstance = await hre.upgrades.deployProxy(SettlementSignatureVerifie, [
        verifyOwner,
        requiredValidators
    ]);
    await verifyInstance.waitForDeployment();
    console.log("SettlementSignatureVerifier contract deployed to: ", await verifyInstance.getAddress())

    // 4. Deploy SettlementHandler Contract
    const no_burn = true;
    const chain_name = "<Your chain name>"   // in this case scroll chainName = scroll ;    arbitrum chainName = arb
    const settlementContractAddress = "<Your settlement contract address>"
    const settlementHandlerOwner = "<Your settlement handler owner address>"
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
    console.log("SettlementHandler contract deployed to: ", await settlementHandlerInstance.getAddress())

    // Add the settlement contract as a manager
    await verifyInstance.add_manager(settlementContractAddress);
    console.log("Settlement contract added as manager to SettlementSignatureVerifier contract: ", settlementContractAddress)
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

#### Deploy to the Scroll Chain

```bash
npx hardhat run scripts/deploy.js --network scroll --config hardhat.config.ts
```

### Contract Interaction

#### Install Dependencies for the Contract Interaction Script

```bash
yarn add ethers
```

#### Write the Interaction Script

Create a file named `cross_chain.js` with the following content to call the `cross_chain_usdc_settlement` method and initiate a cross-chain transfer:

```javascript
const { ethers } = require("ethers");

const RPC_PROVIDER_URL = "https://scroll-sepolia.public.blastapi.io";

const provider = new ethers.providers.JsonRpcProvider(RPC_PROVIDER_URL);

const privateKey = "<YOUR_PRIVATE_KEY>";
const wallet = new ethers.Wallet(privateKey, provider);

const contractAddress = "<YOUR_CONTRACT_ADDRESS>";

const contractABI = [
    "function cross_chain_usdc_settlement(string to_chain, uint256 to_handler, uint256 to_token, uint256 to, uint256 amount) external"
];

const contract = new ethers.Contract(contractAddress, contractABI, wallet);

async function callCrossChainSettlement() {
    try {
        const toChain = "arb"; // target chain name

        const toHandler = "<Your target chain handler contract address>"; // uint256 format
        const toToken = "<Your target chain token contract address>"; // uint256 format
        const to = "<Your target chain receive address>"; // uint256 format

        const amount = ethers.utils.parseUnits("50000", 8);

        const tx = await contract.cross_chain_usdc_settlement(toChain, toHandler, toToken, to, amount);

        const receipt = await tx.wait();

        console.log("Transaction successful with hash:", receipt.transactionHash);
    } catch (error) {
        console.error("Error calling contract method:", error);
    }
}

callCrossChainSettlement();
```

Execute the script with:

```bash
node cross_chain.js
```

This completes the tutorial on transferring tokens to Chakra using the Chakra Settlement Layer. The process involves setting up your token contract, implementing the necessary Settlement Handler functions, deploying the contracts, and finally executing a cross-chain transfer.
