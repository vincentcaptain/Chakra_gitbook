# Interfaces of Artifacts and its Registry

## Interface of Artifacts

Settlement Artifacts are implemented on the Chakra Chain as smart contracts, which extend the ERC20 standard to include metadata revealing the attributes of the Settlement Primitive that created them. Below is the Solidity interface for a Settlement Artifact:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

interface ISettlementArtifact is IERC20 {
    /**
     * @dev Returns the metadata of the settlement artifacts.
     */    
    function artifact_metadata() external view returns (string memory);
}

```

## Settlement Artifacts Registry

The Chakra Chain maintains a Settlement Artifacts Registry Contract, which provides basic information about all registered Settlement Artifacts. Projects developed based on Settlement Artifacts can query this registry to obtain essential data about the artifacts. Below is an example of the main interface for the Settlement Artifacts Registry Contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./ISettlementArtifact.sol";

contract SettlementArtifactFactory {
    address[] public settlementArtifacts;
    
    event SettlementArtifactRegisted(address indexed artifactAddress);

    /**
     * @dev Returns the list of registered Settlement Artifacts.
     */
    function getSettlementArtifacts() external view returns (address[] memory) {
        return settlementArtifacts;
    }

    /**
     * @dev Registers a new Settlement Artifact.
     * @param artifact The address of the Settlement Artifact contract to be registered.
     */
    function registrySettlementArtifact(address artifact) external {
        require(
            ISettlementArtifact(artifact).artifact_metadata.selector ==
            bytes4(keccak256("artifact_metadata()")),
            "Address does not implement ISettlementArtifact"
        );
        
        settlementArtifacts.push(artifact);
        
        emit SettlementArtifactRegisted(artifact);
    }
}

```
