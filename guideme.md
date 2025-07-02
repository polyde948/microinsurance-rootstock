
GUIDEME.md

# 🌍 Decentralized Microinsurance for Smallholder Farmers on Rootstock
### A Step-by-Step Guide to Building Trustless Agricultural Insurance on the Bitcoin-Powered Blockchain

---

## Introduction

Smallholder farmers—especially in developing nations—live at the mercy of unpredictable weather, natural disasters, and unstable harvests. For many, a single season of drought or heatwave can mean financial ruin. Traditional insurance providers are either too expensive, too slow, or simply absent in rural regions.

This guide introduces a blockchain-powered solution: a **decentralized microinsurance system** built on **Rootstock (RSK)**. By using smart contracts and weather oracles, we automate payouts to farmers based on real-world data, reducing fraud, eliminating bureaucracy, and providing farmers with financial resilience.

---

## Why Use Rootstock for Microinsurance?

Rootstock is the smart contract layer built on top of the Bitcoin blockchain. It offers:

- **Bitcoin-level security**: Rootstock uses merge-mining with Bitcoin, inheriting its hash power.
- **Ethereum-compatible development**: You can write in Solidity and deploy as on Ethereum.
- **Low transaction fees**: Perfect for microtransactions like farmer premiums and payouts.
- **Decentralized automation**: Trustless execution without intermediaries.

Rootstock makes it possible to bring real-world impact to underserved communities using decentralized finance.

---

## 🌱 System Architecture

The system contains the following components:

1. **Smart Contract** — Handles farmer registration, premium storage, threshold checks, and payouts.
2. **Oracle Contract** — Provides real-time weather data (rainfall and temperature).
3. **(Optional) Frontend App** — Allows farmers and admins to interact with the contract through a user-friendly interface.

---

## 🔐 Smart Contract Functionality Overview

Here’s how the system works:

1. **Registration**: Farmers register by sending a small insurance premium in RBTC (or BTC/USDT-wrapped tokens).
2. **Weather Oracle**: A decentralized oracle feeds current rainfall and temperature data to the contract.
3. **Claim Processing**: If rainfall is too low (drought) or temperature too high (heatwave), the contract automatically triggers payouts.
4. **Payout**: Farmers receive double their premium as compensation—instantly and without filing paperwork.

---

## 💻 Smart Contract Code Walkthrough

Let’s review the key parts of the contract.

```solidity
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

This contract:

Accepts premiums.

Stores farmer records.

Uses an oracle to fetch external weather data.

Pays out claims automatically if risk conditions are met.



---

🧰 Development Environment Setup

1. Install MetaMask: Required to manage accounts and interact with Rootstock Testnet.


2. Add Rootstock Testnet to MetaMask manually using RPC:

RPC URL: https://public-node.testnet.rsk.co

Chain ID: 31



3. Get Test RBTC from RSK faucet


4. Use Remix IDE for easy deployment and testing, or Hardhat for local development.




---

🚀 Step-by-Step Deployment

1. Deploy Oracle Contract (Mock for Testing)

Create a mock contract to simulate weather data:

contract MockOracle is IOracle {
    function getWeatherData() external pure override returns (uint256, uint256) {
        return (30, 38); // Low rainfall, high temperature (simulate disaster)
    }
}

Deploy this contract first. Copy the contract address.

2. Deploy MicroInsurance Contract

In Remix:

Use Injected Web3 to connect MetaMask (on Rootstock Testnet)

Deploy MicroInsurance with:

_oracle: Address of your deployed oracle

_rainfallThreshold: e.g. 50

_temperatureThreshold: e.g. 35




---

🤝 Interacting with the Contract

Register a Farmer

Call register() from a new wallet address.

Send RBTC as a premium (e.g., 0.001 RBTC).

Event Registered confirms successful registration.


Trigger a Claim

Admin calls checkAndPayClaims()

If weather conditions are met, payouts are sent to farmers.

Event ClaimPaid confirms which farmer was paid.



---

🧪 Testing Tips

Modify your oracle to return normal or extreme weather values to simulate scenarios.

Monitor contract logs and use Rootstock Testnet explorers to view transactions.

Check if farmer wallets received their payouts.



---

🌐 Frontend Ideas (Optional)

You can build a simple React or Vue dApp that:

Shows current insurance status for a farmer wallet

Displays policy and claim logs

Lets farmers register with wallet + premium

Lets admins trigger payouts and update thresholds


Use Ethers.js or Web3.js to connect to the Rootstock smart contract.


---

🔄 Future Improvements

🌐 Use decentralized oracles like Chainlink (when available on Rootstock)

💼 Add staking models for community-led validation

📊 Add analytics dashboard to track climate trends and claims

🌾 Expand to livestock and health microinsurance

🛡️ Integrate KYC if needed for regulatory compliance



---

🎯 Social Impact

💸 Financial Safety Net: Helps poor farmers bounce back from disasters

📖 Transparency: Code enforces policy—no manipulation or corruption

⚖️ Equity: Anyone can join. No paperwork, no banks, no brokers.

🌍 Decentralization: Built on Bitcoin’s security with smart contracts



---

🏁 Conclusion

This decentralized microinsurance system demonstrates the real-world power of Rootstock and blockchain for social good. You can adapt it to different crops, regions, or risk models. With automation and transparency, we can reduce poverty, protect livelihoods, and increase trust in financial systems.

Let’s build tools that truly matter.

Built with ❤️ by Nkanyiso – Rootstock Hacktivate 2025

---

### ✅ FINAL STEP: Message for Caro/Rootstock Team

Here’s a professional message you can send to Caro when submitting your Google Doc:

Hi Caro,

I’ve completed my updated Rootstock Hacktivate submission titled "Decentralized Microinsurance for Smallholder Farmers on Rootstock." It includes a fully detailed guide and code explanation as requested.

Here is the Google Docs link to my tutorial: 🔗 [Insert your Google Docs link here]

GitHub Repo: https://github.com/polyde948/microinsurance-rootstock/commits?author=polyde948

Let me know if you need any additional information. Thank you for your support and for this opportunity.

Best regards,
Nkanyiso
