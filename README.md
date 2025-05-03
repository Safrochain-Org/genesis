# Safrochain Testnet Node Setup Guide

This guide walks you through setting up a Safrochain testnet node and creating a validator, step by step. It’s designed for beginners and includes copy-paste commands with explanations for Linux (Ubuntu), macOS, and Windows (via WSL2). By the end, you’ll have a running node and be ready to stake as a validator.

## 📋 Prerequisites

- **System**: 
  - Linux (Ubuntu recommended), macOS, or Windows (with WSL2), 2GB RAM, 20GB disk space.
  - Windows users must install WSL2 and Ubuntu (see Step 1 for Windows).
- **Internet**: Stable connection for cloning repositories and syncing the blockchain.
- **Permissions**: Root/admin access for installing packages and configuring firewalls.
- **Testnet Tokens**: Required for validator creation. Request from the Safrochain testnet faucet at https://faucet.safrochain.com (provides 5,000,000,000 saf per request).

## 🚀 Setup Steps

### Step 1: Install Dependencies

**What it does**: Installs Go 1.23, git, and make, which are required to build and run the Safrochain node. It also sets up the Go environment. Instructions vary by operating system.

**Prerequisites**: Run with administrative privileges (`sudo` on Linux/macOS, admin terminal in WSL2).

**Code**:

#### Linux (Ubuntu)
```bash
sudo apt update
sudo apt install -y git make
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
# Install Homebrew if not present
if ! command -v brew &> /dev/null; then
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
fi
brew install git make go@1.23
# Set up Go environment
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin:/usr/local/opt/go@1.23/bin
mkdir -p $GOPATH
# Verify Go version
if go version | grep -q "go1.23"; then
    echo "Go 1.23 installed successfully."
else
    echo "Error: Go 1.23 not installed. Check installation steps."
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
sudo apt install -y git make
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
- **macOS**: Uses Homebrew to install `go@1.23`. Ensure Homebrew is installed.
- **Windows**: Requires WSL2 with Ubuntu. Run `wsl --install` in PowerShell if WSL2 isn’t set up, then follow Linux steps in the Ubuntu WSL2 terminal.
- Verify with `go version` (should show `go1.23.x`). If incorrect, remove existing Go (`sudo rm -rf /usr/local/go` on Linux/WSL2, `brew uninstall go` on macOS) and retry.
- The `GOPATH` is set to `~/go` for building the Safrochain binary.

---

### Step 2: Clone Repository, Build Binary, and Export Binary Path

**What it does**: Clones the Safrochain node repository, builds the `safrochaind` binary, and adds the binary’s directory (`~/go/bin`) to your `PATH` so you can run `safrochaind` from any terminal directory. PATH configuration varies by OS.

**Prerequisites**: Git and Go 1.23 must be installed (from Step 1).

**Code**:

#### Linux (Ubuntu) / Windows (WSL2)
```bash
git clone https://github.com/Safrochain-Org/safrochain-node.git
cd safrochain-node
make install

# Add ~/go/bin to PATH for the current session
export PATH=$PATH:$HOME/go/bin

# Make PATH change permanent
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bashrc
source $HOME/.bashrc

# Verify safrochaind is accessible
if command -v safrochaind &> /dev/null; then
    echo "safrochaind is accessible from the command line."
else
    echo "Error: safrochaind not found in PATH. Check if ~/go/bin/safrochaind exists and PATH is set correctly."
    exit 1
fi
```

#### macOS
```bash
git clone https://github.com/Safrochain-Org/safrochain-node.git
cd safrochain-node
make install

# Add ~/go/bin to PATH for the current session
export PATH=$PATH:$HOME/go/bin

# Make PATH change permanent
echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.zshrc
source $HOME/.zshrc

# Verify safrochaind is accessible
if command -v safrochaind &> /dev/null; then
    echo "safrochaind is accessible from the command line."
else
    echo "Error: safrochaind not found in PATH. Check if ~/go/bin/safrochaind exists and PATH is set correctly."
    exit 1
fi
```

**Notes**:
- The binary is installed in `~/go/bin/safrochaind`.
- **Linux/WSL2**: Uses `.bashrc` for PATH persistence.
- **macOS**: Uses `.zshrc` (default for modern macOS). If using Bash, replace `.zshrc` with `.bashrc`.
- **Windows**: WSL2 uses the Linux steps, as it runs Ubuntu.
- If your shell differs (e.g., Fish), manually add `export PATH=$PATH:$HOME/go/bin` to its config (e.g., `~/.config/fish/config.fish`).
- Verify with `safrochaind version`. If it fails, check if `~/go/bin/safrochaind` exists and retry the `export` commands.

---

### Step 3: Initialize Node

**What it does**: Initializes the node with a user-defined moniker (node name) and creates the default configuration directory (`~/.safrochain`).

**Prerequisites**: `safrochaind` binary must be built and accessible.

**Code**:
```bash
echo "Enter a moniker for your node (e.g., my-node):"
read MONIKER
safrochaind init "$MONIKER" --chain-id safrochain-testnet
```

**Notes**:
- The moniker is a public name for your node. Choose something unique.
- This creates `~/.safrochain/config/` with default files like `genesis.json` (which we’ll replace).
- This step is identical across Linux, macOS, and Windows (WSL2).

---

### Step 4: Configure Genesis File

**What it does**: Downloads the official Safrochain testnet `genesis.json` file from the provided URL and places it in the node’s configuration directory.

**Prerequisites**: Internet access to download the file.

**Code**:
```bash
curl -o $HOME/.safrochain/config/genesis.json https://raw.githubusercontent.com/Safrochain-Org/genesis/refs/heads/main/genesis-testnet.json
if [ -f "$HOME/.safrochain/config/genesis.json" ]; then
    echo "genesis.json downloaded successfully."
else
    echo "Error: Failed to download genesis.json. Check the URL or internet connection."
    exit 1
fi
```

**Notes**:
- The genesis file is sourced from `https://raw.githubusercontent.com/Safrochain-Org/genesis/refs/heads/main/genesis-testnet.json`.
- If the download fails, verify the URL or check Safrochain’s GitHub/community for an updated link.
- Confirm the file exists with `ls ~/.safrochain/config/genesis.json`.
- This step is identical across all platforms.

---

### Step 5: Configure Node Settings

**What it does**: Sets up configuration files (`app.toml`, `config.toml`, `client.toml`) with settings for gas prices, ports, consensus, and more. It also prompts for your node’s external IP.

**Prerequisites**: Node must be initialized (Step 3).

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
minimum-gas-prices = "0.001saf"

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
- The external IP is needed if your node is on a public server or behind a router (use your public IP or local network IP like `192.168.x.x`).
- These settings enable API, gRPC, and P2P ports while keeping sensitive ports (e.g., `26658`, `6060`) local.
- This step is identical across all platforms.

---

### Step 6: Open Required Ports

**What it does**: Configures the firewall to allow traffic on ports `26656` (P2P), `26657` (RPC), `1317` (API), and `9090` (gRPC), while blocking sensitive ports.

**Prerequisites**: Firewall tools must be installed (`ufw` on Linux/WSL2, macOS firewall, or Windows Firewall).

**Code**:

#### Linux (Ubuntu) / Windows (WSL2)
```bash
sudo ufw allow 26656,26657,1317,9090
sudo ufw deny 26658,6060
sudo ufw enable
```

#### macOS
```bash
# macOS uses pfctl or GUI firewall; ufw is not standard
# Open ports via terminal (requires sudo)
sudo /sbin/pfctl -f /etc/pf.conf
echo "pass in proto tcp from any to any port {26656, 26657, 1317, 9090}" | sudo pfctl -f -
sudo pfctl -E
# Alternatively, open System Preferences > Security & Privacy > Firewall > Firewall Options
# Add safrochaind and allow ports 26656, 26657, 1317, 9090
```

**Notes**:
- **Linux/WSL2**: Uses `ufw`. Confirm with `sudo ufw status`.
- **macOS**: Uses `pfctl` or System Preferences. The `pfctl` command is temporary; for persistence, edit `/etc/pf.conf` or use the GUI.
- **Windows**: WSL2 uses the Linux firewall (ufw). For the host Windows Firewall, open ports manually:
  - Run in PowerShell (as Administrator):
    ```powershell
    New-NetFirewallRule -DisplayName "Safrochain" -Direction Inbound -Protocol TCP -LocalPort 26656,26657,1317,9090 -Action Allow
    ```
- If on a cloud server, open these ports in your provider’s security group (e.g., AWS, GCP).
- This step is platform-specific due to firewall differences.

---

### Step 7: Start the Node

**What it does**: Starts the Safrochain node in the background, redirects output to a log file, and verifies the node is running.

**Prerequisites**: Configuration files and genesis file must be set up.

**Code**:
```bash
safrochaind start > safrochaind.log 2>&1 &
sleep 5
if pgrep safrochaind > /dev/null; then
    echo "Node is running. Logs are in safrochaind.log."
else
    echo "Error: Node failed to start. Check safrochaind.log."
    exit 1
fi
```

**Notes**:
- Check logs with `cat safrochaind.log` or `tail -f safrochaind.log` if the node fails to start.
- Verify node status with `curl http://localhost:26657/status`. It may take time to sync.
- This step is identical across Linux, macOS, and Windows (WSL2).

---

### Step 8: Create Validator Wallet

**What it does**: Creates a wallet for your validator, which will hold testnet tokens and sign validator transactions.

**Prerequisites**: Node must be running.

**Code**:
```bash
echo "Enter a name for your validator wallet (e.g., validator):"
read WALLET_NAME
safrochaind keys add "$WALLET_NAME"
echo "Your wallet address is displayed above. Visit https://faucet.safrochain.com, paste your address, and request 5,000,000,000 saf (5,000,000,000,000,000 Lovelace)."
```

**Notes**:
- Save the mnemonic phrase securely (e.g., write it down offline). Losing it means losing access to your wallet.
- Go to `https://faucet.safrochain.com`, enter your wallet address, and request tokens. Each request provides **5,000,000,000 saf** (equivalent to 5,000,000,000,000,000 Lovelace, as 1 saf = 1,000,000 Lovelace).
- Verify token receipt with:
  ```bash
  safrochaind query bank balances <your-wallet-address>
  ```
- The faucet may have rate limits or require CAPTCHA. If unavailable, join Safrochain’s Discord, Telegram, or forum (check [Safrochain GitHub](https://github.com/Safrochain-Org)) to request tokens.
- This step is identical across all platforms.

---

### Step 9: Create Validator

**What it does**: Submits a transaction to stake tokens and register your node as a validator on the testnet.

**Prerequisites**: You need testnet tokens (`saf`) in your wallet from the faucet. Your node must be fully synced (check with `curl http://localhost:26657/status` and ensure `catching_up: false`).

**Code**:
```bash
# Before running, ensure you have 5,000,000,000 saf from https://faucet.safrochain.com
# Check balance with: safrochaind query bank balances <your-wallet-address>
safrochaind tx staking create-validator \
  --amount <amount>saf \
  --pubkey $(safrochaind tendermint show-validator) \
  --moniker "$MONIKER" \
  --chain-id safrochain-testnet \
  --commission-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from "$WALLET_NAME"
```

**Notes**:
- **Obtaining Tokens**: Use the faucet at `https://faucet.safrochain.com` to request **5,000,000,000 saf** per request (5,000,000,000,000,000 Lovelace). If the faucet is down, check [Safrochain GitHub](https://github.com/Safrochain-Org) or join community channels (Discord, Telegram, or forum) to request tokens.
- Replace `<amount>` with the number of tokens to stake (e.g., `5000000000saf` for one faucet request). Check testnet rules for minimum staking amounts via the faucet or community.
- Ensure your node is running when executing this command.
- Verify validator status with `safrochaind query staking validators`.
- This step is identical across all platforms.

---

## 🛠️ Post-Setup

### Monitor the Node
- **Check logs**:
  ```bash
  journalctl -fu safrochaind
  ```
- **Check status**:
  ```bash
  curl http://localhost:26657/status
  ```
- **View node ID** (useful for connecting other nodes):
  ```bash
  safrochaind tendermint show-node-id
  ```

### Stop the Node
- Find the process ID:
  ```bash
  pgrep safrochaind
  ```
- Kill the process (replace `<pid>` with the ID):
  ```bash
  kill <pid>
  ```

### Troubleshooting
- **Node not starting**: Check `safrochaind.log` for errors (e.g., invalid genesis file, port conflicts).
- **Sync issues**: Ensure the genesis file is correct and ports are open. Consider enabling state sync in `config.toml` if the blockchain is large.
- **No tokens**: Use the faucet at `https://faucet.safrochain.com` (provides 5,000,000,000 saf). If unavailable, join Safrochain’s Discord, Telegram, or forum (check [Safrochain GitHub](https://github.com/Safrochain-Org)).
- **safrochaind not found**: Ensure `~/go/bin` is in your `PATH` (run `export PATH=$PATH:$HOME/go/bin` or check your shell config file).
- **Wrong Go version**: If `go version` doesn’t show `go1.23.x`, remove existing Go (`sudo rm -rf /usr/local/go` on Linux/WSL2, `brew uninstall go` on macOS) and retry Step 1.
- **Firewall issues**: Verify ports 26656, 26657, 1317, 9090 are open (`sudo ufw status` on Linux/WSL2, check macOS/Windows firewall settings).

## 🌐 Next Steps

- **Join the Community**: Find Safrochain’s Discord, Telegram, or forum for support, faucet updates, or testnet news (check [Safrochain GitHub](https://github.com/Safrochain-Org)).
- **Run a Faucet**: If the faucet at `https://faucet.safrochain.com` is insufficient, set up your own using a wallet with tokens and a web app (ask for a guide).
- **Deploy a Block Explorer**: Use tools like [Big Dipper](https://github.com/forbole/big_dipper) to visualize the testnet (ask for setup instructions).

## 📄 License

This guide is based on the Safrochain node setup process, licensed under the [MIT License](https://github.com/Safrochain-Org/safrochain-node/blob/main/LICENSE).

## 🙌 Acknowledgments

Safrochain is built using [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) and [CometBFT](https://github.com/cometbft/cometbft). Thanks to the community for their contributions!
