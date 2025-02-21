# ZIGChain Networks

This repository contains network information for the various ZIGChain networks.

At any given time, there will be multiple active networks:

| Network     | Status | Version (Binary Version) | Description                                                       |
|-------------|--------|--------------------------|-------------------------------------------------------------------|
| **mainnet** | Soon   | Soon                     | The production-ready ZIGChain mainnet.                            |
| **testnet** | ✔️     | v0 (0.3.0)               | The public ZIGChain testnet for developers and ecosystem testing. |

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

All network data is structured consistently, allowing you to fetch and utilize information seamlessly.

```sh
ZIGCHAIN_NET_BASE="https://raw.githubusercontent.com/ZIGChain/networks/refs/heads/main"
```

### Selecting a Network

```sh
# Testnet
ZIGCHAIN_NET="$ZIGCHAIN_NET_BASE/zig-test-1"
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


Note:
This document is adapted for ZIGChain from the original work done by Mantra Chain Team on their [net repository](https://github.com/MANTRA-Chain/net).
