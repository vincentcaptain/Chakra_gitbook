# Staking Settlement Artifacts with Native BTC Withdraw

In this section, we outline the development process of a smart contract that is both capable of staking the Settlement Artifacts to earn points and initiating withdraw to BTC mainnet.&#x20;

## Before you start

Before you begin, make sure you're familiar with the following:

1. Writing, compiling, and deploying smart contracts. If you need to learn more, check out [Solidity documentation](https://soliditylang.org/).
2. Using cryptocurrency wallets. [MetaMask](https://metamask.io/) is a good starting point.
3. Your account must have some tokens on both Scroll and Arbitrum networks for transaction fees.
   * For Scroll testnet tokens: [Scroll Faucet](https://docs.scroll.io/en/user-guide/faucet/)
   * For Arbitrum testnet tokens:
     * [Chainlink Faucet](https://faucets.chain.link/arbitrum-sepolia)
     * [Alchemy Faucet](https://www.alchemy.com/faucets/arbitrum-sepolia)

## **Example: Implementation of BtcArtifactStaking Contract**&#x20;

Users can stake ERC20 tokens representing settlement artifacts to earn points, with the capability to request BTC withdrawals via the Chakra Settlement Handler Contract.

The BtcArtifactStaking Contract allows users to stake ERC20 tokens representing settlement artifacts and earn points. Users can also request BTC withdrawals via the Chakra Settlement Handler Contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

/**
 * @title BtcArtifactStaking
 * @dev A staking contract where users can stake an ERC20 token representing a settlement artifact to earn points.
 */
contract BtcArtifactStaking {
    using SafeERC20 for IERC20;

    // The ERC20 token used for staking, representing a settlement artifact
    IERC20 public stakingToken;

    // Address of the contract that provides the withdraw_request function
    address public withdrawRequestContract;

    /**
     * @dev Struct to store staking details.
     * @param amount Amount of tokens staked.
     * @param since Timestamp of when the staking started.
     * @param points Accumulated points.
     */
    struct Stake {
        uint256 amount;
        uint256 since;
        uint256 points;
    }

    // Mapping of user address to their stake details
    mapping(address => Stake) public stakes;

    // Events
    event Staked(address indexed user, uint256 amount, uint256 since);
    event Unstaked(address indexed user, uint256 amount);
    event PointsUpdated(address indexed user, uint256 points);
    event WithdrawRequested(address indexed user, string btcAddress, uint256 amount);

    /**
     * @dev Constructor to set the staking token and withdraw request contract.
     * @param _stakingToken The address of the ERC20 token to be used for staking.
     * @param _withdrawRequestContract The address of the contract with the withdraw_request function.
     */
    constructor(address _stakingToken, address _withdrawRequestContract) {
        stakingToken = IERC20(_stakingToken);
        withdrawRequestContract = _withdrawRequestContract;
    }

    /**
     * @dev Function to stake tokens.
     * @param _amount The amount of tokens to stake.
     */
    function stake(uint256 _amount) external {
        require(_amount > 0, "Amount must be greater than 0");

        // Update user points before staking
        _updatePoints(msg.sender);

        // Transfer tokens from user to contract
        stakingToken.safeTransferFrom(msg.sender, address(this), _amount);

        // Update staking details
        stakes[msg.sender].amount += _amount;
        stakes[msg.sender].since = block.timestamp;

        emit Staked(msg.sender, _amount, block.timestamp);
    }

    /**
     * @dev Function to unstake tokens.
     * @param _amount The amount of tokens to unstake.
     */
    function unstake(uint256 _amount) external {
        require(_amount > 0, "Amount must be greater than 0");
        require(stakes[msg.sender].amount >= _amount, "Insufficient staking balance");

        // Update user points before unstaking
        _updatePoints(msg.sender);

        // Transfer tokens from contract to user
        stakingToken.safeTransfer(msg.sender, _amount);

        // Update staking details
        stakes[msg.sender].amount -= _amount;

        // Reset timestamp if the user has completely unstaked
        if (stakes[msg.sender].amount == 0) {
            stakes[msg.sender].since = 0;
        }

        emit Unstaked(msg.sender, _amount);
    }

    /**
     * @dev Internal function to update user points.
     * @param _user The address of the user whose points are being updated.
     */
    function _updatePoints(address _user) internal {
        if (stakes[_user].amount > 0 && stakes[_user].since > 0) {
            // Calculate time staked
            uint256 timeStaked = block.timestamp - stakes[_user].since;

            // Update points
            stakes[_user].points += stakes[_user].amount * timeStaked;
            stakes[_user].since = block.timestamp;

            emit PointsUpdated(_user, stakes[_user].points);
        }
    }

    /**
     * @dev Function to get user points.
     * @param _user The address of the user whose points are being queried.
     * @return The total points accumulated by the user.
     */
    function getPoints(address _user) external view returns (uint256) {
        Stake memory stakeInfo = stakes[_user];
        if (stakeInfo.amount > 0 && stakeInfo.since > 0) {
            uint256 timeStaked = block.timestamp - stakeInfo.since;
            return stakeInfo.points + (stakeInfo.amount * timeStaked);
        } else {
            return stakeInfo.points;
        }
    }

    /**
     * @dev Function to request a withdrawal from the external contract.
     * @param btcAddress The BTC address to withdraw to.
     * @param amount The amount of tokens to withdraw.
     */
    function requestWithdrawal(string calldata btcAddress, uint256 amount) external {
        require(stakes[msg.sender].amount >= amount, "Insufficient staking balance");

        // Update user points before requesting withdrawal
        _updatePoints(msg.sender);

        // ABI encoded data to call withdraw_request function
        bytes memory data = abi.encodeWithSignature("withdraw_request(string,uint256)", btcAddress, amount);

        // Call the withdraw_request function on the external contract
        (bool success, ) = withdrawRequestContract.call(data);
        require(success, "Withdraw request failed");

        emit WithdrawRequested(msg.sender, btcAddress, amount);
    }
}

```

In the contract above:

* **Staking Functionality**: Users can stake and unstake ERC20 tokens representing settlement artifacts. Staking updates the user's points based on the time and amount staked.
* **Withdrawal Request**: Users can request a BTC withdrawal by calling the `requestWithdrawal` function. This function interacts with the Chakra Settlement Handler Contract to initiate the BTC withdrawal process.
