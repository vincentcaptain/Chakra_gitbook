# LST Settlement Tutorial #1: BTC to stBTC Conversion

This tutorial demonstrates how to use Settlement Primitives and Settlement Artifacts to convert BTC into stBTC (staked BTC) on a target chain using the Chakra Network as an intermediary.

## Overview

We'll implement two main processes:

1. Locking Settlement Artifact on Chakra Network and Minting stBTC on Target Chain
2. Burning stBTC on Target Chain and Unlocking Settlement Artifact on Chakra Network

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

## Before you start

* Familiarity with Solidity and [cross-chain development](../cross-chain-transfer-with-settlement-layer.md)
* Understanding of [Settlement Primitives](../../concepts/chakra-settlement-primitives/) and [Settlement Artifacts](../../concepts/chakra-settlement-artifacts/)
* Access to Chakra Network and the target chain (e.g., Ethereum, Arbitrum, Scroll, etc.)

## Contract Structure

We'll create the following contracts:

1. `LSTSettlementHandler` (on Chakra Network)
2. `LSTSettlementHandler` (on Target Chain)
3. `STBTC` (on Target Chain)

Let's start with the implementation.

#### 1. LSTSettlementHandler (Chakra Network)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "./ISettlementContract.sol";

contract LSTSettlementHandler {
    ISettlementContract public settlementContract;
    IERC20 public coboSettlementArtifact;
    mapping(uint256 => bool) public lockedArtifacts;

    event ArtifactLocked(uint256 artifactId, address user);
    event ArtifactUnlocked(uint256 artifactId, address user);

    constructor(address _settlementContract, address _coboSettlementArtifact) {
        settlementContract = ISettlementContract(_settlementContract);
        coboSettlementArtifact = IERC20(_coboSettlementArtifact);
    }

    function lockArtifact(uint256 artifactId) external {
        require(coboSettlementArtifact.ownerOf(artifactId) == msg.sender, "Not the owner");
        coboSettlementArtifact.transferFrom(msg.sender, address(this), artifactId);
        lockedArtifacts[artifactId] = true;
        
        // Send settlement message to target chain
        bytes memory message = abi.encode(msg.sender, artifactId);
        settlementContract.sendCrossChainMessage("TargetChain", message);
        
        emit ArtifactLocked(artifactId, msg.sender);
    }

    function unlockArtifact(uint256 artifactId, address user) external {
        require(msg.sender == address(settlementContract), "Only settlement contract");
        require(lockedArtifacts[artifactId], "Artifact not locked");
        
        lockedArtifacts[artifactId] = false;
        coboSettlementArtifact.transferFrom(address(this), user, artifactId);
        
        emit ArtifactUnlocked(artifactId, user);
    }
}
```

#### 2. LSTSettlementHandler (Target Chain)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./STBTC.sol";
import "./ISettlementContract.sol";

contract LSTSettlementHandler {
    STBTC public stbtc;
    ISettlementContract public settlementContract;

    event STBTCMinted(address user, uint256 amount);
    event STBTCBurned(address user, uint256 amount);

    constructor(address _stbtc, address _settlementContract) {
        stbtc = STBTC(_stbtc);
        settlementContract = ISettlementContract(_settlementContract);
    }

    function mintSTBTC(address user, uint256 amount) external {
        require(msg.sender == address(settlementContract), "Only settlement contract");
        stbtc.mint(user, amount);
        emit STBTCMinted(user, amount);
    }

    function burnSTBTC(uint256 amount) external {
        stbtc.burn(msg.sender, amount);
        
        // Send settlement message to Chakra Network
        bytes memory message = abi.encode(msg.sender, amount);
        settlementContract.sendCrossChainMessage("ChakraNetwork", message);
        
        emit STBTCBurned(msg.sender, amount);
    }
}
```

#### 3. STBTC Token Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract STBTC is ERC20, Ownable {
    constructor() ERC20("Staked BTC", "stBTC") {}

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) external onlyOwner {
        _burn(from, amount);
    }
}
```

### Deployment and Setup

1. Deploy the `STBTC` contract on the target chain.
2. Deploy the `LSTSettlementHandler` on both Chakra Network and the target chain.
3. Set the `LSTSettlementHandler` as the owner of the `STBTC` contract.

### Process 1: Locking Settlement Artifact and Minting stBTC

#### Step 1: User Request

The user calls the `lockArtifact` function on the Chakra Network:

```javascript
const artifactId = 123; // Example artifact ID
await lstSettlementHandlerChakra.lockArtifact(artifactId);
```

#### Step 2: Locking the Artifact

This is handled automatically by the `lockArtifact` function in the contract.

#### Step 3: Settlement Message

The `sendCrossChainMessage` function is called automatically within the `lockArtifact` function.

#### Step 4: Minting stBTC

On the target chain, implement a function to handle the incoming settlement message:

```solidity
function handleSettlementMessage(bytes memory message) external {
    require(msg.sender == address(settlementContract), "Only settlement contract");
    (address user, uint256 artifactId) = abi.decode(message, (address, uint256));
    uint256 stbtcAmount = getSTBTCAmount(artifactId); // Implement this function
    mintSTBTC(user, stbtcAmount);
}
```

### Process 2: Burning stBTC and Unlocking Settlement Artifact

#### Step 1: User Request

The user calls the `burnSTBTC` function on the target chain:

```javascript
const amount = ethers.utils.parseEther("1"); // Amount of stBTC to burn
await lstSettlementHandlerTarget.burnSTBTC(amount);
```

#### Step 2: Burning stBTC

This is handled automatically by the `burnSTBTC` function in the contract.

#### Step 3: Settlement Message

The `sendCrossChainMessage` function is called automatically within the `burnSTBTC` function.

#### Step 4: Unlocking the Artifact

On the Chakra Network, implement a function to handle the incoming settlement message:

```solidity
function handleSettlementMessage(bytes memory message) external {
    require(msg.sender == address(settlementContract), "Only settlement contract");
    (address user, uint256 amount) = abi.decode(message, (address, uint256));
    uint256 artifactId = getArtifactId(amount); // Implement this function
    unlockArtifact(artifactId, user);
}
```

### Testing

Here's a basic test script to verify the functionality:

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("LST Settlement", function () {
  let lstHandlerChakra, lstHandlerTarget, stbtc, owner, user;

  beforeEach(async function () {
    [owner, user] = await ethers.getSigners();

    const STBTC = await ethers.getContractFactory("STBTC");
    stbtc = await STBTC.deploy();

    const LSTSettlementHandler = await ethers.getContractFactory("LSTSettlementHandler");
    lstHandlerChakra = await LSTSettlementHandler.deploy(/* settlement contract */, /* cobo artifact contract */);
    lstHandlerTarget = await LSTSettlementHandler.deploy(stbtc.address, /* settlement contract */);

    await stbtc.transferOwnership(lstHandlerTarget.address);
  });

  it("Should lock artifact and mint stBTC", async function () {
    // Simulate locking artifact on Chakra Network
    await lstHandlerChakra.lockArtifact(123);

    // Simulate receiving settlement message on target chain
    await lstHandlerTarget.handleSettlementMessage(ethers.utils.defaultAbiCoder.encode(["address", "uint256"], [user.address, 123]));

    expect(await stbtc.balanceOf(user.address)).to.be.gt(0);
  });

  it("Should burn stBTC and unlock artifact", async function () {
    // Mint some stBTC for testing
    await lstHandlerTarget.mintSTBTC(user.address, ethers.utils.parseEther("1"));

    // Burn stBTC
    await lstHandlerTarget.connect(user).burnSTBTC(ethers.utils.parseEther("1"));

    // Simulate receiving settlement message on Chakra Network
    await lstHandlerChakra.handleSettlementMessage(ethers.utils.defaultAbiCoder.encode(["address", "uint256"], [user.address, ethers.utils.parseEther("1")]));

    // Check if artifact is unlocked (you'll need to implement a way to verify this)
  });
});
```

This tutorial provides a basic implementation of the LST settlement process using Settlement Primitives and Settlement Artifacts. Remember to implement proper security measures, error handling, and thorough testing before deploying to a live network.
