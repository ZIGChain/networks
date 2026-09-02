---
title: "Builder Guide — Test the v4.1 → v5 Upgrade Locally with Mainnet Data"
type: guide
audience: builders / node operators
upgrade_name: v5
status: verified end-to-end (darwin/arm64 + Debian x86_64); see Troubleshooting for snapshot seen-commit panic
---

# Test the v4.1 → v5 Upgrade Locally with Mainnet Data

This guide forks the live `zigchain-1` mainnet state onto your machine as a single-validator chain, then runs the **v5** (`uzig` → `azig`) redenomination upgrade against it — the same governance software-upgrade used on mainnet, on real state, safely offline.

> **1 ZIG stays 1 ZIG.** v5 is a pure ×10¹² rescale of the base unit `uzig` (6 decimals) → `azig` (18 decimals). Balances-in-ZIG, prices, and total value do not change. `uzig` remains only as IBC escrow backing.

All binaries come from the [`ZIGChain/networks`](https://github.com/ZIGChain/networks) repository — you do **not** build from source.

---

## 1. Prerequisites

- `curl`, `jq`, `lz4`, `tar`, and a SHA-256 checker (`shasum` on macOS, `sha256sum` on Linux)
- ~3 GB free disk (extracted `data/` ≈ 1.1 GB + `wasm/` ≈ 0.8 GB, plus the download + decompressed tar)
- Linux `x86_64` or macOS (`arm64` / `amd64`)

---

## 2. Get the binaries

You need **two** binaries from the `ZIGChain/networks` repo.

| Purpose | Version | Folder |
|---|---|---|
| Load the mainnet snapshot & run Phase 1 | `v4.1.0` | `binaries/archive/` |
| Run the v5 upgrade in Phase 2 | `v5.0.0-rc.1-qa-m3off` | `binaries/archive/v5.0.0-rc.1/` (stale testing build — see note) |

> **Why a `-qa-m3off` build for Phase 2?** `in-place-testnet` replaces the real validator set with your single local validator. The production v5 binary enforces a strict *bonded-pool = Σ validator tokens* check that this substitution intentionally breaks, so the real release would halt. The `-qa-m3off` build disables **only** that one check so the upgrade can complete on a forked chain. **It is for local testing only — never run it on a real network.**

`REF=main` fetches from the public repo's default branch. On Linux, run `export SHACHECK='sha256sum'` before the block below (macOS uses the default `shasum -a 256`).

```bash
OS=linux; ARCH=amd64          # or darwin/arm64, darwin/amd64
REF=main
SHACHECK=${SHACHECK:-shasum -a 256}   # Linux: export SHACHECK='sha256sum' first
BASE="https://github.com/ZIGChain/networks/raw/refs/heads/$REF/binaries"

fetch() {  # <subdir> <version>
  local sub="$1" ver="$2" f="zigchaind-$2-$OS-$ARCH.tar.gz"
  curl -fL -o "/tmp/$f" "$BASE/$sub$f"
  curl -fL -o "/tmp/SHA256SUMS-$ver.txt" "$BASE/${sub}SHA256SUMS-$ver.txt"
  ( cd /tmp && grep " $f\$" "SHA256SUMS-$ver.txt" | $SHACHECK -c - )
  # tarballs wrap the binary in zigchaind-<ver>-<os>-<arch>/ (goreleaser wrap_in_directory)
  tar -xzf "/tmp/$f" -C /tmp
  mv "/tmp/zigchaind-$ver-$OS-$ARCH/zigchaind" "/tmp/zigchaind-$ver"
}

fetch "archive/" v4.1.0
fetch "archive/v5.0.0-rc.1/" v5.0.0-rc.1-qa-m3off

V41=/tmp/zigchaind-v4.1.0
V5=/tmp/zigchaind-v5.0.0-rc.1-qa-m3off
"$V41" version    # v4.1.0
"$V5"  version    # v5.0.0-rc.1-qa-m3off
```

---

## 3. Phase 1 — fork mainnet onto your machine (v4.1)

```bash
export ZH=$HOME/.zigchain-qa   # dedicated home; leave any existing ~/.zigchain alone
rm -rf "$ZH"

# init + mainnet genesis
"$V41" init qa-node --chain-id zigchain-1 --home "$ZH"
curl -fL https://github.com/ZIGChain/networks/raw/refs/heads/main/zigchain-1/genesis.json \
  -o "$ZH/config/genesis.json"

# local key — becomes the sole validator and a funded account
"$V41" keys add qa --keyring-backend test --home "$ZH"
ACC=$("$V41" keys show qa -a --keyring-backend test --home "$ZH")
VALOPER=$("$V41" keys show qa --bech val -a --keyring-backend test --home "$ZH")

# config: keep all history so you can query pre/post-upgrade heights
sed -i.bak 's/^pruning = .*/pruning = "nothing"/' "$ZH/config/app.toml"
sed -i.bak 's/^minimum-gas-prices = .*/minimum-gas-prices = "0uzig"/' "$ZH/config/app.toml"

# bootstrap snapshot (pruned) — latest URL + checksum from the official metadata
# (same source snapshots.zigchain.com reads; archive variant: node_->archive_)
SNAP_JSON=https://raw.githubusercontent.com/cryptocrew-validators/CryptoCrew-Validators/refs/heads/main/chains/zigchain/node_snapshot.json
SNAP_URL=$(curl -fsSL "$SNAP_JSON" | jq -r .url)
curl -fL -o /tmp/snap.tar.lz4 "$SNAP_URL"
echo "$(curl -fsSL "$SNAP_JSON" | jq -r .checksum)  /tmp/snap.tar.lz4" | shasum -a 256 -c -
# decompress + extract in one pipe (lz4's trailing "frameType_unknown" is benign)
lz4 -dc /tmp/snap.tar.lz4 2>/dev/null | tar -xf - -C "$ZH"   # -> $ZH/data + $ZH/wasm

# fork to a local single-validator chain and start producing blocks
"$V41" in-place-testnet zigchain-qa-1 "$VALOPER" \
  --home "$ZH" --accounts-to-fund "$ACC" --skip-confirmation
```

> **Note:** a `Truncated tar archive` warning on a `wasm/cache/…` path during extraction is safe to ignore — that's the regenerable wasmer module cache at the tail of the archive. What matters is that `data/` (`blockstore.db`, `application.db`, `state.db`) extracted fully — verify with `ls "$ZH/data"`.

> ⚠️ **If startup panics** with `converting seen commit: validator address is present` right after `Completed ABCI Handshake`, the snapshot's tip block is the problem, not your machine — see [Troubleshooting → Snapshot seen-commit panic](#snapshot-seen-commit-panic-most-common-blocker). Reproducible and deterministic (identical appHash + panic on Debian x86_64 and macOS arm64).

Leave this running in its own terminal. It produces empty ~5 s blocks, fully offline (0 peers).

> A burst of `error adding vote … cannot find validator N in valSet of size 1` at startup is **expected** — the WAL replays the snapshot's last block, whose precommits are from mainnet's full validator set, against your single local validator. Once `Replay: Done` appears and blocks advance, it's fine.

> The forked chain's id is **`zigchain-qa-1`** — every transaction below uses `--chain-id zigchain-qa-1`.

---

## 4. Phase 2 — upgrade to v5

In a second terminal. `in-place-testnet` sets a 2-minute expedited voting period, so submit an **expedited** software-upgrade proposal. Re-declare the paths from §2 — this is a fresh shell.

```bash
export ZH=$HOME/.zigchain-qa
V41=/tmp/zigchaind-v4.1.0        # from §2; unset in a new terminal
V5=/tmp/zigchaind-v5.0.0-rc.1-qa-m3off
ND=tcp://localhost:26657
CUR=$("$V41" status --node $ND 2>/dev/null | jq -r .sync_info.latest_block_height)
H=$((CUR + 120))                                 # upgrade height (~8 min ahead)
echo "current=$CUR  upgrade height=$H"           # sanity-check: H must be ~current+120, not 120
GOV=zig10d07y265gmmuvt4z0w9aw880jnsr700jmgkh5m   # gov module authority

cat > /tmp/prop.json <<JSON
{ "messages": [ { "@type": "/cosmos.upgrade.v1beta1.MsgSoftwareUpgrade",
    "authority": "$GOV", "plan": { "name": "v5", "height": "$H", "info": "" } } ],
  "deposit": "20000uzig", "expedited": true,
  "title": "v5 upgrade test", "summary": "uzig->azig redenomination" }
JSON

"$V41" tx gov submit-proposal /tmp/prop.json --from qa --keyring-backend test \
  --home "$ZH" --chain-id zigchain-qa-1 --node $ND \
  --gas auto --gas-adjustment 1.5 --fees 5000uzig --yes

PID=$("$V41" query gov proposals --node $ND -o json \
  | jq -r '[.proposals[]|select(.status=="PROPOSAL_STATUS_VOTING_PERIOD")]|last|.id')
echo "voting on proposal $PID (upgrade at height $H)"
"$V41" tx gov vote "$PID" yes --from qa --keyring-backend test \
  --home "$ZH" --chain-id zigchain-qa-1 --node $ND \
  --gas auto --gas-adjustment 1.5 --fees 5000uzig --yes
```

After ~2 min voting closes — confirm it passed, then let the chain run to height `H`:

```bash
"$V41" q gov proposal "$PID" --node $ND -o json | jq -r .proposal.status   # -> PROPOSAL_STATUS_PASSED
"$V41" q gov tally "$PID" --node $ND -o json | jq -r .tally.yes_count       # your validator's full power
```

At height **H** the v4.1 node logs `ERR ... UPGRADE "v5" NEEDED at height: H` and stops producing blocks. Swap binaries:

```bash
# stop the v4.1 node: Ctrl-C in its terminal, or from another shell:
pkill -f "in-place-testnet zigchain-qa-1"
# start the v5 binary on the same home:
"$V5" start --home "$ZH"   # expect "Upgrade v5 complete", then blocks continue past H
```

---

## 5. Verify the redenomination

```bash
ND=tcp://localhost:26657
ACC=$("$V5" keys show qa -a --keyring-backend test --home "$ZH")   # if unset in this shell
"$V5" query staking params --node $ND -o json | jq -r .params.bond_denom   # -> azig
"$V5" query bank total --node $ND -o json | jq -r '.supply[].denom'   # only azig + coin.*
"$V5" query bank balances "$ACC" --node $ND -o json | jq -c .balances   # azig (×1e12)

# a working post-upgrade transfer (5 ZIG = 5e18 azig)
"$V5" tx bank send qa "$ACC" 5000000000000000000azig --from qa --keyring-backend test \
  --home "$ZH" --chain-id zigchain-qa-1 --node $ND \
  --gas auto --gas-adjustment 1.4 --fees 5000000000000000azig --yes
```

Expected: `bond_denom` is `azig`, every former `uzig` amount is multiplied by 10¹², the only remaining `uzig` is IBC escrow, and the chain keeps producing blocks and accepting transactions.

---

## 6. Troubleshooting

### Snapshot seen-commit panic (most common blocker)

Phase 1 `in-place-testnet` panics right after `Completed ABCI Handshake` with `converting seen commit: validator address is present`. It's a property of the snapshot's **tip block** (a `CommitSig` for an absent validator that CometBFT v0.38.21 rejects), not your machine — reproducible across OS/arch. Only `in-place-testnet` hits it; a normal `zigchaind start` doesn't. Two fixes:

**A — roll back one block** (uses the snapshot you already have):
```bash
"$V41" rollback --home "$ZH"
printf '{"height":"0","round":0,"step":0}\n' > "$ZH/data/priv_validator_state.json"   # required, else "height regression"
# then re-run the §3 in-place-testnet command
```

**B — use a Polkachu snapshot** (works unmodified; its `priv_validator_state.json` is already zeroed). No scriptable "latest" endpoint, so grab the current URL manually and swap it into §3:
```bash
curl -fL -o /tmp/snap.tar.lz4 https://snapshots.polkachu.com/snapshots/zigchain/zigchain_<HEIGHT>.tar.lz4
```

### Other

- **`error signing vote: height regression`** — you rolled back without resetting `priv_validator_state.json` (see Fix A).
- **`cannot find validator N in valSet of size 1`** at startup — expected; harmless once `Replay: Done` appears.
- **Chain halts on the first v5 block (bonded-pool assertion)** — you're on the plain `v5.0.0-rc.1` binary; use the `-qa-m3off` build for Phase 2 (§2).

---

## 7. Cleanup

```bash
# stop the node, then:
rm -rf "$HOME/.zigchain-qa" /tmp/snap.tar.lz4 /tmp/snap.tar /tmp/zigchaind-v*
```
