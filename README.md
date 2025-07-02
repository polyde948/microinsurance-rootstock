README.md

# 🌾 Decentralized Microinsurance for Smallholder Farmers on Rootstock

A secure, trustless microinsurance smart contract system built on the Rootstock blockchain. This project enables smallholder farmers to register and receive automated insurance payouts triggered by real-world weather data, helping them recover from crop loss due to droughts or extreme heat.

## 🚀 Features

- 🧑‍🌾 **Farmer Registration** using BTC or USDT-based premium payments
- 🌦️ **Weather Oracle Integration** to detect crop risks based on rainfall/temperature
- 💸 **Automated Claim Payouts** without intermediaries or paperwork
- 🔐 **On-Chain Transparency** with immutable records of all activity
- 🔄 **Extensible Design** for future integration with staking, analytics, or dApps

---

## 🏗️ Architecture Overview

1. Farmers register and pay insurance premiums via a smart contract.
2. A weather oracle feeds rainfall and temperature data to the contract.
3. If thresholds are breached (e.g., drought), the contract verifies and pays claims.
4. All activity is logged on-chain for full auditability.

---

## 🔧 Smart Contract Code (Solidity)

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

    function checkAndPayClaims() external onlyAdmin {
        (uint256 rainfall, uint256 temperature) = oracle.getWeatherData();

        for (address farmerAddress = address(0); farmerAddress <= address(type(uint160).max); farmerAddress = address(uint160(farmerAddress) + 1)) {
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

🧪 Deployment & Testing Instructions

1. Deploy the contract on Rootstock Testnet.


2. Pass oracle address and thresholds as constructor parameters.


3. Register farmers using the register() payable function.


4. Call checkAndPayClaims() as admin to trigger payouts.


5. Use a mock oracle or integrate a decentralized weather oracle like Chainlink.




---

🔮 Future Enhancements

Integrate real decentralized oracles for secure weather data

Add frontend dashboards for farmers and administrators

Introduce staking pools for decentralized claim validators

Expand policy types (e.g., livestock, flood, health)



---

📜 License

MIT License — free to use, modify, and expand.


---

🙌 Built for Rootstock Hacktivate

Empowering vulnerable farming communities using decentralized technology. Built with ❤️ by Nkanyiso.