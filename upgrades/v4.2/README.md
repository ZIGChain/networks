# ZIGChain v4.2 Upgrade

**Coordinated binary upgrade for testnet and mainnet.** v4.2.0 is a security-hardening release: the Linux binary now ships as a static Position-Independent Executable so the OS can load it at a randomized base address. Validators must also have full ASLR enabled — **without ASLR this release provides no benefit.**

> **Superseded.** v4.2.0 was the interim mitigation for the CosmWasm vulnerability disclosed 2026-09-03. [v4.3.0](../v4.3/README.md) ships the actual upstream patch and is what mainnet runs today. This page is kept as the record of the v4.2 upgrade.

This release changes only how the binary is built and linked. There are **no state machine, module, or consensus changes**, and no migration to run.

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Binary version | `v4.2.0` | `v4.2.0` |
| Halt height | **7,611,000** | **11,851,800** |
| Cosmovisor height (`height − 1`) | **7,610,999** | **11,851,799** |
| Estimated time | ~14:30 UTC, 4 Sep 2026 | ~16:30 UTC, 4 Sep 2026 |
| Mechanism | operator-set `halt-height`, or cosmovisor | operator-set `halt-height`, or cosmovisor |

The **height is authoritative** — times are estimates from current block rates (testnet ~5.61 s, mainnet ~3.17 s per block) and will drift.

> ⚠️ **The binary does not halt on its own.** There is no coded fork at these heights, unlike the v3, v4 and v5 upgrades. Your node will only stop if **you** configure `halt-height`, or register the upgrade with cosmovisor at the cosmovisor height.

## Binaries

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v4.2.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/zigchaind-v4.2.0-linux-amd64.tar.gz) | `969d9cd4314c30d7832fc7701e8859d50232758f55fbdb31fb09b212220131e1` |
| `darwin-arm64` | [zigchaind-v4.2.0-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/zigchaind-v4.2.0-darwin-arm64.tar.gz) | `c20a60b871a1989b6f57591cc117a549a3cd76441e7225052a00833fa2a936aa` |
| `darwin-amd64` | [zigchaind-v4.2.0-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/zigchaind-v4.2.0-darwin-amd64.tar.gz) | `f1bf40a662198b0f9e055e8d9fe300962db1f750b25a487565238b9bc97cc238` |

Full checksums: [`SHA256SUMS-v4.2.0.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/SHA256SUMS-v4.2.0.txt)


## Prepare now (before the halt)

### 1. Verify what you downloaded

```bash
sha256sum zigchaind-v4.2.0-linux-amd64.tar.gz
# compare against the table above

tar xzf zigchaind-v4.2.0-linux-amd64.tar.gz
./zigchaind version                   # expect: v4.2.0
readelf -h ./zigchaind | grep Type    # expect: DYN (Position-Independent Executable file)
```

Do **not** replace your running binary yet — keep it staged alongside your current one.

### 2. Confirm full ASLR is enabled

This is the step that makes the release meaningful.

```bash
/usr/sbin/sysctl kernel.randomize_va_space
```

| Output | Meaning |
|--------|---------|
| `kernel.randomize_va_space = 0` | ASLR disabled |
| `kernel.randomize_va_space = 1` | Partial ASLR |
| `kernel.randomize_va_space = 2` | **Full ASLR — required** |

If it is not `2`, set it:

```bash
sudo sysctl -w kernel.randomize_va_space=2
```

**Make it survive a reboot** — `sysctl -w` is temporary and is lost on restart:

```bash
echo 'kernel.randomize_va_space = 2' | sudo tee /etc/sysctl.d/99-zigchain-aslr.conf
sudo sysctl --system
/usr/sbin/sysctl kernel.randomize_va_space    # confirm: 2
```

### 3. Set your halt height

In `~/.zigchain/config/app.toml`:

```toml
# testnet (zig-test-2)
halt-height = 7611000

# mainnet (zigchain-1)
halt-height = 11851800
```

Restart your node so the setting takes effect, or pass `--halt-height` on the command line.

**Running cosmovisor?** Register the upgrade at the **cosmovisor height** from the coordinates table above — one block below the halt height — rather than setting `halt-height`. See [docs.zigchain.com](https://docs.zigchain.com) for cosmovisor mechanics.

### 4. Back up

Snapshot your `data/` directory and keep your **current binary** — that is your rollback path.

## At the halt height

Your node commits the halt height, then stops gracefully.

```bash
# 1. stop the service if it is still running
sudo systemctl stop zigchaind

# 2. swap in the new binary
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v4.2.0

# 3. REMOVE the halt height  <-- do not skip this
#    set halt-height = 0 in app.toml, or drop the --halt-height flag.
#    If you leave it set, your node halts again immediately on start.

# 4. restart
sudo systemctl start zigchaind
```

### Verify after restart

```bash
zigchaind version                              # v4.2.0
readelf -h $(which zigchaind) | grep Type      # DYN
/usr/sbin/sysctl kernel.randomize_va_space     # 2
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

## Rollback

If your node will not start or produce blocks: restore your previous binary, set `halt-height = 0`, and restart. Because v4.2.0 contains no state changes, the older binary reads the same state without issue. Report the problem so the team can help.

## Why ASLR matters here

v4.2.0 ships the Linux binary as a static Position-Independent Executable, so it can be loaded at a randomized base address. That randomization is performed by the kernel — with `kernel.randomize_va_space` at `0` or `1`, the binary loads predictably and the hardening is lost. **A PIE binary on a host without full ASLR gains nothing.** Verify the value on every machine you operate, and confirm it persists across reboots.

## Verification of the shipped build

```
tag         v4.2.0
commit      5ae196e987fc954c3976634f1ff2b6ac3948b47a
build_tags  muslc,ledger

$ file zigchaind
ELF 64-bit LSB pie executable, x86-64, static-pie linked, stripped
$ readelf -h zigchaind | grep Type
  Type: DYN (Position-Independent Executable file)
$ readelf -lW zigchaind | grep GNU_STACK
  GNU_STACK ... RW          # non-executable
```
