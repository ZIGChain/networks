# ZIGChain Networks

This repository contains network information for the various ZIGChain networks.

At any given time, there will be multiple active networks:

| Network     | Status | Version (Binary Version) | Description                                                       |
|-------------|--------|--------------------------|-------------------------------------------------------------------|
| **mainnet** | ✔️     | v5.0 (v5.0.0)             | The production-ready ZIGChain mainnet.                            |
| **testnet** | ✔️     | v5.0 (v5.0.0)             | The public ZIGChain testnet for developers and ecosystem testing. |

Each network has a dedicated directory containing essential configuration details. These directories include:

| File               | Description                                          |
|--------------------|------------------------------------------------------|
| `version.txt`      | Specifies the ZIGChain version used for the network. |
| `chain-id.txt`     | The `chain-id` identifier for the network.           |
| `genesis.json`     | The genesis file required to join the network.       |
| `seed-nodes.txt`   | A list of available seed node addresses.             |
| `rpc-nodes.txt`    | A list of available RPC node addresses.              |
| `api-nodes.txt`    | A list of API (LCD) node addresses.                  |


## Usage

The configuration details in this repository can be used to automate deployment and configuration tasks for ZIGChain.

All network data is structured consistently, allowing you to fetch and use information seamlessly.

```sh
ZIGCHAIN_NET_BASE="https://raw.githubusercontent.com/ZIGChain/networks-private/refs/heads/main/"
```

### Selecting a Network

```sh
# Testnet
ZIGCHAIN_NET="$ZIGCHAIN_NET_BASE/zig-test-2"
```

### Fetching Network Information

#### Get the Current Version
```sh
ZIGCHAIN_VERSION="$(curl -s "$ZIGCHAIN_NET/version.txt")"
echo "ZIGChain Version: $ZIGCHAIN_VERSION"
```

#### Get the Chain ID
```sh
ZIGCHAIN_CHAIN_ID="$(curl -s "$ZIGCHAIN_NET/chain-id.txt")"
echo "Chain ID: $ZIGCHAIN_CHAIN_ID"
```

#### Download the Genesis File
```sh
curl -s "$ZIGCHAIN_NET/genesis.json" -o genesis.json
```

#### List Seed Nodes
```sh
curl -s "$ZIGCHAIN_NET/seed-nodes.txt" | shuf -n 1
```

#### Fetch a Random RPC Node
```sh
curl -s "$ZIGCHAIN_NET/rpc-nodes.txt" | shuf -n 1
```

#### Fetch a Random API Node
```sh
curl -s "$ZIGCHAIN_NET/api-nodes.txt" | shuf -n 1
```

By following this structure, developers and operators can easily integrate ZIGChain into their infrastructure and automation workflows.

## Binaries

Pre-built `zigchaind` binaries are published under `binaries/` at the repo root:

| Path                                             | Description                                                          |
|--------------------------------------------------|----------------------------------------------------------------------|
| `binaries/zigchaind-vX.Y.Z-<os>-<arch>.tar.gz`   | Current release builds for the active mainnet/testnet version.       |
| `binaries/SHA256SUMS-vX.Y.Z.txt`                 | SHA-256 checksums for the current release archives.                  |
| `binaries/archive/`                              | Previous releases, kept for historical reference.                    |

Supported platforms: `darwin-amd64`, `darwin-arm64`, `linux-amd64`.

### Downloading a Binary

```sh
ZIGCHAIN_BINARIES="$ZIGCHAIN_NET_BASE/binaries"
ZIGCHAIN_VERSION="$(curl -s "$ZIGCHAIN_NET/version.txt")"

# Adjust for your platform
PLATFORM="linux-amd64"

curl -sL "$ZIGCHAIN_BINARIES/zigchaind-${ZIGCHAIN_VERSION}-${PLATFORM}.tar.gz" \
  -o "zigchaind-${ZIGCHAIN_VERSION}.tar.gz"

curl -sL "$ZIGCHAIN_BINARIES/SHA256SUMS-${ZIGCHAIN_VERSION}.txt" \
  -o "SHA256SUMS-${ZIGCHAIN_VERSION}.txt"

# Verify
shasum -a 256 -c "SHA256SUMS-${ZIGCHAIN_VERSION}.txt" --ignore-missing
```


## Upgrades

Per-upgrade documentation lives under [`upgrades/`](upgrades/) — one chain-neutral folder per upgrade with the impact summary, coordinates (heights, proposal, binaries), and operator/builder guides.


Note: This document is adapted for ZIGChain from the original work done by Mantra Chain Team on their [net repository](https://github.com/MANTRA-Chain/net).
