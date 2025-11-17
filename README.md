MediChain (MDC) - Decentralized Medical Data Platform  

🏥 Overview  

MediChain is a blockchain-based healthcare platform that empowers patients to own, control, and monetize their medical data while enabling researchers to access anonymized health information for breakthrough discoveries. The platform also facilitates clinical trial matching and pharmaceutical supply chain verification.

🌟 Key Features

Patient-Owned Medical Records: Encrypted health data stored on IPFS, fully controlled by patients  

Data Marketplace: Patients earn MDC tokens by contributing anonymized data to research studies  

Clinical Trial Matching: Smart contract-based trial enrollment with automated compensation  

Drug Supply Chain Tracking: Track medications from manufacturer to patient, eliminate counterfeits  

Granular Access Control: Grant/revoke access to medical records with time-based expiration  

Transparent Reward System: Automatic token distribution for data contributions  


📋 Table of Contents

Architecture  

Smart Contracts  

Installation  

Deployment  

Usage Examples  

Testing  

Security  

Contributing  

License


┌─────────────────────────────────────────────────────────┐
│                    Frontend Layer                        │
│            (React + Web3.js + IPFS Client)              │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│               Smart Contracts Layer                      │
├──────────────────────────────────────────────────────────┤
│  • MediChainToken (ERC20)                               │
│  • MedicalRecordsRegistry                               │
│  • DataMarketplace                                       │
│  • ClinicalTrialMatching                                │
│  • DrugSupplyChain                                       │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│          Blockchain Network (Ethereum/Polygon)          │
│                  + IPFS Storage Layer                    │
└─────────────────────────────────────────────────────────┘



<img width="1770" height="766" alt="Screenshot 2025-11-17 131407" src="https://github.com/user-attachments/assets/8d3ad6a1-cc9f-4a8c-9903-1f6560f3daaa" />


Contract Address - 0xd9145CCE52D386f254917e481eB44e9943F39138  
