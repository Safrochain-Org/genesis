# Safrochain Testnet Node Setup Guide

This guide walks you through setting up a Safrochain testnet node and creating a validator, step by step. It’s designed for beginners and includes copy-paste commands with explanations for Linux (Ubuntu), macOS, and Windows (via WSL2). By the end, you’ll have a running node and be ready to stake as a validator.

## 📋 Prerequisites

- **System**: 
  - Linux (Ubuntu recommended), macOS, or Windows (with WSL2), 2GB RAM, 20GB disk space.
  - Windows users must install WSL2 and Ubuntu (see Step 1 for Windows).
- **Internet**: Stable connection for cloning repositories and syncing the blockchain.
- **Permissions**: Root/admin access for installing packages and configuring firewalls.
- **Testnet Tokens**: Required for validator creation. Request from the Safrochain testnet faucet at https://faucet.safrochain.com (provides 5,000 tSaf per request).
- **Main Node Info**: If syncing with an existing node (e.g., main node), obtain its node ID and address for seeding (optional, see Step 7).
- **Home Directory**: Uses `~/.safrochain` for configuration and data.
- **Testnet Denominations**: Uses `tSaf`, `tKuta`, `tHela` for testnet transactions (mainnet uses `saf`).

## 🚀 Setup Steps

### Step 1: Install Dependencies

**What it does**: Installs Go 1.23, git, make, and jq (for JSON parsing), which are required to build and run the Safrochain node. It also sets up the Go environment. Instructions vary by operating system.

**Prerequisites**: Run with administrative privileges (`sudo` on Linux/macOS, admin terminal in WSL2).

**Code**:

#### Linux (Ubuntu)
```bash
sudo apt update
sudo apt install -y git make jq
# Download and install Go 1.23
wget https://go.dev/dl/go1.23.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz
rm go1.23.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
# Set up Go environment
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
mkdir -p $GOPATH
# Verify Go version
if go version | grep -q "go1.23"; then
    echo "Go 1.23 installed successfully."
else
    echo "Error: Go 1.23 not installed. Check installation steps."
    exit 1
fi
```

#### macOS
```bash
# Install git, make, and jq
xcode-select --install || true
brew install jq || true
# Download and install Go 1.23
curl -LO https://go.dev/dl/go1.23.0.darwin-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.0.darwin-amd64.tar.gz
rm go1.23.0.darwin-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
# Set up Go environment
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
mkdir -p $GOPATH
# Make PATH changes permanent
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.zshrc
source $HOME/.zshrc
# Verify Go version
if go version | grep -q "go1.23"; then
    echo "Go 1.23 installed successfully."
else
    echo "Error: Go 1.23 not installed. Check installation steps or run 'sudo rm -rf /usr/local/go' and retry."
    exit 1
fi
```

#### Windows (via WSL2)
```bash
# Install WSL2 and Ubuntu if not already set up
# In a Windows PowerShell (run as Administrator):
# wsl --install
# After Ubuntu setup, run in Ubuntu WSL2 terminal:
sudo apt update
sudo apt install -y git make jq
# Download and install Go 1.23
wget https://go.dev/dl/go1.23.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.0.linux-amd64.tar.gz
rm go1.23.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
# Set up Go environment
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
mkdir -p $GOPATH
# Verify Go version
if go version | grep -q "go1.23"; then
    echo "Go 1.23 installed successfully."
else
    echo "Error: Go 1.23 not installed. Check installation steps."
    exit 1
fi
```

**Notes**:
- **Linux**: Installs Go 1.23 via official tarball. Use `https://go.dev/dl/` for other architectures (e.g., ARM).
- **macOS**: Installs Go 1.23 via official tarball. Requires Xcode Command Line Tools and Homebrew for jq. If using Bash, replace `.zshrc` with `.bashrc`.
- **Windows**: Requires WSL2 with Ubuntu. Run `wsl --install` in PowerShell if WSL2 isn’t set up.
- Verify with `go version` (should show `go1.23.x`). If incorrect, remove existing Go (`sudo rm -rf /usr/local/go`) and retry.
- The `GOPATH` is set to `~/go` for building the Safrochain binary. `jq` is used in later steps for troubleshooting.

---

### Step 2: Clone Repository, Build Binary, and Export Binary Path

**What it does**: Clones the Safrochain node repository, builds the `safrochaind` binary, and adds `~/go/bin` to your `PATH`. PATH configuration varies by OS.

**Prerequisites**: Git and Go 1.23 must be installed (from Step 1).

**Code**:

#### Linux (Ubuntu) / Windows (WSL2)
```bash
git clone https://github.com/Safrochain-Org/safrochain-node.git
cd safrochain-node
make install

# Add ~/go/bin to PATH
export PATH=$PATH:$HOME/go/bin
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bashrc
source $HOME/.bashrc

# Verify safrochaind
if command -v safrochaind &> /dev/null; then
    echo "safrochaind is accessible."
else
    echo "Error: safrochaind not found. Check if ~/go/bin/safrochaind exists."
    exit 1
fi
```

#### macOS
```bash
git clone https://github.com/Safrochain-Org/safrochain-node.git
cd safrochain-node
make install

# Add ~/go/bin to PATH
export PATH=$PATH:$HOME/go/bin
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.zshrc
source $HOME/.zshrc

# Verify safrochaind
if command -v safrochaind &> /dev/null; then
    echo "safrochaind is accessible."
else
    echo "Error: safrochaind not found. Check if ~/go/bin/safrochaind exists."
    exit 1
fi
```

**Notes**:
- The binary is installed in `~/go/bin/safrochaind`.
- **Linux/WSL2**: Uses `.bashrc` for PATH persistence.
- **macOS**: Uses `.zshrc`. If using Bash, replace with `.bashrc`.
- **Windows**: WSL2 uses Linux steps.
- If your shell differs (e.g., Fish), add `export PATH=$PATH:$HOME/go/bin` to its config (e.g., `~/.config/fish/config.fish`).
- Verify with `safrochaind version`.

---

### Step 3: Initialize Node

**What it does**: Initializes the node with a moniker and creates the configuration directory (`~/.safrochain`).

**Prerequisites**: `safrochaind` binary must be built.

**Code**:
```bash
echo "Enter a moniker for your node (e.g., my-node):"
read MONIKER
safrochaind init "$MONIKER" --chain-id safrochain-testnet --home $HOME/.safrochain
```

**Notes**:
- The moniker is a public node name.
- Creates `~/.safrochain/config/` with default files like `genesis.json`.

---

### Step 4: Configure Genesis File

**What it does**: Downloads the official `genesis.json` file from the Safrochain testnet repository and places it in the node’s configuration directory.

**Prerequisites**: Internet access, Step 3 completed to create the configuration directory.

**Code**:
```bash
curl -L -o $HOME/.safrochain/config/genesis.json https://raw.githubusercontent.com/Safrochain-Org/genesis/refs/heads/main/genesis-testnet.json
if [ -f "$HOME/.safrochain/config/genesis.json" ]; then
    echo "genesis.json downloaded successfully."
else
    echo "Error: Failed to download genesis.json. Check the URL, internet connection, or ensure ~/.safrochain/config/ exists."
    echo "Contact Safrochain’s community (https://github.com/Safrochain-Org, Discord, or Telegram) for the correct genesis file."
    exit 1
fi
```

**Notes**:
- Downloads from `https://raw.githubusercontent.com/Safrochain-Org/genesis/refs/heads/main/genesis-testnet.json` using `curl -L` to follow redirects.
- Overwrites any existing `genesis.json` in `~/.safrochain/config/`.
- If the download fails or later errors (e.g., `validator set is empty`) occur, verify the URL in a browser or contact the Safrochain community for an updated `genesis.json` matching the main node’s network.
- To manually inspect the file:
  ```bash
  cat $HOME/.safrochain/config/genesis.json
  ```

---

### Step 5: Configure Node Settings

**What it does**: Sets up configuration files (`app.toml`, `config.toml`, `client.toml`) with gas prices, ports, and consensus settings.

**Prerequisites**: Node initialized (Step 3).

**Code**:
```bash
export NODE_DIR=$HOME/.safrochain
echo "Enter your node's external IP (e.g., 192.168.1.100, or press Enter for local):"
read EXTERNAL_IP
if [ -z "$EXTERNAL_IP" ]; then
    EXTERNAL_IP="127.0.0.1"
fi

# app.toml
cat > "$NODE_DIR/config/app.toml" <<EOL
minimum-gas-prices = "0.001tSaf"

[api]
enable = true
swagger = true
address = "tcp://0.0.0.0:1317"

[grpc]
enable = true
address = "0.0.0.0:9090"

[grpc-web]
enable = true

[mempool]
max-txs = 5000

[telemetry]
enabled = false

[streaming.abci]
keys = []
plugin = ""
stop-node-on-err = true
EOL

# config.toml
cat > "$NODE_DIR/config/config.toml" <<EOL
proxy_app = "tcp://127.0.0.1:26658"
moniker = "$MONIKER"
db_backend = "goleveldb"
db_dir = "data"
log_level = "info"
log_format = "plain"
genesis_file = "config/genesis.json"
priv_validator_key_file = "config/priv_validator_key.json"
priv_validator_state_file = "data/priv_validator_state.json"
node_key_file = "config/node_key.json"
abci = "socket"
filter_peers = false

[rpc]
laddr = "tcp://0.0.0.0:26657"
cors_allowed_origins = []
cors_allowed_methods = ["HEAD", "GET", "POST"]
cors_allowed_headers = ["Origin", "Accept", "Content-Type", "X-Requested-With", "X-Server-Time"]
tls_cert_file = ""
tls_key_file = ""
pprof_laddr = "localhost:6060"

[p2p]
laddr = "tcp://0.0.0.0:26656"
external_address = "$EXTERNAL_IP:26656"
seeds = ""
persistent_peers = ""
pex = true

[mempool]
type = "flood"
broadcast = true

[statesync]
enable = false

[blocksync]
version = "v0"

[consensus]
timeout_propose = "3s"
timeout_propose_delta = "500ms"
timeout_prevote = "1s"
timeout_prevote_delta = "500ms"
timeout_precommit = "1s"
timeout_precommit_delta = "500ms"
timeout_commit = "5s"
create_empty_blocks = false

[storage]
discard_abci_responses = false

[tx_index]
indexer = "kv"

[instrumentation]
prometheus = true
prometheus_listen_addr = ":26660"
max_open_connections = 3
namespace = "cometbft"
EOL

# client.toml
cat > "$NODE_DIR/config/client.toml" <<EOL
chain-id = "safrochain-testnet"
keyring-backend = "os"
output = "json"
node = "tcp://localhost:26657"
EOL
```

**Notes**:
- Sets `minimum-gas-prices` to `0.001tSaf` for testnet.
- The external IP is needed for public servers or routers.
- Settings enable API, gRPC, and P2P ports, keeping sensitive ports local.

---

### Step 6: Open Required Ports

**What it does**: Configures the firewall to allow ports `26656` (P2P), `26657` (RPC), `1317` (API), and `9090` (gRPC).

**Prerequisites**: Firewall tools installed (`ufw` for Linux/WSL2, macOS firewall, Windows Firewall).

**Code**:

#### Linux (Ubuntu) / Windows (WSL2)
```bash
sudo ufw allow 26656,26657,1317,9090
sudo ufw deny 26658,6060
sudo ufw enable
```

#### macOS
```bash
# Open ports via terminal (requires sudo)
sudo /sbin/pfctl -f /etc/pf.conf
echo "pass in proto tcp from any to any port {26656, 26657, 1317, 9090}" | sudo pfctl -f -
sudo pfctl -E
# Alternatively, use System Preferences > Security & Privacy > Firewall > Firewall Options
# Add safrochaind and allow ports 26656, 26657, 1317, 9090
```

**Notes**:
- **Linux/WSL2**: Verify with `sudo ufw status`.
- **macOS**: `pfctl` is temporary; edit `/etc/pf.conf` or use GUI for persistence.
- **Windows**: For host Windows Firewall, run in PowerShell (as Administrator):
  ```powershell
  New-NetFirewallRule -DisplayName "Safrochain" -Direction Inbound -Protocol TCP -LocalPort 26656,26657,1317,9090 -Action Allow
  ```
- Open ports in cloud provider security groups if needed.

---

### Step 7: Start the Node

**What it does**: Starts the node, configures peers, and handles potential errors like OE hash mismatches.

**Prerequisites**: Configuration files and genesis file set up.

**Code**:
```bash
# Configure main node or community seeds
echo "Enter the main node’s ID@IP:26656 (e.g., 12345abcde@192.168.1.100:26656, or press Enter to skip):"
read MAIN_NODE_PEER
if [ -n "$MAIN_NODE_PEER" ]; then
    sed -i.bak "s/seeds = \"\"/seeds = \"$MAIN_NODE_PEER\"/g" $HOME/.safrochain/config/config.toml
    echo "Main node added as a seed peer."
fi

# Reset node state
safrochaind tendermint unsafe-reset-all --home $HOME/.safrochain

# Start the node
safrochaind start --home $HOME/.safrochain > safrochaind.log 2>&1 &
sleep 5
if pgrep safrochaind > /dev/null; then
    echo "Node is running. Logs are in safrochaind.log."
else
    echo "Error: Node failed to start. Check safrochaind.log for details."
    echo "If you see 'validator set is empty':"
    echo "1. Verify Step 4 (genesis.json download)."
    echo "2. Redownload genesis.json: curl -L -o $HOME/.safrochain/config/genesis.json https://raw.githubusercontent.com/Safrochain-Org/genesis/refs/heads/main/genesis-testnet.json"
    echo "3. Reset state: safrochaind tendermint unsafe-reset-all --home $HOME/.safrochain"
    echo "4. Contact Safrochain’s community for an updated genesis file."
    echo "If you see 'OE aborted due to hash mismatch':"
    echo "1. Ensure reliable peers (main node or community seeds)."
    echo "2. Reset state again."
    echo "3. Disable OE: Add 'optimistic_execution_enabled = false' under [consensus] in $HOME/.safrochain/config/config.toml"
    echo "4. Contact Safrochain’s community for OE guidance."
    exit 1
fi
```

**Notes**:
- Check logs: `tail -f safrochaind.log`.
- Verify status: `curl http://localhost:26657/status`.
- For OE hash mismatch errors, follow the troubleshooting steps or disable OE:
  ```bash
  pkill safrochaind
  echo -e "\n[consensus]\noptimistic_execution_enabled = false" >> $HOME/.safrochain/config/config.toml
  safrochaind tendermint unsafe-reset-all --home $HOME/.safrochain
  safrochaind start --home $HOME/.safrochain
  ```

---

### Step 8: Create Validator Wallet

**What it does**: Creates a wallet for validator transactions.

**Prerequisites**: Node running.

**Code**:
```bash
echo "Enter a name for your validator wallet (e.g., validator):"
read WALLET_NAME
safrochaind keys add "$WALLET_NAME" --keyring-backend os --home $HOME/.safrochain
WALLET_ADDRESS=$(safrochaind keys show "$WALLET_NAME" -a --home $HOME/.safrochain)
echo "Your wallet address is: $WALLET_ADDRESS"
echo "Visit https://faucet.safrochain.com, paste your address, and request 5,000 tSaf."
```

**Notes**:
- Save the mnemonic phrase securely (offline).
- Request tokens at `https://faucet.safrochain.com` (5,000 tSaf).
- Verify balance:
  ```bash
  safrochaind query bank balances "$WALLET_ADDRESS" --home $HOME/.safrochain
  ```
- If faucet is down, join Safrochain’s community for tokens.
- Testnet addresses use the `taddr_safro` prefix (mainnet uses `addr_safro`).

---

### Step 9: Create Validator

**What it does**: Submits a transaction to stake tokens and register your node as a validator using a JSON configuration file.

**Prerequisites**: Tokens in wallet, node fully synced (`curl http://localhost:26657/status` shows `catching_up: false`).

**Code**:
```bash
# Check sync status
curl http://localhost:26657/status | jq '.result.sync_info'
# Wait for catching_up: false
watch -n 10 "curl -s http://localhost:26657/status | jq '.result.sync_info'"

# Get validator public key
PUBKEY=$(safrochaind tendermint show-validator --home $HOME/.safrochain)

# Create validator.json
cat > $HOME/validator.json <<EOL
{
  "pubkey": $PUBKEY,
  "amount": "5000tSaf",
  "moniker": "$MONIKER",
  "identity": "",
  "website": "",
  "security": "",
  "details": "My Safrochain testnet validator",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
EOL

# Verify JSON
jq . $HOME/validator.json

# Submit validator transaction
safrochaind tx staking create-validator $HOME/validator.json \
  --from "$WALLET_NAME" \
  --chain-id safrochain-testnet \
  --fees 5tSaf \
  --keyring-backend os \
  --home $HOME/.safrochain
```

**Notes**:
- **Obtaining Tokens**: Use `https://faucet.safrochain.com` for 5,000 tSaf. If down, request tokens via Safrochain’s community.
- Check balance: `safrochaind query bank balances "$WALLET_ADDRESS" --home $HOME/.safrochain`.
- Adjust `amount` in `validator.json` if you have more/less tokens.
- Alternative denominations (`tKuta`, `tHela`) may be used if specified by the testnet; replace `tSaf` in `amount` and `--fees` as needed.
- Verify validator:
  ```bash
  safrochaind query staking validators --home $HOME/.safrochain | grep -A 5 "$MONIKER"
  ```
- If the command fails with `unknown flag`, ensure `validator.json` is correct (`jq . $HOME/validator.json`).

---

## 🛠️ Post-Setup

### Monitor the Node
- **Check logs**:
  ```bash
  tail -f safrochaind.log
  ```
- **Check status**:
  ```bash
  curl http://localhost:26657/status
  ```
- **View node ID**:
  ```bash
  safrochaind tendermint show-node-id --home $HOME/.safrochain
  ```

### Stop the Node
- Find process ID:
  ```bash
  pgrep safrochaind
  ```
- Kill process:
  ```bash
  kill <pid>
  ```

### Troubleshooting
- **Node not starting**: Check `safrochaind.log` for errors.
- **Empty validator set**:
  - Verify Step 4 download.
  - Redownload: `curl -L -o $HOME/.safrochain/config/genesis.json <correct-url>`.
  - Reset: `safrochaind tendermint unsafe-reset-all --home $HOME/.safrochain`.
  - Contact community for a matching genesis file.
- **OE hash mismatch**:
  - Configure reliable peers (Step 7).
  - Reset state: `safrochaind tendermint unsafe-reset-all --home $HOME/.safrochain`.
  - Disable OE: Add `optimistic_execution_enabled = false` under `[consensus]` in `config.toml`.
  - Contact community for guidance.
- **create-validator errors**:
  - Verify `validator.json`: `jq . $HOME/validator.json`.
  - Ensure sufficient tokens: `safrochaind query bank balances <address> --home $HOME/.safrochain`.
  - Check node sync: `curl http://localhost:26657/status`.
- **No tokens**: Use faucet or community channels.
- **safrochaind not found**: Add `~/go/bin` to PATH: `export PATH=$PATH:$HOME/go/bin`.
- **Wrong Go version**: Remove Go (`sudo rm -rf /usr/local/go`) and retry Step 1.
- **Firewall issues**: Verify ports 26656, 26657, 1317, 9090 (`sudo ufw status` or macOS/Windows firewall settings).

## 🌐 Next Steps

- **Join the Community**: Find Safrochain’s Discord, Telegram, or forum for support (check [Safrochain GitHub](https://github.com/Safrochain-Org)).
- **Run a Faucet**: Set up a faucet if needed (ask for a guide).
- **Deploy a Block Explorer**: Use tools like [Big Dipper](https://github.com/forbole/big_dipper) (ask for setup instructions).

## 📄 License

This guide is based on the Safrochain node setup process, licensed under the [MIT License](https://github.com/Safrochain-Org/safrochain-node/blob/main/LICENSE).

## 🙌 Acknowledgments

Safrochain is built using [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) and [CometBFT](https://github.com/cometbft/cometbft). Thanks to the community for their contributions!
