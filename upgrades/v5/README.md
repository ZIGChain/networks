# ZIGChain v5 Upgrade

**`uzig` → `azig` redenomination.** ZIGChain's base denomination changes from `uzig` (6 decimals) to `azig` (18 decimals). **1 ZIG is still 1 ZIG** — balances, total supply, and value are unchanged; only the on-chain representation gains precision (1 ZIG = 10¹⁸ `azig`, previously 10⁶ `uzig`). `uzig` remains only as IBC escrow backing.

## Who's affected

| Audience | What changes |
|----------|--------------|
| **Exchanges / integrators** | Base denom is now `azig` with **18 decimals** (was `uzig`, 6). Update deposit/withdrawal accounting and display math accordingly. Nominal balances are unchanged — 1 ZIG stays 1 ZIG. |
| **Builders / dApp devs** | Any code that hard-codes `uzig` or 6-decimal math must move to `azig` / 18 decimals. Test against real state first — see the [local test guide](local-test-guide.md). |
| **Validators / node operators** | A standard governance software-upgrade at the coordinates below. |

## Coordinates

| Field | Testnet (`zig-test-2`) | Mainnet (`zigchain-1`) |
|-------|------------------------|------------------------|
| Upgrade name | `v5` | `v5` |
| Binary version | `v5.0.0-patch-1` (upgrade ran on this) | `v5.1.0` |
| Upgrade height | `7,669,200` | pending |
| Cosmovisor height | `7,669,200` | pending |
| Proposal | pending | pending |
| Status | ✅ done | ⏳ pending |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time.

## Binaries

### Release — `v5.1.0`

Use these **`v5.1.0`** builds for the governance software-upgrade. They supersede `v5.0.0-patch-1`: same v5 state machine, plus the patched CosmWasm runtime (`wasmd v0.60.9-rc.3`, `wasmvm v2.3.5-rc.3`). The upgrade name stays `v5` and the height is unchanged.

**Linux only.** The patched wasmvm tags publish no `libwasmvmstatic_darwin.a`, and building one requires compiling osxcross from source, so this release has no darwin build. macOS operators stay on `v5.0.0-patch-1` for local work and run `v5.1.0` on their Linux validators.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.1.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.0/zigchaind-v5.1.0-linux-amd64.tar.gz) | `93f2be769ebafb369ed6fee03a0159ee6699e5aaae27d5ca165bc2b88b42d914` |

Full checksums: [`SHA256SUMS-v5.1.0.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.1.0/SHA256SUMS-v5.1.0.txt). Built from `release/v5` at commit `3dd8a7ea24a62e92d16b6f094a34f27637e00cc5`; `zigchaind version --long` reports `v5.1.0` and `zigchaind query wasm libwasmvm-version` reports `2.3.5-rc.3`.

### Superseded — `v5.0.0-patch-1`

TestNet ran the v5 upgrade on these builds. Kept for the record, and as the last release with darwin artifacts.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-patch-1-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-linux-amd64.tar.gz) | `002c1edb1db0f32ac16bc3faaa96438b1720f9330b2fca438108f19cd49586ee` |
| `darwin-arm64` | [zigchaind-v5.0.0-patch-1-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-darwin-arm64.tar.gz) | `408972867f66ae17fcf6730c5e5c96432f2175a96a36d991b6e0f26d7fa567ef` |
| `darwin-amd64` | [zigchaind-v5.0.0-patch-1-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/zigchaind-v5.0.0-patch-1-darwin-amd64.tar.gz) | `3b9dfc2cfd290fe2cf8e7a2f6ba6cff5f93f9a2f1fb1c2565ca27b17803e9b28` |

Full checksums: [`SHA256SUMS-v5.0.0-patch-1.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-patch-1/SHA256SUMS-v5.0.0-patch-1.txt). The **authoritative** download URLs + checksums cosmovisor uses for auto-download will also live on-chain in the proposal's `plan.info`.

### QA build — `v5.0.0-rc.1-qa-m3off` (local testing only)

A **stale release-candidate** build, kept solely for testing the upgrade against **mainnet data** on a local fork — see the [local test guide](local-test-guide.md). It is based on `v5.0.0-rc.1` (not the final `v5.0.0` release) and disables the `in-place-testnet`-incompatible check so the upgrade can complete offline. **Do not use on production, testnet, or mainnet.**

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz) | `a9b029461ae5a456f6b40acd41ce5c90dbabe8aec2b5039ad8bb477aaefb4b5a` |
| `darwin-arm64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz) | `0d7cb074fbd47672de410f77f642b32056ff9f0118f615b109135789bc165bba` |
| `darwin-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz) | `81fc6b83210c6c4e07880b6522259fd7527aa4de9635c4e97d0185139f73099e` |

Full checksums: [`SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt).

## Guides

- **[Local test guide](local-test-guide.md)** — for builders testing the upgrade locally against mainnet data.
