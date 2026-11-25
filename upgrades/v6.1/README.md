# ZIGChain v6.1 Upgrade

ZIGChain `v6.1.0` through the `v6.1` governance upgrade, from `v6.0.0`. It ships the Cosmos Labs December security patches for `cosmos/evm`, `wasmd`, `wasmvm`, the Cosmos SDK and CometBFT. Details of the security fixes follow once the upstream advisories are published.

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Mechanism | governance software-upgrade | governance software-upgrade |
| Upgrade name | `v6.1` | `v6.1` |
| Binary version | `v6.1.0` | `v6.1.0` |
| Upgrade height | **[8,885,500](https://explorer.nodestake.org/zigchain-testnet/block/8885500)** (Thu 2026-11-26 ~09:30 UTC) | **[14,110,000](https://explorer.nodestake.org/zigchain/block/14110000)** (Fri 2026-11-27 ~09:30 UTC) |
| Proposal | pending | pending |
| Status | scheduled | scheduled |

> Heights and proposal links are filled in as each stage is reached, never guessed ahead of time. The **height is authoritative**; any time given next to it is an estimate from the current block rate and will drift.

## Binaries

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v6.1.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.1.0/zigchaind-v6.1.0-linux-amd64.tar.gz) | `13e4d22bac24319d1bd8a80c23db309788381d0dafae29718df4bb2b0850a5d3` |

Full checksums: [`SHA256SUMS-v6.1.0.txt`](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.1.0/SHA256SUMS-v6.1.0.txt). Built from commit `cb9c23a642f4f5347c5ef569d996cd7e5d8b887b` with go1.26.10. `zigchaind version --long` reports `v6.1.0`. Its dependency list shows `cosmos/evm`, `wasmd`, `wasmvm`, `cosmos-sdk` and `cometbft` replaced by the Cosmos Labs security releases this binary is built with, so seeing replacements there is expected.

This release ships **Linux binaries only**. macOS binaries return once the patched `wasmvm` is publicly tagged. The source is not public yet, so `v6.1.0` cannot be built from source until then.

## app.toml and config.toml

**No changes are needed.** Keep the `app.toml` you run on `v6.0.0`, including `[mempool] max-txs = 5000` and your `[evm] evm-chain-id`. Unlike `v6`, nothing has to be edited between the halt and the restart, so cosmovisor can restart the node on its own (`DAEMON_RESTART_AFTER_UPGRADE=true`).

## What changes for users

These changes are part of the `v6.1` upgrade and need no action from node operators. The full list of EVM changes follows with the security details.

- An EVM transaction may ask for at most **16,777,216 gas**. The EVM mempool rejects a larger one, and `eth_estimateGas` stops at that value. Tooling that sends EVM transactions with a higher gas limit must lower it.
- Some EVM precompiles change their gas cost, so a contract call can cost slightly more or less gas than on `v6.0.0`.

### RPC operator nodes

An EVM transaction rejected while committing its state is no longer part of the block's EVM view, as in upstream `cosmos/evm`. `eth_getTransactionReceipt` and `eth_getTransactionByHash` return `null` for it, and every later EVM transaction in the block moves up one position, with its logs. Blocks produced before the upgrade are served the same way, so indexers that stored `v6.0.x` results for such blocks should re-index them.

## Upgrade through governance

The proposal schedules the `v6.1` plan at a height. This is a coded upgrade, so **do not set `halt-height`**: the running binary stops on its own at the upgrade height.

### 1. Vote

```bash
zigchaind tx gov vote <proposal-id> yes --from <your-key> --chain-id <chain-id> --fees <fee>
```

### 2. Stage the binary

```bash
sha256sum -c --ignore-missing SHA256SUMS-v6.1.0.txt
tar -xzf zigchaind-v6.1.0-linux-amd64.tar.gz
```

- **Cosmovisor:** place it at `$DAEMON_HOME/cosmovisor/upgrades/v6.1/bin/zigchaind`. The directory name must be the plan name `v6.1`, not `v6.1.0`. With `DAEMON_ALLOW_DOWNLOAD_BINARIES=true` cosmovisor fetches it from `plan.info` instead, but staging it yourself removes the dependency on GitHub at halt time.
- **Manual:** keep it staged next to your current binary. Do not replace anything yet.

Snapshot your `data/` directory before the height.

### At the upgrade height

Your node logs `UPGRADE "v6.1" NEEDED at height: <height>` and stops. Cosmovisor swaps the binary and restarts it.

`v6.0.0` can hang after that line on `Shutting down EVM mempool` instead of exiting. Cosmovisor kills the old process itself, after `DAEMON_SHUTDOWN_GRACE` if you set one. On a manual setup, `systemctl stop` waits for the unit's stop timeout; if the process is still there, `kill -9` it. The halt happens before the upgrade block is committed, so this is safe.

Manually, from the extracted `zigchaind-v6.1.0-linux-amd64/` folder:

```bash
sudo systemctl stop zigchaind
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v6.1.0
sudo systemctl start zigchaind
```

The `v6.1` upgrade handler has no state migration, so the first `v6.1.0` block is quick. Stored CosmWasm contracts are recompiled the first time they run after the upgrade, so their first calls are slower than usual.

### Verify after restart

```bash
zigchaind version                                      # v6.1.0
zigchaind version --long | grep '^commit'              # commit: cb9c23a642f4f5347c5ef569d996cd7e5d8b887b
zigchaind query wasm libwasmvm-version                 # 2.3.7-rc.4
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Check the commit, not the CometBFT version: the patched CometBFT still reports itself as `0.38.26`.

Confirm your node is signing and blocks are advancing.

### Rollback

**Not possible by swapping binaries.** There is no state migration, but the upgrade plan is recorded on chain and the security patches change execution, so `v6.0.0` cannot run past the upgrade height and stops again with the same `UPGRADE NEEDED` panic. Recovery before the chain moves on means restoring your snapshot, coordinated with the team. Do not use `--unsafe-skip-upgrades` on your own; that forks your node off the network.
