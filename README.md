Decentralized Microinsurance for Smallholder Farmers on Rootstock

---

Overview

This project implements a decentralized microinsurance smart contract system tailored for smallholder farmers. Leveraging Rootstock’s secure smart contract platform, the system automates premium collection, claim validation, and payout processes based on real-world data inputs from decentralized oracles. This ensures trustless, transparent, and timely insurance services for vulnerable farming communities.

---

Features

- *Farmer Registration & Premium Payment* in BTC/USDT tokens  
- *Automated Claim Processing* triggered by oracle-verified crop loss or adverse weather conditions  
- *Instant Payouts* eliminating intermediaries and delays  
- *Immutable Audit Trail* of all insurance-related transactions  
- *Extensible Design* for future integration with staking, risk assessment, and frontends

---

Architecture & Workflow

1. Farmers register by paying an insurance premium through the smart contract.  
2. Oracles feed verified data (e.g., rainfall, temperature, crop yield) to the contract.
3. 3. When adverse conditions meet predefined thresholds, claims are automatically validated.  
4. The smart contract disburses payouts directly to affected farmers.  
5. All actions are recorded on-chain for transparency and auditability.

---

Smart Contract Code

```solidity
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
        emit Registered(msg.sender, msg.value);
    }

    function updateThresholds(uint256 _rainfallThreshold, uint256 _temperatureThreshold) external onlyAdmin {
        rainfallThreshold = _rainfallThreshold;
        temperatureThreshold = _temperatureThreshold;
        emit ThresholdsUpdated(rainfallThreshold, temperatureThreshold);
    }

    function checkAndPayClaims() external {
        (uint256 rainfall, uint256 temperature) = oracle.getWeatherData();

        for (address farmerAddress = address(0); farmerAddress <= address(type(uint160).max); farmerAddress = address(uint160(farmerAddress) + 1)) {
            Farmer storage farmer = farmers[farmerAddress];
            if (farmer.isRegistered && !farmer.claimPaid) {
                if (rainfall < rainfallThreshold || temperature > temperatureThreshold) {
                    payable(farmerAddress).transfer(farmer.premiumPaid * 2); // Example payout: double premium
                    farmer.claimPaid = true;
                    emit ClaimPaid(farmerAddress, farmer.premiumPaid * 2);
}
            }
        }
    }

    // Fallback function to receive payments
    receive() external payable {}
}
```

---

Deployment & Usage

- Deploy the contract on Rootstock Testnet  
- Provide the oracle address (deployed separately or mocked for testing)  
- Set rainfall and temperature thresholds according to insurance policy  
- Farmers register by sending premiums via `register()`  
- Admin triggers `checkAndPayClaims()` periodically (can be automated via bots)  
- Claim payouts are sent automatically when conditions meet thresholds  

---

Extensibility & Future Work

- Integrate with Chainlink or Band Protocol for reliable oracle data  
- Develop a frontend dashboard for farmers to track policies and claims  
- Introduce staking mechanisms for claim validators or risk assessors  
- Expand insurance types: livestock, health, weather-based derivatives  

---

License

MIT License – free to use and adapt for your projects.

--
