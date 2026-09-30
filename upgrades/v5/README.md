# ZIGChain v5 Upgrade

**`uzig` → `azig` redenomination.** ZIGChain's base denomination changes from `uzig` (6 decimals) to `azig` (18 decimals). **1 ZIG is still 1 ZIG** — balances, total supply, and value are unchanged; only the on-chain representation gains precision (1 ZIG = 10¹⁸ `azig`, previously 10⁶ `uzig`). `uzig` remains only as IBC escrow backing.

## Who's affected

| Audience | What changes |
|----------|--------------|
| **Exchanges / integrators** | Base denom is now `azig` with **18 decimals** (was `uzig`, 6). Update deposit/withdrawal accounting and display math accordingly. Nominal balances are unchanged — 1 ZIG stays 1 ZIG. |
| **Builders / dApp devs** | Any code that hard-codes `uzig` or 6-decimal math must move to `azig` / 18 decimals. Test against real state first — see the [local test guide](local-test-guide.md). |
| **Validators / node operators** | Two different mechanisms: testnet is a coordinated `halt-height` binary swap, mainnet is a governance software-upgrade. See the coordinates below. |

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Mechanism | operator-set `halt-height` | governance software-upgrade |
| Upgrade name | none | `v5` |
| Binary version | `v5.1.0` | `v5.1.0`, recovery on `v5.1.2` |
| Halt / upgrade height | **7,930,000** (Fri 2026-09-25 08:57 UTC) | **12,549,000** (~Wed 2026-09-30 09:00 UTC) |
| Cosmovisor height (`height − 1`) | **7,929,999** | not applicable, cosmovisor switches on the `v5` plan |
| Proposal | not applicable | [#41](https://explorer.nodestake.org/zigchain/gov/41), voting ends Tue 2026-09-29 11:00 UTC |
| Status | ✅ done, running `v5.1.0` | ⛔ halted at 12,549,000, recover with [`v5.1.2`](#mainnet--recovery-from-the-halt-at-12549000-v512) |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time. The **height is authoritative**; the mainnet time is an estimate from the current block rate (~3.17 s per block) and will drift. Countdown: [block 12,549,000](https://explorer.nodestake.org/zigchain/block/12549000).

**Why the two differ.** The `v5` redenomination already ran on `zig-test-2` at height `7,669,200`, so testnet needs only the patched CosmWasm runtime. That swap changes no state and is not consensus-breaking, so testnet takes it as a coordinated `halt-height` restart: no plan name, no proposal, and no upgrade handler is involved. Mainnet has not run `v5` yet and takes the redenomination and the patched runtime together, in one governance upgrade at the `v5` height.

## Binaries

### Latest — `v5.1.2`

**Required on MainNet to recover from the halt at 12,549,000.** Same v5 state machine as `v5.1.0` / `v5.1.1`, plus the fix for the oversized upgrade block and a built-in `state.db` repair. Follow the [recovery steps](#mainnet--recovery-from-the-halt-at-12549000-v512). TestNet does not need it: it already ran `v5`, and the repair is a no-op on a healthy node.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.1.2-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/zigchaind-v5.1.2-linux-amd64.tar.gz) | `409ea0cc283f6fa90fcda32ae346a539e8eef99bd67ae8fa5176190de46f9c52` |
| `darwin-amd64` | [zigchaind-v5.1.2-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/zigchaind-v5.1.2-darwin-amd64.tar.gz) | `e4c6aad1a6ec5f90285dc107c43febc7874ca8321bf8b7c590eb712d4abe7a68` |
| `darwin-arm64` | [zigchaind-v5.1.2-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/zigchaind-v5.1.2-darwin-arm64.tar.gz) | `34a28984a081bc990b6a1db1ece8bf06c07d9bf55afe51867148e0bdfeadf297` |

Full checksums: [`SHA256SUMS-v5.1.2.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/SHA256SUMS-v5.1.2.txt). Built from commit `2de3c0f4d637efd187c373f17e30ac534ca308a0`. `zigchaind version --long` reports `v5.1.2`.

### Previous — `v5.1.1`

Same code as `v5.1.0`, built against the public CosmWasm tags (`wasmd v0.60.9`, `wasmvm v2.3.5`) now that the embargo has ended. Upstream confirms they carry the same fix as the `-rc.3` tags in `v5.1.0`. Only the compiled library changes, so `v5.1.0` and `v5.1.1` nodes interoperate: swap one node at a time, with no halt height, no proposal and no coordination. It is not urgent. darwin builds are back.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.1.1-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.1/zigchaind-v5.1.1-linux-amd64.tar.gz) | `03d46cf891c2d466bfa91e228521914f2ddf9bb043ab74f3702803a50498e4ea` |
| `darwin-amd64` | [zigchaind-v5.1.1-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.1/zigchaind-v5.1.1-darwin-amd64.tar.gz) | `02f2a4e5ca5e39c28a0ea166f5e66de8ae37f7f9289192fd95c56586039bba39` |
| `darwin-arm64` | [zigchaind-v5.1.1-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.1/zigchaind-v5.1.1-darwin-arm64.tar.gz) | `1067267ffb866c95d0492bb225559f8fa3bb962d616030b8e86031abbf0135e1` |

Full checksums: [`SHA256SUMS-v5.1.1.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.1/SHA256SUMS-v5.1.1.txt). Built from the `v5.1.1` tag on GitHub Public (`ZIGChain/zigchain` commit `4b75028fdd2fdb79e6126ea83bb8b1ef4867351e`). `zigchaind version --long` reports `v5.1.1`, `wasmd v0.60.9` and `wasmvm v2.3.5`.

### Release — `v5.1.0`

Use these **`v5.1.0`** builds for the governance software-upgrade. They supersede `v5.0.0-patch-1`: same v5 state machine, plus the patched CosmWasm runtime (`wasmd v0.60.9-rc.3`, `wasmvm v2.3.5-rc.3`). Mainnet runs them at the `v5` governance height; testnet, already redenominated, takes them as a `halt-height` swap.

**Linux only.** The patched wasmvm tags publish no `libwasmvmstatic_darwin.a`, and building one requires compiling osxcross from source, so this release has no darwin build. macOS operators stay on `v5.0.0-patch-1` for local work and run `v5.1.0` on their Linux validators.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.1.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.0/zigchaind-v5.1.0-linux-amd64.tar.gz) | `93f2be769ebafb369ed6fee03a0159ee6699e5aaae27d5ca165bc2b88b42d914` |

Full checksums: [`SHA256SUMS-v5.1.0.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.0/SHA256SUMS-v5.1.0.txt). Built from `release/v5` at commit `3dd8a7ea24a62e92d16b6f094a34f27637e00cc5`; `zigchaind version --long` reports `v5.1.0` and `zigchaind query wasm libwasmvm-version` reports `2.3.5-rc.3`.

Proposal #41's `plan.info` carries this same URL and SHA-256, which is what cosmovisor uses for auto-download.

### Superseded — `v5.0.0-patch-1`

TestNet ran the v5 upgrade on these builds. Kept for the record, and as the last release with darwin artifacts.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-patch-1-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-linux-amd64.tar.gz) | `002c1edb1db0f32ac16bc3faaa96438b1720f9330b2fca438108f19cd49586ee` |
| `darwin-arm64` | [zigchaind-v5.0.0-patch-1-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-darwin-arm64.tar.gz) | `408972867f66ae17fcf6730c5e5c96432f2175a96a36d991b6e0f26d7fa567ef` |
| `darwin-amd64` | [zigchaind-v5.0.0-patch-1-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-darwin-amd64.tar.gz) | `3b9dfc2cfd290fe2cf8e7a2f6ba6cff5f93f9a2f1fb1c2565ca27b17803e9b28` |

Full checksums: [`SHA256SUMS-v5.0.0-patch-1.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/SHA256SUMS-v5.0.0-patch-1.txt).

### QA build — `v5.0.0-rc.1-qa-m3off` (local testing only)

A **stale release-candidate** build, kept solely for testing the upgrade against **mainnet data** on a local fork — see the [local test guide](local-test-guide.md). It is based on `v5.0.0-rc.1` (not the final `v5.0.0` release) and disables the `in-place-testnet`-incompatible check so the upgrade can complete offline. **Do not use on production, testnet, or mainnet.**

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz) | `a9b029461ae5a456f6b40acd41ce5c90dbabe8aec2b5039ad8bb477aaefb4b5a` |
| `darwin-arm64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz) | `0d7cb074fbd47672de410f77f642b32056ff9f0118f615b109135789bc165bba` |
| `darwin-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz) | `81fc6b83210c6c4e07880b6522259fd7527aa4de9635c4e97d0185139f73099e` |

Full checksums: [`SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt).

## Mainnet — recovery from the halt at 12,549,000 (`v5.1.2`)

`zigchain-1` halted at the `v5` upgrade height, **12,549,000**, and nodes on goleveldb (the default backend) could not restart. They panic opening CometBFT's `state.db`:

```
panic: snappy: decoded block is too large
```

The `v5` handler emitted bank events for every balance it migrated, which made the upgrade block's FinalizeBlock response 3,788,334,043 bytes. goleveldb cannot store a value that size, so every start, and `zigchaind rollback`, panics the same way.

`v5.1.2` fixes both sides:

- **The handler** keeps only its own `v5_*` events, so a node that replays the upgrade block stores a response of a few hundred bytes.
- **The node** repairs `state.db` before `start`: it strips the oversized events from the stored response, completes an interrupted write if the node crashed between CometBFT's two writes, and compacts the old versions away. The same repair is available as `zigchaind repair-state-db`, which prints what it checked. It is a no-op on a healthy database and on non-goleveldb backends.

`v5.1.2` runs the same v5 state machine as `v5.1.0` / `v5.1.1`. It is **not** a new upgrade: no proposal, no new height, and the upgrade name stays `v5`. Block-level events are in neither the app hash nor the header's `LastResultsHash`, so dropping them changes no state and no header field. The only visible difference is that `/block_results?height=12549000` lists only the `v5_*` events.

The chain resumes once validators with more than two-thirds of the voting power are back. Everyone on `zigchain-1` needs this.

### Before you start

- **Free RAM: about 18 GB.** The repair holds the 3.79 GB value several times while goleveldb replays its journal. Measured peak: 17.9 GiB. Add temporary swap if the machine has less.
- **Free disk: about 10 GB**, plus room for the backups below.
- **Stop the node** so systemd or cosmovisor does not restart it in a loop.
- **Do not** start `v5.1.0` or `v5.1.1` again.
- **Do not** use `--unsafe-skip-upgrades`.
- **Do not** delete your `data` folder.
- **Do not** use the `zigchaind rollback` feature.

The steps assume the home is `~/.zigchain` and the service is `zigchaind`. Adjust if yours differ.

### 1. Stop and back up

```bash
sudo systemctl stop zigchaind
systemctl is-active zigchaind                 # must be "inactive"

cp -a ~/.zigchain/data/state.db ~/state.db.bak-12549000
cp -a ~/.zigchain/data/priv_validator_state.json ~/priv_validator_state.json.bak
free -g; df -h ~/.zigchain
```

### 2. Download and verify `v5.1.2`

The files are also listed in the [`v5.1.2` table above](#latest--v512).

```bash
mkdir -p ~/v5.1.2 && cd ~/v5.1.2
curl -LO https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/zigchaind-v5.1.2-linux-amd64.tar.gz
curl -LO https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.2/SHA256SUMS-v5.1.2.txt
sha256sum -c SHA256SUMS-v5.1.2.txt --ignore-missing   # zigchaind-v5.1.2-linux-amd64.tar.gz: OK

tar xzf zigchaind-v5.1.2-linux-amd64.tar.gz   # extracts into zigchaind-v5.1.2-linux-amd64/
cd zigchaind-v5.1.2-linux-amd64
./zigchaind version                           # v5.1.2
```

Steps 3 and 4 run `./zigchaind` from this folder. In a new shell, `cd ~/v5.1.2/zigchaind-v5.1.2-linux-amd64` first.

### 3. Run the repair and read its output

Run it as the user that runs the node, not with `sudo`. The repair writes new files into `state.db`, and files owned by another user can stop the node from opening it.

```bash
./zigchaind repair-state-db --home ~/.zigchain
# node runs as another user:  sudo -u <node-user> ./zigchaind repair-state-db --home <node-home>
```

It takes a minute or two on an affected node. Every line starts with `state.db repair:`. Find your case:

| Output contains | Meaning | Next |
|---|---|---|
| `REPAIRED 3788334043 -> … bytes` and `done, N key(s) repaired` | The oversized response was stripped. | Step 4 |
| `lastABCIResponseKey COMPLETED interrupted write: now height 12549000, app_hash <HASH>` | Your node crashed between CometBFT's two writes. | **Check the hash below**, then step 4 |
| Only `ok, … bytes` lines and `done, 0 key(s) repaired` | Your `state.db` was not affected, for example a node restored from a pre-upgrade snapshot. | Step 4 |
| `skipped, db_backend is not goleveldb` | Not applicable to pebbledb / rocksdb. | Step 4. If it still does not start, report it. |
| a panic, an error, or `Killed` (check `dmesg \| tail`) | The repair did not complete. `Killed` usually means not enough RAM. | **Stop and report it** with the output. |

**Hash check, only if you saw `COMPLETED interrupted write`:** `<HASH>` must equal the app hash of the upgrade block:

```
7EC4D38C039DCF9D47E8BFFB79845A770BC6481F377E80D7BABA5DF0EEF7A3BB
```

It also appears in your own log as `ABCI Handshake App Info hash=… height=12549000` from an earlier start attempt. **If it differs, do not start. Report it.**

### 4. Install

**Cosmovisor:** the binary replaces the one in the `v5` upgrade directory. The directory name is the plan name, `v5`.

```bash
cp ~/.zigchain/cosmovisor/upgrades/v5/bin/zigchaind ~/zigchaind-v5.1.1.bak
install -m 0755 ./zigchaind ~/.zigchain/cosmovisor/upgrades/v5/bin/zigchaind
readlink -f ~/.zigchain/cosmovisor/current    # …/cosmovisor/upgrades/v5
~/.zigchain/cosmovisor/current/bin/zigchaind version    # v5.1.2
```

**Without cosmovisor:** replace the binary your service runs. `systemctl cat zigchaind | grep ExecStart` shows its path if it is not the one on your `PATH`.

```bash
BIN=$(which zigchaind)                        # or the path from ExecStart
sudo install -m 0755 ./zigchaind "$BIN"
"$BIN" version                                # v5.1.2
```

### 5. Start and watch

```bash
sudo systemctl start zigchaind
journalctl -u zigchaind -f -o cat
```

The automatic repair runs first and finds nothing left to do. Then you see one of these:

- **`Replay last block using mock app`** → `Completed ABCI Handshake`. Your app had already committed the upgrade block. This takes seconds.
- **`Replay last block using real app`** → `applying upgrade "v5"` → about 5 to 10 minutes of silence → `Upgrade v5 complete` → `Completed ABCI Handshake`. Your app had not committed it, so the upgrade runs again with the fixed handler. **Do not restart during the silence.** The `v5: stubbed gov proposal … proposal_id=3` warning is expected.
- **`Completed ABCI Handshake`** directly. Your node was already in sync.

After the handshake, the node waits at 12,549,001 until more than two-thirds of the voting power is up. Prevote timeouts and missing peers are normal until then.

### 6. Verify

```bash
curl -s localhost:26657/status | jq '.result.sync_info | {latest_block_height, latest_app_hash, catching_up}'
# before the chain resumes: height 12549000, app hash 7EC4D38C…

# once blocks advance past 12549000 (cosmovisor: use ~/.zigchain/cosmovisor/current/bin/zigchaind):
zigchaind version                                                    # v5.1.2
zigchaind query staking params -o json | jq -r .params.bond_denom    # azig
zigchaind query wasm libwasmvm-version                               # 2.3.5
```

Confirm your validator signs: its address shows `BLOCK_ID_FLAG_COMMIT` in the next blocks' `last_commit`.

### If the handshake fails with `expected height 12549000 but last stored abci responses was at height 12548999`

The repair found no stored response to complete the interrupted write with. Now that `state.db` opens, roll the app back one height and let the upgrade re-run:

```bash
sudo systemctl stop zigchaind
zigchaind rollback --home ~/.zigchain         # cosmovisor: ~/.zigchain/cosmovisor/current/bin/zigchaind
sudo systemctl start zigchaind                # expect "Replay last block using real app"
```

Report it as well, so the team knows this case happened.

### Undoing the procedure

Stop the node, then restore `~/state.db.bak-12549000` over `data/state.db`. **Keep your current `priv_validator_state.json`.** Report what you saw. Do not start `v5.1.1` on the restored data.

## Mainnet — governance upgrade to `v5`

Proposal [#41](https://explorer.nodestake.org/zigchain/gov/41) schedules the `v5` plan at height **12,549,000**. This one is a coded upgrade, so the flow is the reverse of testnet: **do not set `halt-height`**. The running `v5.0.0` binary stops on its own at the upgrade height.

### 1. Vote

Validators vote before Tue 2026-09-29 11:00 UTC:

```bash
zigchaind tx gov vote 41 yes --from <your-key> --chain-id zigchain-1 --fees <fee>
```

### 2. Stage the binary

Verify it as in [step 1 of the testnet section](#1-verify-what-you-downloaded) (same tarball, same SHA-256), then:

- **Cosmovisor:** place it at `$DAEMON_HOME/cosmovisor/upgrades/v5/bin/zigchaind`. The directory name must be the plan name `v5`, not `v5.1.0`. With `DAEMON_ALLOW_DOWNLOAD_BINARIES=true` cosmovisor fetches it from `plan.info` instead, but staging it yourself removes the dependency on GitHub at halt time.
- **Manual:** keep it staged next to your current binary. Do not replace anything yet.

Snapshot your `data/` directory before the height.

### At the upgrade height

Your node logs `UPGRADE "v5" NEEDED at height: 12549000` and stops. Cosmovisor swaps and restarts by itself. Manually, from the extracted `zigchaind-v5.1.0-linux-amd64/` folder:

```bash
sudo systemctl stop zigchaind
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v5.1.0
sudo systemctl start zigchaind
```

The first `v5.1.0` block runs the redenomination migration, so it can take noticeably longer than a normal block. Let it finish; do not restart mid-migration.

### Verify after restart

```bash
zigchaind version                                      # v5.1.0
zigchaind query wasm libwasmvm-version                 # 2.3.5-rc.3
zigchaind query staking params -o json | jq -r .params.bond_denom   # azig
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

### Rollback

**Not possible by swapping binaries.** The upgrade migrates state (`uzig` → `azig`), so `v5.0.0` cannot run past height 12,549,000 and stops again with the same `UPGRADE NEEDED` panic. Recovery before the chain moves on means restoring your snapshot, coordinated with the team. Do not use `--unsafe-skip-upgrades` on your own; that forks your node off the network.

## Testnet — halt-height swap to `v5.1.0` (done 2026-09-25)

No governance proposal is submitted for this one. Every operator stops at an agreed height, swaps the binary and restarts.

> ⚠️ **The binary does not halt on its own.** There is no coded fork and no upgrade plan at this height. Your node only stops if **you** configure `halt-height`, or register the swap with cosmovisor at the cosmovisor height.

### 1. Verify what you downloaded

```bash
sha256sum zigchaind-v5.1.0-linux-amd64.tar.gz
# compare against the table above

tar xzf zigchaind-v5.1.0-linux-amd64.tar.gz   # extracts into zigchaind-v5.1.0-linux-amd64/
cd zigchaind-v5.1.0-linux-amd64
./zigchaind version                   # expect: v5.1.0
```

Keep it staged alongside your current binary. Do not replace anything yet. The `./zigchaind` commands below run from this folder.

### 2. Set your halt height

In `~/.zigchain/config/app.toml`:

```toml
# testnet (zig-test-2)
halt-height = 7930000
```

Restart your node so the setting takes effect, or pass `--halt-height` on the command line.

### 3. Back up

Snapshot your `data/` directory and keep your **current binary**. That is your rollback path.

### At the halt height

Your node commits the block *before* the halt height, then stops. The new binary produces the halt height itself.

> **The stop looks like a crash, and that is expected.** CometBFT logs `CONSENSUS FAILURE!!!` with a panic stack, wrapping `halt per configuration height <halt height>`. That is how a configured halt is implemented; it is not a fault and no state is lost. Verified against a `zig-test-2` snapshot.

```bash
# 1. stop the service if it is still running
sudo systemctl stop zigchaind

# 2. swap in the new binary
sudo install -m 0755 ./zigchaind $(which zigchaind)
zigchaind version                     # expect: v5.1.0

# 3. REMOVE the halt height  <-- do not skip this
#    set halt-height = 0 in app.toml, or drop the --halt-height flag.
#    If you leave it set, your node halts again immediately on start.

# 4. restart
sudo systemctl start zigchaind
```

### Verify after restart

```bash
zigchaind version                                      # v5.1.0
zigchaind query wasm libwasmvm-version                 # 2.3.5-rc.3
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'
```

Confirm your node is signing and blocks are advancing.

### Rollback

Restore your previous binary, set `halt-height = 0`, and restart.

Rolling back is **safe** here, unlike v4.3.0. Only the compiled CosmWasm library changes between `v5.0.0-patch-1` and `v5.1.0`, so the two interoperate and a rolled-back node does not diverge. Report the problem so the team can help.

## Guides

- **[Local test guide](local-test-guide.md)** — for builders testing the upgrade locally against mainnet data.
