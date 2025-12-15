# 🐉 Hydra SDK Demo App

A **lightweight** web application for interacting with [Hydra Head Protocol](https://hydra.family) - **no `cardano-node` required!**

This demo allows you to manage a Hydra Head using only:
- **Blockfrost API** for L1 (Cardano) interactions
- **hydra-node WebSocket** for L2 (Hydra) interactions  
- **Browser-based wallet** for transaction signing

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔌 **Connect to Hydra Node** | WebSocket connection to local hydra-node(s) |
| 💰 **Wallet Management** | Generate, restore, or load wallets from `.sk` files |
| 🔑 **Hydra Key Generation** | Generate and export Hydra signing/verification keys |
| 📥 **Commit UTxOs** | Select L1 UTxOs and commit them to the Head |
| 💸 **In-Head Transactions** | Build and sign transactions inside the Head (fee=0) |
| 📊 **View Snapshots** | Watch confirmed snapshots and UTxO state changes |
| 🔒 **Head Lifecycle** | Init, Commit, Close, Fanout - full lifecycle management |
| 📋 **Event Log** | Real-time event stream from the Hydra node |
| ✂️ **UTxO Splitting** | Split large UTxOs for better commit flexibility |

## 🚀 Quick Start

### Prerequisites

- **Blockfrost API key** for Preview/Preprod network (get one free at [blockfrost.io](https://blockfrost.io))
- **Nix** (recommended) or **Node.js 18+** with **pnpm**

### Option A: Using Nix (Recommended)

```bash
# Clone the repo
git clone https://github.com/cardano-scaling/hydra-sdk-app.git
cd hydra-sdk-app

# Enter nix shell (includes Node.js, pnpm, etc.)
nix develop

# Install dependencies and start
pnpm install
pnpm dev
```

### Option B: Using Node.js directly

```bash
git clone https://github.com/cardano-scaling/hydra-sdk-app.git
cd hydra-sdk-app

# Using pnpm
pnpm install
pnpm dev

# Or using npm
npm install
npm run dev
```

Open http://localhost:5173 in your browser.

## 📋 Setup Guide

### Step 1: Configure Blockfrost & Create Wallet

1. Enter your **Blockfrost API key** (e.g., `previewXXXXX`) and click **Save**
2. Click **"✨ Generate New Wallet"** or load an existing `.sk` file
3. **Save your private key!** (shown only once for new wallets)
4. Export your keys using **"💾 Export cardano.sk"** and **"📤 Export cardano.vk"**

### Step 2: Fund Your Wallet

Your wallet needs test ADA for L1 transactions.

1. **Copy your address** from the app (`addr_test1...`)
2. Go to the **[Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)**
3. Select **"Preview Testnet"** and paste your address
4. Click **"Request Funds"** (~10,000 test ADA)
5. Wait ~20 seconds, then click ⟳ to refresh your balance

### Step 3: Generate Hydra Keys

In the web app:
1. Click **"🔑 Generate Hydra Keys"**
2. Export **"💾 hydra.sk"** and **"📤 hydra.vk"**

### Step 4: Start Hydra Node

Create a working directory with your keys:

```bash
mkdir ~/hydra-demo && cd ~/hydra-demo

# Copy your exported keys here:
# - cardano.sk, cardano.vk (from web app)
# - hydra.sk, hydra.vk (from web app)

# Create Blockfrost config
echo "previewXXXXXX" > blockfrost-project.txt

# Download protocol parameters
curl -o protocol-parameters.json \
  https://raw.githubusercontent.com/cardano-scaling/hydra/master/hydra-cluster/config/protocol-parameters.json
```

**Start hydra-node**:

```bash
nix run github:cardano-scaling/hydra/0.20.0#hydra-node -- \
  --node-id alice \
  --api-host 0.0.0.0 \
  --api-port 4001 \
  --blockfrost ./blockfrost-project.txt \
  --hydra-signing-key ./hydra.sk \
  --cardano-signing-key ./cardano.sk \
  --hydra-scripts-tx-id "SCRIPT_IDS" \
  --ledger-protocol-parameters ./protocol-parameters.json \
  --persistence-dir ./persistence \
  --contestation-period 60s
```

> **Note:** The `--hydra-scripts-tx-id` above is for Hydra v0.21.0 on Preview. Check [hydra-node/networks.json](https://github.com/cardano-scaling/hydra/blob/master/hydra-node/networks.json) for the specific versions/networks of `--hydra-scripts-tx-id`.

### Step 5: Connect from Web App

1. Set **WebSocket URL** to `ws://localhost:4001`
2. Click **Connect**
3. You should see "Status: connected" and the Head state

## 📖 Head Lifecycle

| Action | State Required | Description |
|--------|----------------|-------------|
| **🚀 Init** | Idle | Initialize a new Head |
| **📥 Commit** | Initializing | Commit L1 UTxOs to the Head |
| **✕ Abort** | Initializing | Abort before all parties commit |
| **💸 Send** | Open | Submit transactions in the Head |
| **🔒 Close** | Open | Initiate Head closure |
| **📤 Fanout** | FanoutPossible | Distribute final UTxOs back to L1 |

---

## 🌐 Network Configuration

The app auto-detects your network based on Blockfrost API key prefix:

| API Key Prefix | Network | 
|----------------|---------|
| `mainnet...` | Mainnet |
| `preview...` | Preview |
| `preprod...` | Preprod |

Find the correct `--hydra-scripts-tx-id` for your version in:
👉 [hydra-node/networks.json](https://github.com/cardano-scaling/hydra/blob/master/hydra-node/networks.json)

## 🔧 Troubleshooting

| Error | Solution |
|-------|----------|
| **"NoUTxOFound"** when starting hydra-node | Fund your `cardano.sk` wallet via faucet |
| **"WebSocket connection failed"** | Make sure hydra-node is running |
| **"404 Not Found"** fetching UTxOs | Normal for new wallets - fund via faucet |
| **"Blockfrost error: 403"** | API key is for wrong network |
| **"ScriptFailedInWallet / MissingScript"** | Script version mismatch - check `--hydra-scripts-tx-id` matches your hydra-node version |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Browser (React App)                      │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Wallet     │  │  Blockfrost │  │  Hydra WebSocket   │ │
│  │  (.sk file) │  │  Provider   │  │  Connection        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
          │                    │                    │
          ▼                    ▼                    ▼
   ┌────────────┐      ┌────────────────┐   ┌────────────────┐
   │ Hydra Node │      │   Blockfrost   │   │   Cardano L1   │
   │  ws:4001   │      │   (preview)    │──▶│  (via BF API)  │
   └────────────┘      └────────────────┘   └────────────────┘
```

---

## 🔧 Technology Stack

- **React 19** + **TypeScript** - UI framework
- **Vite** - Build tool
- **Tailwind CSS** - Styling
- **[@hydra-sdk](https://github.com/ADAPhilippines/hydra-sdk)** - Hydra client SDK
- **Blockfrost API** - L1 chain queries and tx submission

## 📚 References

- [Hydra Documentation](https://hydra.family/head-protocol/)
- [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnets/tools/faucet)
- [Blockfrost API](https://blockfrost.io/docs)
- [Hydra SDK](https://github.com/ADAPhilippines/hydra-sdk)

## License

Apache 2.0
