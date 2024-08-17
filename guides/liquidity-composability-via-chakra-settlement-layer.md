# Liquidity Composability via Chakra Settlement Layer

The Chakra Settlement Layer, through collaboration with LST/LRT, has accumulated substantial liquidity on-chain, represented by [Settlement Artifacts](../concepts/chakra-settlement-artifacts/). In this tutorial, you will develop a DApp using the liquidity provided by Liquidity Composability Settlement Artifacts. These artifacts can be utilized as standard ERC20 tokens, allowing you to wrap them into tokens, create liquidity pools, and more.

## Before you start

1. The Chakra Settlement Layer is fully compatible with EVM, so your development journey will revolve around EVM and Solidity. You should know how to [develop](https://soliditylang.org/) and [deploy](https://remix.ethereum.org/) Solidity smart contracts. Additionally, familiarity with wallets, such as [MetaMask](https://metamask.io/), is beneficial.
2. Since you will be developing on the Chakra Settlement Layer, you need some Chakra Settlement Layer Chain native tokens. Here is how you can get them from the Chakra Settlement Layer faucet:&#x20;

## Example 1: Wrapping a Single Artifact into a Token

This example demonstrates how to use the Settlement Artifacts and wrap them into another ERC20 token, providing methods for deposit and withdrawal.

💡 The code is for illustrative purposes only and has not been audited. Use it cautiously.&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract SingleArtifactWrapper is ERC20 {
    address public artifact;

    /// @notice Emitted when an artifact is deposited
    /// @param artifact The address of the artifact that was deposited
    event ArtifactDeposited(address indexed artifact);

    /// @notice Emitted when a deposited artifact is withdrawn back into the original artifact
    /// @param artifact The address of the artifact that was withdrawn
    event ArtifactWithdrawn(address indexed artifact);

    constructor(
        address _artifact, 
        string memory name, 
        string memory symbol
    ) ERC20(name, symbol) {
        artifact = _artifact;
    }

    /// @notice Deposit the artifact into tokens
    /// @param amount The amount of artifact tokens to deposit
    function deposit(uint256 amount) external {
        require(IERC20(artifact).transferFrom(msg.sender, address(this), amount), "Transfer failed");
        _mint(msg.sender, amount);
        emit ArtifactDeposited(artifact);
    }

    /// @notice Withdraw the tokens back into the original artifact
    /// @param amount The amount of deposited tokens to withdraw
    function withdraw(uint256 amount) external {
        _burn(msg.sender, amount);
        require(IERC20(artifact).transfer(msg.sender, amount), "Transfer failed");
        emit ArtifactWithdrawn(artifact);
    }
}

```

### **Parameter Explanation:**

* `address public artifact` represents the address of the Settlement Artifacts.
* `deposit(uint256 amount)` function is for users to deposit artifacts.
* `withdraw(uint256 amount)` function is for users to withdraw artifacts.

### Contract Deployment

1. Configure the networks parameter in your project's hardhat.config.ts for the Chakra testnet chain:

```json
 networks: {
    chakra: {
        url: "https://rpcv1-dn-1.chakrachain.io",
        accounts: ["<Your private key in this chain>"],
    }
  }

```

2. Write the deployment script:&#x20;

```bash
mkdir scripts
touch scripts/deploy.js

```

An example of the script can be found [here](../resources/contracts-and-script-templates/deployment-script-template.md)

3. Deploy the contract to the Chakra network using the hardhat command:

```bash
npx hardhat run scripts/deploy.js --network chakra
```

### Contract Interaction&#x20;

1. Install Dependencies: `yarn add ethers`
2. Write the interaction script `touch deposit.js`

An example script to call `deposit` is attached here:&#x20;

```jsx
const { ethers } = require("ethers");

const RPC_PROVIDER_URL = "https://rpcv1-dn-1.chakrachain.io";

const provider = new ethers.providers.JsonRpcProvider(RPC_PROVIDER_URL);

const privateKey = "<YOUR_PRIVATE_KEY>";
const wallet = new ethers.Wallet(privateKey, provider);

const contractAddress = "<YOUR_CONTRACT_ADDRESS>";

const contractABI = [
    "function deposit(uint256 amount) external"
];

const contract = new ethers.Contract(contractAddress, contractABI, wallet);

async function deposit() {
    try {
        const amount = 10000;

        const tx = await contract.deposit(amount);

        const receipt = await tx.wait();

        console.log("Transaction successful with hash:", receipt.transactionHash);
    } catch (error) {
        console.error("Error calling contract method:", error);
    }
}

deposit();

```

3. You can execute the Script via `node deposit.js`

## Example 2: Wrapping Multiple Artifacts into single Token

This example demonstrates how to wrap multiple Settlement Artifacts into a single ERC20 token, providing methods for deposit and withdrawal.

💡 The code is for illustrative purposes only and has not been audited. Use it cautiously.&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MultiArtifactDepository is ERC20 {
    address[] public artifacts;
    uint256[] public ratios;

    event ArtifactsDeposited(address[] indexed artifacts, uint256[] amounts);

    event ArtifactsWithdrawn(address[] indexed artifacts, uint256[] amounts);

    constructor(
        address[] memory _artifacts, 
        uint256[] memory _ratios,
        string memory name, 
        string memory symbol
    ) ERC20(name, symbol) {
        require(_artifacts.length == _ratios.length, "Artifacts and ratios length mismatch");
        artifacts = _artifacts;
        ratios = _ratios;
    }

    /// @notice Deposit artifacts into tokens
    /// @param amounts The amounts of each artifact to deposit
    function deposit(uint256[] calldata amounts) external {
        require(amounts.length == artifacts.length, "Amounts length mismatch");

        uint256 totalAmount = 0;

        for (uint256 i = 0; i < artifacts.length; i++) {
            require(IERC20(artifacts[i]).transferFrom(msg.sender, address(this), amounts[i]), "Transfer failed");
            totalAmount += amounts[i] * ratios[i];
        }

        _mint(msg.sender, totalAmount);
        emit ArtifactsDeposited(artifacts, amounts);
    }

    /// @notice Withdraw tokens back into the original artifacts
    /// @param amount The total amount of tokens to withdraw
    function withdraw(uint256 amount) external {
        uint256 totalRatio = sum(ratios);
        uint256[] memory amounts = new uint256[](artifacts.length);

        for (uint256 i = 0; i < artifacts.length; i++) {
            uint256 artifactAmount = (amount * ratios[i]) / totalRatio;
            amounts[i] = artifactAmount;
        }

        _burn(msg.sender, amount);

        for (uint256 i = 0; i < artifacts.length; i++) {
            require(IERC20(artifacts[i]).transfer(msg.sender, amounts[i]), "Transfer failed");
        }

        emit ArtifactsWithdrawn(artifacts, amounts);
    }

    /// @notice Helper function to sum an array of uint256 values
    /// @param values The array of values to sum
    /// @return The sum of the values
    function sum(uint256[] memory values) private pure returns (uint256) {
        uint256 total = 0;
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        return total;
    }
}

```

### **Parameter Explanation:**

* `address[] public artifacts` represents the addresses of the Settlement Artifacts included in the token.
* `deposit(uint256[] calldata amounts)` function is for users to deposit multiple artifacts.
* `withdraw(uint256 amount)` function is for users to withdraw multiple artifacts.

You can use the same procedure for the deployment and interaction process in Example 1.&#x20;
