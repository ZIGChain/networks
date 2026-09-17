# ZIGChain v4.3 Upgrade

**Coordinated binary upgrade for mainnet.** v4.3.0 applies the upstream patch for the critical CosmWasm vulnerability disclosed 2026-09-03, taking patched `wasmd v0.60.9-rc.2` and `wasmvm v2.3.5-rc.2`. It supersedes [v4.2.0](../v4.2/README.md), which could only harden the binary's memory layout while no patched release existed.

**Consensus-breaking** — the contract VM changes, so every validator must run the same binary from the halt height onward. There is no state migration.

> **Mainnet only.** Testnet (`zig-test-2`) never ran v4.3.0 — it was already on the v5 line. There is nothing to do on testnet.

## Coordinates

| Field | Mainnet (`zigchain-1`) |
|-------|------------------------|
| Binary version | `v4.3.0` |
| Halt height | **12,197,000** |
| Cosmovisor height (`height − 1`) | **12,196,999** |
| Committed at | 2026-09-17 09:05:35 UTC |
| Resumed at | ~2026-09-17 09:08 UTC |
| Mechanism | operator-set `halt-height`, or cosmovisor |
| Status | ✅ complete |

> ⚠️ **The binary does not halt on its own.** There is no coded fork at this height, as with the v4.2 upgrade. Your node only stops if **you** configure `halt-height`, or register the upgrade with cosmovisor at the cosmovisor height.

This upgrade is already done. The steps below are kept as the record of what operators ran, and remain accurate for anyone bringing a node up from a snapshot taken before the halt height.

## Binaries

**Linux only.** The patched wasmvm tags publish no `libwasmvmstatic_darwin.a`, and building one requires compiling osxcross from source, so this release has no darwin build. macOS operators stay on v4.2.0 for local work and run v4.3.0 on their Linux validators.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v4.3.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/zigchaind-v4.3.0-linux-amd64.tar.gz) | `cb1a4e3a2aef700cb3962a328f654680f9cc436d63711f4e3bd07338d845a934` |

Full checksums: [`SHA256SUMS-v4.3.0.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v4.x/SHA256SUMS-v4.3.0.txt)

## Prepare (before the halt)

### 1. Verify what you downloaded

```bash
sha256sum zigchaind-v4.3.0-linux-amd64.tar.gz
# compare against the table above

tar xzf zigchaind-v4.3.0-linux-amd64.tar.gz
./zigchaind version                   # expect: v4.3.0
```

Do **not** replace your running binary yet — keep it staged alongside your current one.

> v4.2.0's `readelf`/PIE and `kernel.randomize_va_space` checks do **not** apply here. PIE was the interim mitigation and is dropped in v4.3.0, which ships the actual patch — this binary is a static non-PIE executable by design. Full ASLR is still worth having on any node host, but nothing in this release depends on it.

### 2. Set your halt height

In `~/.zigchain/config/app.toml`:

```toml
# mainnet (zigchain-1)
halt-height = 12197000
```

Restart your node so the setting takes effect, or pass `--halt-height` on the command line.

**Running cosmovisor?** Register the upgrade at the **cosmovisor height** from the coordinates table above — one block below the halt height — rather than setting `halt-height`. See [docs.zigchain.com](https://docs.zigchain.com) for cosmovisor mechanics.

### 3. Back up

Snapshot your `data/` directory and keep your **current binary** — that is your rollback path.

## At the halt height

Your node commits the halt height, then stops gracefully.

```bash
# 1. stop the service if it is still running
sudo systemctl stop zigchaind

# 2. swap in the new binary
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v4.3.0

# 3. REMOVE the halt height  <-- do not skip this
#    set halt-height = 0 in app.toml, or drop the --halt-height flag.
#    If you leave it set, your node halts again immediately on start.

# 4. restart
sudo systemctl start zigchaind
```

### Verify after restart

```bash
zigchaind version                              # v4.3.0
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

## Rollback

If your node will not start or produce blocks: restore your previous binary, set `halt-height = 0`, and restart.

**Rolling back past the halt height is not safe for a validator.** Unlike v4.2.0, this release changes contract execution, so a v4.2.0 binary will diverge from the patched network on any block containing wasm activity. Roll back only to get a node running for diagnosis, and report the problem so the team can help.

## Verification of the shipped build

```
tag         v4.3.0
commit      74bed28821d9afa50a421855d56e49e0f4641a87
build_tags  muslc,ledger

$ file zigchaind
ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped
$ readelf -h zigchaind | grep Type
  Type: EXEC (Executable file)          # non-PIE by design, see above
$ readelf -lW zigchaind | grep GNU_STACK
  GNU_STACK ... RW          # non-executable
```

Dependency versions in the shipped binary, from `zigchaind version --long`:

```
github.com/CosmWasm/wasmd@v0.60.9-rc.2
github.com/CosmWasm/wasmvm/v2@v2.3.5-rc.2
```
