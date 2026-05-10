# BookChain – Decentralized Book Rental Platform 📚

**Project:** Project 1 – Decentralised Book Rental (CS218)

## Team Members
* Mohd Hassan Raza Ansari - 240008019
* Dhyan Chandra C - 240041014
* Manish Garasiya - 240041026
* Manjeet Kumar - 240041027
* Harsha Varshan Bonu - 240001020

---

## What is this?
BookChain is a dApp built on Ethereum for renting books. Book owners (lenders) can list their books, and readers can borrow them by putting down a security deposit and a daily fee. The whole thing (listing, renting, returning, and disputes) is managed by our smart contract so it's completely trustless.

---

## Features

### Rental Lifecycle
- **Listing**: Lenders can add books with a daily fee and deposit. The actual book details are stored on IPFS.
- **Borrowing**: Readers pay the deposit + 1 day fee upfront to get a book. (They also have to accept our T&Cs)
- **Returning**: When a reader returns a book, the status goes to `AwaitingConfirm`.
- **Confirming**: The lender checks the book. The contract calculates the final cost and refunds whatever is left of the deposit.
- **Auto-Confirm**: If the lender forgets to confirm within 48 hours, anyone can trigger the confirmation to get the renter their refund.

### Rules & Constraints
- Max 5 active rentals per user (don't hoard the books!).
- Late penalty: 2x the daily rate if you keep it more than 7 days.
- Zero-fee lending is supported if you just want to lend a book for free.

### Dispute System (This was hard to build!)
- **Raise Dispute**: If a book comes back ruined, either party can raise a dispute.
- **Random Arbitrator**: We use `block.prevrandao` to pick a random arbitrator from the pool (excluding the lender and renter so it's fair). 
- **Fallback**: The contract deployer is automatically an arbitrator just in case the pool is empty.
- **Resolution**: The assigned arbitrator picks a winner, who gets all the locked funds.

### Frontend
We have 3 main views:
- **Lender Dashboard** - list books, confirm returns.
- **Library Catalog** - for readers to browse and borrow.
- **Arbitrator Panel** - for resolving disputes.


---

## Tech Stack
- **Smart Contract**: Solidity ^0.8.20, OpenZeppelin (Ownable, ReentrancyGuard)
- **Development**: Hardhat
- **Frontend**: Vanilla HTML/CSS/JS with ethers.js v5.7.2
- **Wallet**: MetaMask
- **Styling**: Custom warm library theme with Playfair Display serif typography

---

## Setup Instructions

### Prerequisites
- Node.js (v18+)
- npm
- MetaMask browser extension

### Installation & Deployment

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Compile the smart contract:**
   ```bash
   npx hardhat compile
   ```

3. **Start a local Hardhat node** (Terminal 1):
   ```bash
   npx hardhat node
   ```

4. **Deploy the contract** (Terminal 2):
   ```bash
   npx hardhat run scripts/deploy.js --network localhost
   ```
   Copy the deployed contract address from the output.

5. **Update the frontend config:**
   Open `frontend/config.js` and replace the `contractAddress` value with the new deployed address.

6. **Start the frontend server** (Terminal 3):
   ```bash
   cd frontend
   python3 -m http.server 8000
   ```
   Open `http://localhost:8000` in your browser.

7. **Connect MetaMask:**
   - Add the Hardhat network to MetaMask (RPC: `http://127.0.0.1:8545`, Chain ID: `31337`).
   - Import Hardhat test accounts using the private keys printed when the node started.

### Running Tests
```bash
npx hardhat test
npx hardhat coverage
```

---

## Project Structure
```
DAppBook/
├── contracts/
│   └── BookRental.sol          # Main smart contract
├── scripts/
│   └── deploy.js               # Deployment script
├── test/
│   └── BookRental.test.js      # Test suite
├── frontend/
│   ├── index.html              # Landing page
│   ├── owner.html              # Lender dashboard
│   ├── renter.html             # Reader / library catalog
│   ├── arbitrator.html         # Arbitrator panel
│   ├── config.js               # Shared contract ABI, address, wallet logic
│   ├── owner.js                # Lender page logic
│   ├── renter.js               # Reader page logic
│   ├── arbitrator.js           # Arbitrator page logic
│   └── style.css               # Library-themed UI styles
├── hardhat.config.js
└── package.json
```

---

## Gas Optimisation

**Optimisations Applied:**
- Replaced string-based `require()` statements with **Custom Errors** (e.g., `if (...) revert BookRental__InvalidPrice()`)
- Switched postfix increments (`counter++`) to prefix increments (`++counter`)

| Metric | Before | After |
|--------|--------|-------|
| Deployment Cost | ~1,100,000 gas | 1,064,310 gas |
| `listItem` Execution | ~226,500 gas | 225,911 gas |
| Deployment Cost | ~1,716,566 gas | 1,744,141 gas |
| `listItem` Execution | ~166,177 gas | 137,585 gas |
| `rentItem` Execution | ~121,957 gas | 95,388 gas |
| `returnItem` Execution | ~52,543 gas | 30,466 gas |

**Reasoning:** Custom errors eliminate on-chain string storage and encode to a compact 4-byte selector, reducing both deployment and runtime gas costs. Prefix increments avoid caching the previous variable state.

---

## Off-Chain vs On-Chain Data

We have strictly adhered to privacy requirements:

| Location | Data Stored |
|----------|-------------|
| **On-Chain** | Prices, statuses, timestamps, IPFS CID reference, wallet addresses |
| **Off-Chain (IPFS)** | Book titles, descriptions, cover images, and any personal information |

---

## Smart Contract Details

### Key Functions
| Function | Who | Description |
|----------|-----|-------------|
| `listItem()` | Lender | Add a book with daily fee + deposit |
| `rentItem()` | Reader | Borrow a book (pays deposit + 1 day fee) |
| `returnItem()` | Reader | Mark a book as returned |
| `confirmReturn()` | Lender / Anyone after 48h | Confirm return, trigger refund |
| `raiseDispute()` | Lender or Reader | Raise dispute on a returned book |
| `resolveDispute()` | Assigned Arbitrator | Resolve dispute, send funds to winner |
| `registerAsArbitrator()` | Anyone | Join the arbitrator pool |
| `unregisterAsArbitrator()` | Arbitrator | Leave the pool |

### Security Features
- **ReentrancyGuard**: All functions involving ETH transfers are protected against reentrancy attacks.
- **Custom Errors**: Gas-efficient error handling with descriptive error names.
- **Access Control**: Only the book owner, assigned renter, or assigned arbitrator can perform their respective actions.
- **Pseudo-Random Arbitrator Selection**: Uses `keccak256(block.prevrandao, block.timestamp, itemId)` for randomized dispute assignment.

---

## Known Issues / Things we couldn't fix
- The IPFS CID requires pinning. If we forget to pin it or the node goes down, the book cover might not load.
- Random arbitrator selection uses `block.prevrandao`. It works fine for our project demo, but we know miners could theoretically manipulate it on mainnet.
- Hardhat localhost only for now.
- MetaMask is weird about internal transactions (contract -> wallet refunds), so you have to actually check your balance to see the refund. It doesn't show in the activity tab.
- If someone interrupts the `confirmReturn` flow, the `activeRentals` counter might get slightly out of sync.

---

## License
MIT (feel free to use it)
