# TELx Rewards Calculator - Setup Guide

Complete guide for running the TELx Rewards Calculator for LP fee distribution calculations.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Running Period 25](#running-period-25)
- [Output Files](#output-files)
- [Troubleshooting](#troubleshooting)
- [Repository Structure](#repository-structure)

---

## Prerequisites

### Required Software
- **Node.js v18.10.0** (recommended, also works with v22)
- **npm** (comes with Node.js)
- **Git**

### RPC Access
You need RPC endpoints with **archive node access** for:
- Polygon Mainnet
- Base Mainnet  
- Ethereum Mainnet (optional, for future use)

**Recommended providers:**
- Alchemy (Growth plan or higher for archive access)
- Infura
- Your own archive nodes

---

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/telcoin-application-network-issuance.git
cd telcoin-application-network-issuance
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment

Create a `.env` file in the root directory:

```bash
POLYGON_RPC_URL="https://polygon-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
BASE_RPC_URL="https://base-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
MAINNET_RPC_URL="https://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
```

**⚠️ Important:** Replace `YOUR_API_KEY` with your actual API keys.

### 4. Build the Project

```bash
npm run build
```

### 5. Run the Calculator

For period 25, run all three pools:

```bash
# Polygon WETH/TEL pool
node dist/calculators/TELxRewardsCalculator.js 0x25412ca33f9a2069f0520708da3f70a7843374dd46dc1c7e62f6d5002f5f9fa7:25

# Polygon USDC/EMXN pool
node dist/calculators/TELxRewardsCalculator.js 0x29f94ec9b66df7fe4068e2d7e9bf0147b49afcdc7cd3283dff03088b8026169f:25

# Base ETH/TEL pool
node dist/calculators/TELxRewardsCalculator.js 0x727b2741ac2b2df8bc9185e1de972661519fc07b156057eeed9b07c50e08829b:25
```

### 6. Generate Excel Reports

```bash
node dist/telxHumanReadable.js
```

This generates human-readable Excel files from the JSON outputs.

---

## Configuration

### Period Configuration

The calculator is configured for **periods 0-25** in `backend/calculators/TELxRewardsCalculator.ts`:

```typescript
export const PERIODS = Array.from({ length: 26 }, (_, i) => i);
```

### Block Numbers

Period boundaries are defined per network:

**Polygon:**
- Period 24: 82,219,206 → 82,521,596 (Jan 21-28, 2026)
- Period 25: 82,521,597 → next period start (Jan 28 - Feb 4, 2026)

**Base:**
- Period 24: 41,384,527 → 41,686,926 (Jan 21-28, 2026)
- Period 25: 41,686,927 → next period start (Jan 28 - Feb 4, 2026)

### Pool IDs

Three pools are configured:

1. **Polygon WETH/TEL:** `0x25412ca33f9a2069f0520708da3f70a7843374dd46dc1c7e62f6d5002f5f9fa7`
2. **Polygon USDC/EMXN:** `0x29f94ec9b66df7fe4068e2d7e9bf0147b49afcdc7cd3283dff03088b8026169f`
3. **Base ETH/TEL:** `0x727b2741ac2b2df8bc9185e1de972661519fc07b156057eeed9b07c50e08829b`

---

## Running Period 25

### Important Notes

1. **Sequential Processing:** Periods must be run in order. Period 25 requires period 24 checkpoint files to exist.

2. **Checkpoint Files:** The calculator uses checkpoint files in `backend/checkpoints/` to track position state between periods.

3. **First Time Setup:** If you don't have checkpoint files, you'll need to run from period 0 or use existing checkpoints.

### Step-by-Step for Period 25

```bash
# 1. Ensure you have period 24 checkpoints (already included in this repo)

# 2. Run each pool for period 25
node dist/calculators/TELxRewardsCalculator.js 0x25412ca33f9a2069f0520708da3f70a7843374dd46dc1c7e62f6d5002f5f9fa7:25
node dist/calculators/TELxRewardsCalculator.js 0x29f94ec9b66df7fe4068e2d7e9bf0147b49afcdc7cd3283dff03088b8026169f:25
node dist/calculators/TELxRewardsCalculator.js 0x727b2741ac2b2df8bc9185e1de972661519fc07b156057eeed9b07c50e08829b:25

# 3. Generate Excel reports
node dist/telxHumanReadable.js

# 4. Results are saved to backend/checkpoints/
# Copy to output/ directory if needed:
cp backend/checkpoints/*-25.* output/
```

### Expected Output

Each pool calculation will:
- Load previous checkpoint state
- Analyze fee growth events for the period
- Process liquidity modifications
- Apply JIT/ACTIVE/PASSIVE classifications
- Calculate weighted fees
- Generate reward distributions
- Save new checkpoint

Example output:
```
Checkpoint file found, loading previous state...
Analyzing fees from block 82219206 to 82521596...
[Info] Skipping 'unsubscribe' for token 68477: not part of this pool.
Using passive threshold: 43200 blocks
Analysis complete. New checkpoint saved.
```

---

## Output Files

### JSON Files
Located in `backend/checkpoints/`:

- `polygon-ETH-TEL-25.json`
- `polygon-USDC-EMXN-25.json`
- `base-ETH-TEL-25.json`

**Structure:**
```json
{
  "blockRange": {
    "network": "Polygon",
    "startBlock": "82219206n",
    "endBlock": "82521596n"
  },
  "poolId": "0x...",
  "positions": [...],
  "lpData": [
    ["0xAddress", {
      "periodFeesCurrency0": "12345n",
      "periodFeesCurrency1": "67890n",
      "reward": "100000n"
    }]
  ]
}
```

### Excel Files
Human-readable spreadsheets in `backend/checkpoints/`:

- `polygon-ETH-TEL-25.xlsx`
- `polygon-USDC-EMXN-25.xlsx`
- `base-ETH-TEL-25.xlsx`

**Contains:**
- LP addresses
- Position details (tick ranges, liquidity)
- Fee growth (Currency0 & Currency1)
- Weighted fees
- Final rewards
- Position designations (JIT/ACTIVE/PASSIVE)

---

## Troubleshooting

### Common Issues

#### 1. "Invalid period" Error
```
Error: Invalid period, must be [0:25]
```
**Solution:** Ensure `PERIODS` array in `TELxRewardsCalculator.ts` is configured correctly:
```typescript
export const PERIODS = Array.from({ length: 26 }, (_, i) => i);
```

#### 2. "No checkpoint file found" Error
```
Error: No checkpoint file found. Period must be 0 for first runs
```
**Solution:** 
- Run periods sequentially starting from 0, OR
- Ensure checkpoint files from previous periods exist in `backend/checkpoints/`

#### 3. "Provided startBlock does not correspond" Error
```
Error: Provided startBlock (82219205) does not correspond to lastProcessedBlock + 1 (82219206)
```
**Solution:** Check that your period start blocks match the previous period's end blocks exactly.

#### 4. RPC Rate Limiting
```
HttpRequestError: HTTP request failed. Rate limit exceeded
```
**Solutions:**
- Upgrade to Alchemy Growth plan or higher
- Use multiple API keys and rotate between them
- Add delays between RPC calls
- Use a fanout service to distribute load

#### 5. Archive Node Access Required
```
Error: missing trie node
```
**Solution:** Ensure your RPC provider has archive node access enabled. Standard nodes only keep recent state.

#### 6. TypeScript Compilation Errors
```
error TS7053: Element implicitly has an 'any' type
```
**Solution:** Ensure `backend/config.ts` includes Base chain in `telToken` config:
```typescript
[ChainId.Base]: {
  address: getAddress("0x467Bccd9d29f223BcE8043b84E8C8B282827790F"),
  decimals: 2n,
  chain: ChainId.Base,
},
```

#### 7. Node Version Issues
If using Node v22 instead of v18:
```bash
# Use npx to specify compatible versions
npx -p ts-node@10.9.1 -p typescript@5.0.2 ts-node backend/calculators/TELxRewardsCalculator.ts 0x...:25
```

---

## Repository Structure

```
telcoin-application-network-issuance/
├── backend/
│   ├── calculators/
│   │   ├── TELxRewardsCalculator.ts    # Main LP rewards calculator
│   │   ├── StakerIncentivesCalculator.ts
│   │   └── ...
│   ├── checkpoints/                     # Checkpoint JSON & Excel files
│   │   ├── polygon-ETH-TEL-25.json
│   │   ├── polygon-ETH-TEL-25.xlsx
│   │   └── ...
│   ├── config.ts                        # Network & chain configuration
│   ├── helpers.ts                       # Utility functions
│   └── telxHumanReadable.ts            # Excel generator
├── output/                              # Generated output files
├── dist/                                # Compiled JavaScript
├── src/                                 # Smart contracts
├── test/                                # Test files
├── .env                                 # RPC configuration (create this)
├── package.json
├── tsconfig.json
└── README.md
```

### Key Files

- **`backend/calculators/TELxRewardsCalculator.ts`**: Core calculator logic
  - Period definitions
  - Network configurations
  - Position processing
  - Fee calculations
  - Reward distributions

- **`backend/config.ts`**: Chain configurations
  - RPC URLs
  - Token addresses
  - Network parameters

- **`backend/telxHumanReadable.ts`**: Converts JSON to Excel
  - Fetches token decimals
  - Formats human-readable output
  - Generates position reports

- **`backend/checkpoints/`**: Persistent state storage
  - Position states
  - Liquidity modifications
  - Fee growth tracking
  - Reward calculations

---

## Advanced Usage

### Running Specific Block Ranges

You can manually specify block ranges if needed (though not recommended for official distributions):

```bash
# This will fail validation - periods must use official boundaries
node dist/calculators/TELxRewardsCalculator.js 0x...:25 --start 82219206 --end 82521596
```

### Batch Processing

To run all pools for a period:

```bash
#!/bin/bash
PERIOD=25

# Run all three pools
node dist/calculators/TELxRewardsCalculator.js 0x25412ca33f9a2069f0520708da3f70a7843374dd46dc1c7e62f6d5002f5f9fa7:$PERIOD
node dist/calculators/TELxRewardsCalculator.js 0x29f94ec9b66df7fe4068e2d7e9bf0147b49afcdc7cd3283dff03088b8026169f:$PERIOD
node dist/calculators/TELxRewardsCalculator.js 0x727b2741ac2b2df8bc9185e1de972661519fc07b156057eeed9b07c50e08829b:$PERIOD

# Generate Excel
node dist/telxHumanReadable.js

# Copy to output
cp backend/checkpoints/*-$PERIOD.* output/
```

### Docker Setup (Optional)

```bash
# Build Docker image
docker build -t telx-calculator .

# Run in container
docker run -it --rm \
  -e POLYGON_RPC_URL="your_url" \
  -e BASE_RPC_URL="your_url" \
  -v $(pwd)/output:/app/output \
  telx-calculator
```

---

## Testing

### Run Backend Tests

```bash
npm test
```

### Run Specific Calculator Tests

```bash
npx jest backend/test/TELxRewardsCalculator.test.ts
```

### Smart Contract Tests (Requires Foundry)

```bash
forge test
```

---

## Support

For issues or questions:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review logs in `output/` directory
3. Verify RPC endpoint connectivity
4. Ensure archive node access
5. Check that checkpoint files exist for previous periods

---

## License

This project is licensed under MIT OR Apache-2.0.

