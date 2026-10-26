# ZIGChain v6 Upgrade

ZIGChain `v6.0.0` (Whirlpool) through the `v6` governance upgrade. What changes is in the [release notes](https://github.com/ZIGChain/networks/releases/tag/v6.0.0).

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Mechanism | governance software-upgrade | governance software-upgrade |
| Upgrade name | `v6` | `v6` |
| Binary version | `v6.0.0` | `v6.0.0` |
| Upgrade height | **8,423,210** (~Tue 2026-10-27 09:30 UTC) | not scheduled yet |
| Proposal | not submitted yet | not submitted yet |
| Status | height agreed, proposal not submitted yet | binaries published, not scheduled |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time. The **height is authoritative**; any time given next to it is an estimate from the current block rate and will drift.

## Binaries

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v6.0.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-linux-amd64.tar.gz) | `4373e53d7a05ff0f50037a1cd83801d5f0d972207e7ff50f3cc5d8ded62867a4` |
| `darwin-amd64` | [zigchaind-v6.0.0-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-amd64.tar.gz) | `e37dc2a8b12a2d2d226bb766651b0342bc71d6cccd424bb13b10f22240951b8f` |
| `darwin-arm64` | [zigchaind-v6.0.0-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-arm64.tar.gz) | `7233dd163f6f5e04aaecc3b63091b96babb03d265ea432fa140579ccdf6842d1` |

Full checksums: [`SHA256SUMS-v6.0.0.txt`](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/SHA256SUMS-v6.0.0.txt). Built from commit `81af3b281338487d1c6e17993754f45f9fd974db`, source tree `9a84bbe48256b62d365ccc5776ef8387993abe91`, with go1.26.9. `zigchaind version --long` reports `v6.0.0`.

## app.toml

**No change is needed to upgrade.** `v6.0.0` starts on a v5 `app.toml` as it is: the EVM chain ID is derived from the Cosmos chain ID (`2061` on `zig-test-2`, `944` on `zigchain-1`), so a missing `[evm]` section is fine. What behaves differently:

- `minimum-gas-prices`: keep it set, the node still refuses to start when it is empty. It no longer decides which transactions you accept, the on-chain base fee from `x/feemarket` does.
- `query-gas-limit`: `0` now means a cap of 50,000,000 gas per query instead of unbounded.
- JSON-RPC: off by default, and only needed on nodes that serve Ethereum tooling. A v5 `app.toml` has no `[json-rpc]` section, so generate the v6 template with `zigchaind init tmp --home /tmp/zig-v6` and copy its `[evm]`, `[evm.mempool]`, `[json-rpc]` and `[tls]` sections over. Then set `[json-rpc] enable = true` and keep `[grpc] enable = true`, because `eth_call`, `eth_estimateGas` and `debug_trace*` are served over gRPC.

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

Your node logs `UPGRADE "v6" NEEDED at height: <height>` and stops. Cosmovisor swaps and restarts by itself. Manually, from the extracted `zigchaind-v6.0.0-linux-amd64/` folder:

```bash
sudo systemctl stop zigchaind
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v6.0.0
sudo systemctl start zigchaind
```

The first `v6.0.0` block runs the `v6` upgrade handler, so it can take longer than a normal block. Let it finish; do not restart mid-migration.

### Verify after restart

```bash
zigchaind version                                      # v6.0.0
zigchaind query wasm libwasmvm-version                 # 2.3.5
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

### Rollback

**Not possible by swapping binaries.** The upgrade handler migrates state, so the previous binary cannot run past the upgrade height and stops again with the same `UPGRADE NEEDED` panic. Recovery before the chain moves on means restoring your snapshot, coordinated with the team. Do not use `--unsafe-skip-upgrades` on your own; that forks your node off the network.
