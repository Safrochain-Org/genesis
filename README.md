# Safrochain Testnet Node Setup Guide

This guide walks you through setting up a Safrochain testnet node and creating a validator, step by step. It’s designed for beginners and includes copy-paste commands with explanations. By the end, you’ll have a running node and be ready to stake as a validator.

## 📋 Prerequisites

- **System**: Linux (Ubuntu recommended), 2GB RAM, 20GB disk space.
- **Internet**: Stable connection for cloning repositories and syncing the blockchain.
- **Permissions**: Root access for installing packages and configuring firewalls (use `sudo` if needed).
- **Genesis File**: You’ll need the Safrochain testnet `genesis.json` file (check [Safrochain GitHub](https://github.com/Safrochain-Org) or community channels).
- **Testnet Tokens**: Required for validator creation (request from a Safrochain faucet or community).

## 🚀 Setup Steps

### Step 1: Install Dependencies

**What it does**: Installs Go, git, and make, which are required to build and run the Safrochain node. It also sets up the Go environment.

**Prerequisites**: Run as a user with `sudo` privileges.

**Code**:
```bash
sudo apt update
sudo apt install -y golang git make
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
mkdir -p $GOPATH
```

**Notes**:
- If Go is already installed, you can skip the `apt install` lines but ensure `GOPATH` is set.
- Verify Go installation with `go version` (should be 1.18 or higher).

---

### Step 2: Clone Repository, Build Binary, and Export Binary Path

**What it does**: Clones the Safrochain node repository, builds the `safrochaind` binary, and adds the binary’s directory (`~/go/bin`) to your `PATH` so you can run `safrochaind` from any terminal directory.

**Prerequisites**: Git and Go must be installed (from Step 1).

**Code**:
```bash
git clone https://github.com/Safrochain-Org/safrochain-node.git
cd safrochain-node
make install

# Add ~/go/bin to PATH for the current session
export PATH=$PATH:$HOME/go/bin

# Make PATH change permanent by adding to shell configuration
if [ -f "$HOME/.bashrc" ]; then
    echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.bashrc
    source $HOME/.bashrc
elif [ -f "$HOME/.zshrc" ]; then
    echo 'export PATH=$PATH:$HOME/go/bin' >> $HOME/.zshrc
    source $HOME/.zshrc
else
    echo "Warning: No .bashrc or .zshrc found. Manually add 'export PATH=$PATH:$HOME/go/bin' to your shell configuration file."
fi

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
- The `export PATH` command makes `safrochaind` accessible in the current session.
- Adding to `~/.bashrc` or `~/.zshrc` ensures it’s available in future sessions. If you use a different shell (e.g., `fish`), manually add the export to its config file.
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

---

### Step 4: Configure Genesis File

**What it does**: Downloads or copies the testnet `genesis.json` file, which defines the initial state of the Safrochain testnet.

**Prerequisites**: You need the URL or local path to the testnet `genesis.json`. Check Safrochain’s GitHub, website, or community (e.g., Discord/Telegram).

**Code**:
```bash
echo "Enter the URL or local path to the testnet genesis.json file (e.g., https://raw.githubusercontent.com/Safrochain-Org/testnet/main/genesis.json):"
read GENESIS_SOURCE
if [[ $GENESIS_SOURCE == http* ]]; then
    curl -o $HOME/.safrochain/config/genesis.json "$GENESIS_SOURCE"
else
    cp "$GENESIS_SOURCE" $HOME/.safrochain/config/genesis.json
fi
```

**Notes**:
- If the genesis file is invalid or missing, the node won’t sync. Ensure the source is official.
- Verify the file exists with `ls ~/.safrochain/config/genesis.json`.

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

---

### Step 6: Open Required Ports

**What it does**: Configures the firewall to allow traffic on ports `26656` (P2P), `26657` (RPC), `1317` (API), and `9090` (gRPC), while blocking sensitive ports.

**Prerequisites**: `ufw` must be installed (included in most Ubuntu distributions). Run as `sudo`.

**Code**:
```bash
sudo ufw allow 26656,26657,1317,9090
sudo ufw deny 26658,6060
sudo ufw enable
```

**Notes**:
- Confirm firewall status with `sudo ufw status`.
- If you’re on a cloud server, also open these ports in your provider’s security group (e.g., AWS, GCP).

---

### Step 7: Start the Node

**What it does**: Starts the Safrochain node in the background and logs output to a file for debugging.

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

---

### Step 8: Create Validator Wallet

**What it does**: Creates a wallet for your validator, which will hold testnet tokens and sign validator transactions.

**Prerequisites**: Node must be running.

**Code**:
```bash
echo "Enter a name for your validator wallet (e.g., validator):"
read WALLET_NAME
safrochaind keys add "$WALLET_NAME"
```

**Notes**:
- Save the mnemonic phrase securely (e.g., write it down offline). Losing it means losing access to your wallet.
- The wallet address will be displayed; share it with a faucet to receive testnet tokens.

---

### Step 9: Create Validator

**What it does**: Submits a transaction to stake tokens and register your node as a validator on the testnet.

**Prerequisites**: You need testnet tokens (`saf`) in your wallet. Request them from a Safrochain faucet or community. Your node must be fully synced (check with `curl http://localhost:26657/status` and ensure `catching_up: false`).

**Code**:
```bash
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
- Replace `<amount>` with the number of tokens to stake (e.g., `1000saf`).
- Ensure your node is running when executing this command.
- Verify validator status with `safrochaind query staking validators`.

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
- **No tokens**: Join Safrochain’s Discord, Telegram, or forum to request testnet tokens.
- **safrochaind not found**: Ensure `~/go/bin` is in your `PATH` (run `export PATH=$PATH:$HOME/go/bin` or check your shell config file).

## 🌐 Next Steps

- **Join the Community**: Find Safrochain’s Discord, Telegram, or forum for support, faucet access, or testnet updates.
- **Run a Faucet**: If no faucet exists, set up your own using a wallet with tokens and a simple web app (ask for a guide if needed).
- **Deploy a Block Explorer**: Use tools like [Big Dipper](https://github.com/forbole/big_dipper) to visualize the testnet (ask for setup instructions).

## 📄 License

This guide is based on the Safrochain node setup process, licensed under the [MIT License](https://github.com/Safrochain-Org/safrochain-node/blob/main/LICENSE).

## 🙌 Acknowledgments

Safrochain is built using [Cosmos SDK](https://github.com/cosmos/cosmos-sdk) and [CometBFT](https://github.com/cometbft/cometbft). Thanks to the community for their contributions!
