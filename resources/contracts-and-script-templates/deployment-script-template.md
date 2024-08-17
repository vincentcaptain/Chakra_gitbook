# Deployment Script Template

The following script will guide you through deploying the required smart contracts on both the source and destination chains. This includes deploying the token contract, codec contract, verify contract, and settlement handler contract.

You can use&#x20;

```
npx hardhat run scripts/deploy.js --network scroll --config hardhat.config.ts
```

to deploy to Scroll,&#x20;

```
npx hardhat run scripts/deploy.js --network arb --config hardhat.config.ts
```

to deploy to Arbitrum.

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
