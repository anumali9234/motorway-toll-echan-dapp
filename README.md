# Motorway Toll & E-Challan Management dApp

A student prototype showing how a smart contract can record motorway toll payments and traffic e-challans transparently on the Ethereum **Sepolia** test network.

> **Course:** MG3012 - Blockchain Technology for Business | **Assignment 2:** Mini dApp
> **Program:** BS Financial Technology, 5th Semester (Fall 2026), FAST-NUCES
> **Instructor:** Dr. Usama Arshad

---

## Deployment Details 

| Item | Value |
|---|---|
| Network | Ethereum Sepolia testnet (Chain ID 11155111) |
| Contract address | `0xd1281c31b5ce4e7bd7b98310f0335e68d005591e` |
| Etherscan link | `https://sepolia.etherscan.io/address/0xd1281c31b5ce4e7bd7b98310f0335e68d005591e` |
| Deployment transaction hash | `0x59152223fae95ecd5d7722a4e17ce6dc1e89f4b78b42afadd5ba83192b5cc807` |
| Etherscan verified (bonus) | Yes / No |

---

## 1. Overview

This dApp combines two ideas in one contract:

- **Toll collection:** drivers register a vehicle and pay toll by route and vehicle class. Each payment becomes a permanent record.
- **E-challan management:** an authority wallet issues challans for violations. The vehicle owner pays them, and the challan status changes on-chain.

Both features share one vehicle record, so the system can apply a simple automated rule: **a vehicle with 3 or more unpaid challans cannot pay toll until it clears them.**

## 2. Problem Statement

Toll and challan data is usually held in separate databases controlled by the operator. This makes it hard for the public or an auditor to confirm that records are complete and unchanged, and manual handling can lead to errors, disputes or record manipulation.

## 3. Proposed Blockchain Solution

Store payments and challan states in a smart contract so that:

- every payment has a public transaction hash,
- records cannot be silently edited or deleted,
- business rules (exact fare, owner-only payment, no double payment) are enforced by code,
- anyone can audit the history using events and Etherscan.

## 4. Key Features

- MetaMask wallet connection with Sepolia network check and one-click network switch
- Vehicle registration (plate + class: Car, Bus, Truck)
- Toll payment for 3 routes with class-based pricing
- Authority-only e-challan issuing (3 violation types)
- Owner-only challan payment, with duplicate-payment protection
- Live blockchain records: latest tolls, latest challans, totals
- Pending / confirmed / error messages with Etherscan transaction links
- Illustrative PKR estimates (clearly marked as simulated)

## 5. Technologies Used

- Solidity ^0.8.20 (Remix IDE)
- Ethereum Sepolia testnet and MetaMask
- HTML, CSS, JavaScript (no frameworks)
- ethers.js v5.7.2 (loaded from CDN, same as Week 8)

## 6. Smart Contract (`SmartContract.sol`)

The contract has **5 functions**. It also has a constructor and public state variables (Solidity creates free read getters for these).

| # | Function | Type | Who can call | MetaMask approval? | What it does |
|---|---|---|---|---|---|
| 1 | `registerVehicle(plate, vehicleClass)` | Write | Anyone | Yes (gas) | Creates a vehicle record owned by the caller. Rejects duplicate plates. |
| 2 | `payToll(plate, route)` | Write, payable | Vehicle owner | Yes (gas + toll) | Checks exact fare, records the toll, forwards ETH to the authority. |
| 3 | `issueChallan(plate, violation)` | Write | Authority (deployer) only | Yes (gas) | Creates an unpaid challan and increases the vehicle's unpaid count. |
| 4 | `payChallan(challanId)` | Write, payable | Vehicle owner | Yes (gas + fine) | Checks exact fine, marks the challan paid, forwards ETH to the authority. |
| 5 | `getVehicle(plate)` | Read (view) | Anyone | No | Returns owner, class, toll count, total toll paid, unpaid challans. |

**Public read getters** used by the frontend: `owner`, `tollCount`, `challanCount`, `routeBaseFare`, `classMultiplier`, `violationFine`, `tollRecords`, `challans`.

**Events:** `VehicleRegistered`, `TollPaid`, `ChallanIssued`, `ChallanPaid`.

**Payment safety:**
- Exact amounts only: `msg.value` must equal the fare or fine, so no change or refund logic is needed.
- State is updated **before** ETH is sent (prevents double payment and re-entry problems).
- Funds are forwarded straight to the authority address, so the contract never holds a balance.

### Demo prices (Sepolia test ETH)

| Route | Base fare |
|---|---|
| M-2 Islamabad - Lahore | 0.0010 ETH |
| M-1 Islamabad - Peshawar | 0.0006 ETH |
| M-3 Pindi Bhattian - Multan | 0.0008 ETH |

Class multiplier: Car x1, Bus x2, Truck x3.

| Violation | Fine |
|---|---|
| Over-speeding | 0.0020 ETH |
| Lane violation | 0.0010 ETH |
| No seatbelt | 0.0005 ETH |

These are demo numbers, not real NHA or NHMP rates. The interface shows PKR estimates using a made-up rate of 1 test ETH = PKR 1,000,000, labelled as simulated.

## 7. Architecture

```
  User (browser)
      |  clicks a button
      v
  index.html + style.css + JavaScript
      |  uses ethers.js
      v
  MetaMask  --- user approves transaction --->  Sepolia blockchain
                                                MotorwayTollChallan contract
      ^                                                 |
      |------------ results, events, tx hash -----------|
```

- **Backend:** the smart contract on Sepolia
- **Frontend:** one HTML page with inline JavaScript
- **Wallet:** MetaMask signs every write transaction

## 8. How to Deploy the Contract

1. Install MetaMask, create a wallet, and enable **Show test networks**. Select **Sepolia**.
2. Get free Sepolia test ETH from a faucet (about 0.02 ETH is plenty).
3. Open [Remix IDE](https://remix.ethereum.org) and create a file named `SmartContract.sol`. Paste the contract code.
4. In **Solidity Compiler**, choose version 0.8.20 or newer and click **Compile**.
5. In **Deploy & Run**, set Environment to **Injected Provider - MetaMask**. Confirm the account and that the network is Sepolia.
6. Select `MotorwayTollChallan` and click **Deploy**. Approve in MetaMask.
7. Copy the deployed contract address from Remix ("Deployed Contracts") and save the transaction hash.

> The wallet that deploys the contract becomes the **authority** (it can issue challans and receives payments).

## 9. How to Configure the Frontend

Open `index.html`, find this line near the bottom, and paste your address:

```js
const CONTRACT_ADDRESS = "PASTE_YOUR_SEPOLIA_CONTRACT_ADDRESS_HERE";
```

The ABI is already included in the file and matches the contract. If you change the contract, update the `ABI` list too.

## 10. How to Run

MetaMask works best when the page is served from a local web server (not opened as a file):

```bash
# Option A (Node.js)
npx http-server .

# Option B (Python)
python -m http.server 8000
```

Then open the shown address (for example `http://localhost:8080` or `http://localhost:8000`) in a browser with MetaMask installed.

## 11. How to Test

1. Click **Connect MetaMask**. Check that your address and the green **Sepolia** badge appear.
2. **Register** plate `ICT-1234` as a Car. Approve in MetaMask.
3. **Look up** `ICT-1234`. Check the record shows 0 tolls and 0 unpaid challans.
4. **Pay toll** on a route. Approve. Check the hash link, confirmation message and the records table.
5. As the authority wallet, **issue a challan** for `ICT-1234`.
6. **Check challans** for the plate, then **pay** the challan. Status changes to Paid.
7. Try invalid actions to see clear errors: unregistered plate, duplicate registration, paying a paid challan, issuing a challan from a non-authority wallet, wrong network.
8. Issue 3 challans without paying and check that toll payment is blocked until one is paid.

## 12. Screenshots

| Screenshot | File |
|---|---|
| Contract deployed in Remix, with address visible | `screenshots/1_remix_deployment.png` |
| Frontend connected to MetaMask on Sepolia | `screenshots/2_wallet_connected.png` |
| Successful transaction with hash and updated record | `screenshots/3_transaction_success.png` |

(Add your own images to the `screenshots/` folder before submitting.)

## 13. Limitations

- Prototype only. It is **not** integrated with M-Tag/RFID scanners, NHMP cameras or handheld devices, NHA/FWO systems, Raast, JazzCash, Easypaisa, or any national vehicle registration database.
- Anyone can register any unused plate, because there is no link to a real registration database.
- One authority wallet for all challans; no multiple officers or appeal process.
- Uses Sepolia test ETH, not PKR. Prices are demo values.
- Public blockchain data is visible to everyone, so real use would need privacy and legal review.
- The records tables show only the latest entries; the plate challan check scans the latest 100 challans.

## 14. Future Improvements

- M-Tag / RFID and camera integration through trusted oracles
- Officer role management (several authorised wallets)
- PKR payment rails and a stablecoin or token for toll credit
- Link to verified vehicle registration data
- Challan dispute and appeal workflow
- Event-based indexing for faster history search

## 15. Academic Information

- Course: MG3012 - Blockchain Technology for Business
- Assignment 2: Mini dApp (Solidity + HTML/JS)
- Student: _Your name and roll number here_
- Institution: FAST-NUCES
- License: MIT (educational use)

> Never commit private keys, seed phrases or `.env` files to this repository.
