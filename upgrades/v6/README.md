# ZIGChain v6 Upgrade

ZIGChain `v6.0.0` (Whirlpool) through the `v6` governance upgrade. What changes is in the [release notes](https://github.com/ZIGChain/networks/releases/tag/v6.0.0).

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Mechanism | governance software-upgrade | governance software-upgrade |
| Upgrade name | `v6` | `v6` |
| Binary version | `v6.0.0` | `v6.0.0` |
| Upgrade height | **[8,423,210](https://explorer.nodestake.org/zigchain-testnet/block/8423210)** (~Tue 2026-10-27 09:30 UTC) | not scheduled yet |
| Proposal | [#122](https://explorer.nodestake.org/zigchain-testnet/gov/122), passed Mon 2026-10-26 21:18 UTC | not submitted yet |
| Status | upgrade scheduled at 8,423,210 | binaries published, not scheduled |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time. The **height is authoritative**; any time given next to it is an estimate from the current block rate and will drift.

## Binaries

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v6.0.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-linux-amd64.tar.gz) | `4373e53d7a05ff0f50037a1cd83801d5f0d972207e7ff50f3cc5d8ded62867a4` |
| `darwin-amd64` | [zigchaind-v6.0.0-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-amd64.tar.gz) | `e37dc2a8b12a2d2d226bb766651b0342bc71d6cccd424bb13b10f22240951b8f` |
| `darwin-arm64` | [zigchaind-v6.0.0-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-arm64.tar.gz) | `7233dd163f6f5e04aaecc3b63091b96babb03d265ea432fa140579ccdf6842d1` |

Full checksums: [`SHA256SUMS-v6.0.0.txt`](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/SHA256SUMS-v6.0.0.txt). Built from commit `81af3b281338487d1c6e17993754f45f9fd974db`, source tree `9a84bbe48256b62d365ccc5776ef8387993abe91`, with go1.26.9. `zigchaind version --long` reports `v6.0.0`. Its dependency list shows `github.com/cosmos/evm v0.6.3` replaced by `v0.6.4-rc4`: the replacement is the cosmos/evm security release this binary is built with, so seeing `v0.6.3` there is expected.

## app.toml

**Every node needs `app.toml` changes for v6.** `config.toml` needs none.

### Step 1: every node, every network

1. Append the sections below, unless your `app.toml` already has them. You can stage this before the upgrade, and it must be in place before the node starts on v6.
2. Change `[mempool] max-txs` from `-1` to `5000`. Do this **after** the node halts at the upgrade height and **before** it starts on v6. With `-1` the EVM mempool stays off, and a node with JSON-RPC enabled panics on start.

```toml
[evm]
tracer = ""
max-tx-gas-wanted = 0
cache-preimage = false
evm-chain-id = 0
min-tip = 0
geth-metrics-address = "127.0.0.1:8100"

[evm.mempool]
price-limit = 1
price-bump = 10
account-slots = 16
global-slots = 5120
account-queue = 64
global-queue = 1024
lifetime = "3h0m0s"

[json-rpc]
enable = false
address = "127.0.0.1:8545"
ws-address = "127.0.0.1:8546"
ws-origins = ["127.0.0.1", "localhost"]
api = "eth,net,web3"
allow-insecure-unlock = false
evm-timeout = "5s"
txfee-cap = 1
filter-cap = 200
feehistory-cap = 100
logs-cap = 10000
block-range-cap = 10000
http-timeout = "30s"
http-idle-timeout = "2m0s"
allow-unprotected-txs = false
max-open-connections = 0
enable-indexer = false
metrics-address = "127.0.0.1:6065"
batch-request-limit = 1000
batch-response-max-size = 25000000
enable-profiling = false

[tls]
certificate-path = ""
key-path = ""
```

Cosmovisor restarts the node on v6 right after the halt, which leaves no window for the `max-txs` change. Set `DAEMON_RESTART_AFTER_UPGRADE=false` before the height, then after the halt edit `max-txs` and start the service again. Check that your service manager does not restart it on its own in between. Setting `max-txs` earlier is not equivalent: on v5 any value from `0` up replaces the no-op mempool with an app-side one.

### Step 2: the EVM chain ID

Set `[evm] evm-chain-id` to match the network:

- DevNet: `91902`
- Testnet (`zig-test-2`): `2061`
- Mainnet (`zigchain-1`): `944`

`0` also works and resolves to the registered ID. Any other value stops the node at startup. Do not copy `[evm]` from a `zigchaind init` run without `--chain-id`: it renders `evm-chain-id = 91999`, which the node rejects.

### RPC operator nodes only

Set `[json-rpc] enable = true` to serve EVM JSON-RPC (ports `8545` / `8546`). Keep `[grpc] enable = true`, because `eth_call`, `eth_estimateGas` and `debug_trace*` are served over gRPC. This needs the `max-txs` change from step 1.

### Also different on v6

- `minimum-gas-prices`: keep it set, the node still refuses to start when it is empty. It no longer decides which transactions you accept, the on-chain base fee from `x/feemarket` does, with a floor of `2500000000azig` set by the upgrade.
- `query-gas-limit`: `0` now means a cap of 50,000,000 gas per query instead of unbounded.
- Hardware requirements rise modestly.

## Keys

- `zigchaind keys add` now defaults to coin type 60 / `eth_secp256k1`, so a MetaMask or Foundry seed derives the same key and address. This is a client-side default, existing accounts and validator keys are unaffected.
- To recover a pre-v6 (coin type 118) key, use `keys add <name> --recover --legacy`, or the explicit pair `--recover --coin-type 118 --key-type secp256k1`. Without it you get a different, empty address.
- Ledger through the CLI is legacy only for now: `keys add <name> --ledger --legacy`. Coin-type-60 Ledger keys through the CLI are not possible yet, MetaMask + Ledger works for EVM.

## Upgrade through governance

The proposal schedules the `v6` plan at a height. This is a coded upgrade, so **do not set `halt-height`**: the running binary stops on its own at the upgrade height.

### 1. Vote

```bash
zigchaind tx gov vote <proposal-id> yes --from <your-key> --chain-id <chain-id> --fees <fee>
```

### 2. Stage the binary

```bash
sha256sum -c --ignore-missing SHA256SUMS-v6.0.0.txt   # macOS: shasum -a 256 -c --ignore-missing
tar -xzf zigchaind-v6.0.0-linux-amd64.tar.gz
```

- **Cosmovisor:** place it at `$DAEMON_HOME/cosmovisor/upgrades/v6/bin/zigchaind`. The directory name must be the plan name `v6`, not `v6.0.0`. With `DAEMON_ALLOW_DOWNLOAD_BINARIES=true` cosmovisor fetches it from `plan.info` instead, but staging it yourself removes the dependency on GitHub at halt time.
- **Manual:** keep it staged next to your current binary. Do not replace anything yet.

Snapshot your `data/` directory before the height.

### At the upgrade height

Your node logs `UPGRADE "v6" NEEDED at height: <height>` and stops. Cosmovisor swaps the binary, and with `DAEMON_RESTART_AFTER_UPGRADE=false` waits for you to set `max-txs` (app.toml step 1) and start it. Manually, from the extracted `zigchaind-v6.0.0-linux-amd64/` folder:

```bash
sudo systemctl stop zigchaind
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v6.0.0
# set [mempool] max-txs = 5000 in app.toml (app.toml step 1)
sudo systemctl start zigchaind
```

The first `v6.0.0` block runs the `v6` upgrade handler, so it can take longer than a normal block. Let it finish; do not restart mid-migration.

### Verify after restart

```bash
zigchaind version                                      # v6.0.0
zigchaind query wasm libwasmvm-version                 # 2.3.5
zigchaind version --long | grep cosmos/evm             # v0.6.3 => ... v0.6.4-rc4
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

### Rollback

**Not possible by swapping binaries.** The upgrade handler migrates state, so the previous binary cannot run past the upgrade height and stops again with the same `UPGRADE NEEDED` panic. Recovery before the chain moves on means restoring your snapshot, coordinated with the team. Do not use `--unsafe-skip-upgrades` on your own; that forks your node off the network.
