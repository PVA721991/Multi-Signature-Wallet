# Multi-Signature-Wallet
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleMultiSig {
    address[3] public owners;
    uint256 public required = 2;
    uint256 public transactionCount;

    struct Transaction {
        address to;
        uint256 value;
        bool executed;
    }

    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;

    constructor(address[3] memory _owners) {
        owners = _owners;
    }

    function submitTransaction(address _to, uint256 _value) public {
        uint256 txId = transactionCount++;
        transactions[txId] = Transaction(_to, _value, false);
    }

    function confirmTransaction(uint256 txId) public {
        bool isOwner = false;
        for (uint i = 0; i < 3; i++) {
            if (owners[i] == msg.sender) isOwner = true;
        }
        require(isOwner, "Not owner");
        confirmations[txId][msg.sender] = true;
    }

    function executeTransaction(uint256 txId) public {
        Transaction storage tx = transactions[txId];
        require(!tx.executed, "Already executed");
        uint256 count = 0;
        for (uint i = 0; i < 3; i++) {
            if (confirmations[txId][owners[i]]) count++;
        }
        require(count >= required, "Not enough confirmations");
        tx.executed = true;
        payable(tx.to).transfer(tx.value);
    }
}
