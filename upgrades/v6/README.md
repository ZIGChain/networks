# ZIGChain v6 Upgrade

ZIGChain `v6.0.0` (Whirlpool) through the `v6` governance upgrade. What changes is in the [release notes](https://github.com/ZIGChain/zigchain/releases/tag/v6.0.0).

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Mechanism | governance software-upgrade | governance software-upgrade |
| Upgrade name | `v6` | `v6` |
| Binary version | `v6.0.0` | `v6.0.0` |
| Upgrade height | not scheduled yet | not scheduled yet |
| Proposal | not submitted yet | not submitted yet |
| Status | binaries published, not scheduled | binaries published, not scheduled |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time. The **height is authoritative**; any time given next to it is an estimate from the current block rate and will drift.

## Binaries

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v6.0.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-linux-amd64.tar.gz) | `edd6cf1c23fbcd696f5187aa0b18b2345f84d477ae1337c170e257aabddc701b` |
| `darwin-amd64` | [zigchaind-v6.0.0-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-amd64.tar.gz) | `0f226129f78c9b45c2703fc17dd49e246e18d9d4ac698e320ea6a729537aae32` |
| `darwin-arm64` | [zigchaind-v6.0.0-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/zigchaind-v6.0.0-darwin-arm64.tar.gz) | `165fb90dee7d250bf56d1897edd64d8b083d114b2a9281360d9a8d97e99e3023` |

Full checksums: [`SHA256SUMS-v6.0.0.txt`](https://github.com/ZIGChain/networks/raw/refs/heads/main/binaries/v6.0.0/SHA256SUMS-v6.0.0.txt). Built from commit `79b3049fd003eaa27cf3616e66c23cf8c9a6b5a6`, source tree `369b6f697c846d919e15fe21c22485a7b65bd166`, with go1.26.9. `zigchaind version --long` reports `v6.0.0`.

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
