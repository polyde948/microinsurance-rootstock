Decentralized Microinsurance for Smallholder Farmers on Rootstock

A Step-by-Step Guide to Building Trustless Agricultural Insurance on Bitcoin-Powered Rootstock


---

Introduction

Smallholder farmers face significant risks from unpredictable weather events like droughts and heatwaves. Traditional insurance is often inaccessible or slow. This project uses Rootstock smart contracts and oracles to create decentralized microinsurance, enabling automatic payouts when weather thresholds are breached.


---

Why Rootstock?

Bitcoin Security: Inherits security from Bitcoin’s merge-mining.

Ethereum Compatibility: Solidity smart contracts, easy tooling.

Low Fees: Affordable microtransactions ideal for small premiums.

Decentralization: Removes intermediaries, fraud, and delays.



---

System Overview

Farmers register and pay insurance premiums (in RBTC).

A weather oracle supplies current rainfall and temperature data.

When bad weather occurs, payouts are triggered automatically.

The admin manages thresholds and payouts securely.



---

Smart Contract Details

Core Features

Farmers register by sending premium payments.

Registered farmers stored in a manageable array for iteration.

Admin sets rainfall & temperature thresholds.

Admin triggers claims payouts if conditions met.

Events emitted for transparency.


Fixed Solidity Code (Key Part)

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

interface IOracle {
    function getWeatherData() external view returns (uint256 rainfall, uint256 temperature);
}

contract MicroInsurance {
    struct Farmer {
        bool isRegistered;
        uint256 premiumPaid;
        bool claimPaid;
    }

    address public admin;
    IOracle public oracle;
    uint256 public rainfallThreshold;
    uint256 public temperatureThreshold;

    address[] public farmerAddresses;
    mapping(address => Farmer) public farmers;

    event Registered(address farmer, uint256 premium);
    event ClaimPaid(address farmer, uint256 amount);
    event ThresholdsUpdated(uint256 rainfall, uint256 temperature);

    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin");
        _;
    }

    constructor(address _oracle, uint256 _rainfallThreshold, uint256 _temperatureThreshold) {
        admin = msg.sender;
        oracle = IOracle(_oracle);
        rainfallThreshold = _rainfallThreshold;
        temperatureThreshold = _temperatureThreshold;
    }

    function register() external payable {
        require(!farmers[msg.sender].isRegistered, "Already registered");
        require(msg.value > 0, "Premium required");

        farmers[msg.sender] = Farmer(true, msg.value, false);
        farmerAddresses.push(msg.sender);

        emit Registered(msg.sender, msg.value);
    }

    function updateThresholds(uint256 _rainfallThreshold, uint256 _temperatureThreshold) external onlyAdmin {
        rainfallThreshold = _rainfallThreshold;
        temperatureThreshold = _temperatureThreshold;

        emit ThresholdsUpdated(rainfallThreshold, temperatureThreshold);
    }

    function checkAndPayClaims() external onlyAdmin {
        (uint256 rainfall, uint256 temperature) = oracle.getWeatherData();

        for (uint i = 0; i < farmerAddresses.length; i++) {
            address farmerAddress = farmerAddresses[i];
            Farmer storage farmer = farmers[farmerAddress];
            if (farmer.isRegistered && !farmer.claimPaid) {
                if (rainfall < rainfallThreshold || temperature > temperatureThreshold) {
                    payable(farmerAddress).transfer(farmer.premiumPaid * 2);
                    farmer.claimPaid = true;
                    emit ClaimPaid(farmerAddress, farmer.premiumPaid * 2);
                }
            }
        }
    }

    receive() external payable {}
}


---

Development Environment Setup

Prerequisites

Node.js (v16+) and npm

MetaMask installed with Rootstock Testnet added:

RPC URL: https://public-node.testnet.rsk.co

Chain ID: 31


RSK Testnet RBTC from faucet

Remix IDE (https://remix.ethereum.org) or Hardhat for local deployment



---

Deployment Steps

1. Deploy Oracle Contract (Mock)

Use this mock oracle for testing:

contract MockOracle is IOracle {
    function getWeatherData() external pure override returns (uint256, uint256) {
        return (30, 38); // Simulate drought and heatwave
    }
}

Deploy MockOracle first and copy the contract address.



---

2. Deploy MicroInsurance Contract

Use Remix or Hardhat.

Deploy MicroInsurance passing:

_oracle = MockOracle contract address

_rainfallThreshold = 50 (example)

_temperatureThreshold = 35 (example)




---

3. Register Farmers

From different accounts (wallets), call register() payable with RBTC premium (e.g., 0.001 RBTC).

Confirm registration via Registered event.



---

4. Trigger Claims Payout

Admin calls checkAndPayClaims() after oracle updates.

Contract checks weather data and pays eligible farmers 2x their premium.



---

Interacting with the Contract

You can use Remix or build a simple React frontend with Ethers.js to:

Register farmers by sending premiums.

Admin update thresholds.

Admin trigger payouts.



---

Future Improvements

Replace mock oracle with Chainlink or other decentralized oracles.

Build full UI dashboards for farmers and admins.

Add staking and community governance for claim validation.

Extend to cover livestock, health insurance, or crop varieties.



---

Social Impact

This decentralized insurance helps poor farmers gain financial resilience by automating payouts on climate risk, reducing fraud, and increasing trust.


---

License

MIT License — free to use and modify.

