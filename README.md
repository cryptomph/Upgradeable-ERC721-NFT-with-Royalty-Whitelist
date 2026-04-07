# Upgradeable-ERC721-NFT-with-Royalty-Whitelist
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable.sol";
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/security/ReentrancyGuard.sol";

contract Crowdfunding is Ownable, ReentrancyGuard {
    struct Milestone {
        string description;
        uint256 targetAmount;
        bool completed;
    }

    string public projectName;
    uint256 public goal;
    uint256 public raised;
    uint256 public deadline;
    Milestone[] public milestones;

    mapping(address => uint256) public contributions;
    bool public funded;

    event Contribution(address indexed backer, uint256 amount);
    event MilestoneCompleted(uint256 index);
    event FundsWithdrawn(uint256 amount);

    constructor(string memory _name, uint256 _goal, uint256 _durationDays) Ownable(msg.sender) {
        projectName = _name;
        goal = _goal;
        deadline = block.timestamp + (_durationDays * 1 days);
    }

    function addMilestone(string memory desc, uint256 target) external onlyOwner {
        milestones.push(Milestone(desc, target, false));
    }

    function contribute() external payable nonReentrant {
        require(block.timestamp < deadline, "Campaign ended");
        require(msg.value > 0, "No ETH sent");

        contributions[msg.sender] += msg.value;
        raised += msg.value;

        emit Contribution(msg.sender, msg.value);
    }

    function markMilestoneComplete(uint256 index) external onlyOwner {
        require(index < milestones.length, "Invalid milestone");
        milestones[index].completed = true;
        emit MilestoneCompleted(index);
    }

    function withdrawFunds() external onlyOwner nonReentrant {
        require(raised >= goal, "Goal not reached");
        funded = true;
        uint256 amount = address(this).balance;
        payable(owner()).transfer(amount);
        emit FundsWithdrawn(amount);
    }

    function refund() external nonReentrant {
        require(block.timestamp > deadline && raised < goal, "No refund available");
        uint256 amount = contributions[msg.sender];
        require(amount > 0, "No contribution");

        contributions[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
