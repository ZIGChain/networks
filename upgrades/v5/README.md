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
| Binary version | `v5.0.0` | `v5.0.0` |
| Upgrade height | pending | pending |
| Cosmovisor height (`height − 1`) | pending | pending |
| Proposal | pending | pending |
| Status | ⏳ pending | ⏳ pending |

> Heights and proposal links are filled in as each stage is reached — never guessed ahead of time.

## Binaries

### Release — `v5.0.0`

Use these **`v5.0.0`** builds for the governance software-upgrade on mainnet and testnet.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/zigchaind-v5.0.0-linux-amd64.tar.gz) | `3cf12ad3d40e861d6d8302317fb42695d2994def3bd57cc64f69407a6c21e80d` |
| `darwin-arm64` | [zigchaind-v5.0.0-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/zigchaind-v5.0.0-darwin-arm64.tar.gz) | `7cd3e5deeec6be8c959f11e295bb4f3dca0d7493e6dd23006774e9ebe4cb3714` |
| `darwin-amd64` | [zigchaind-v5.0.0-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/zigchaind-v5.0.0-darwin-amd64.tar.gz) | `a7c4e675b70fb2d2023937491fc3cecc2d3e6fc22bc501d55151cf8f1743e51f` |

Full checksums: [`SHA256SUMS-v5.0.0.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/SHA256SUMS-v5.0.0.txt). The **authoritative** download URLs + checksums cosmovisor uses for auto-download will also live on-chain in the proposal's `plan.info`.

### Release candidate — `v5.0.0-rc.1` (superseded)

Earlier **`v5.0.0-rc.1`** release-candidate builds, kept for reference. Use **`v5.0.0`** above for production upgrades.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-rc.1-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-linux-amd64.tar.gz) | `58c939364c9d390d21c69d114917f659b6648e437ad2a942df0929f2f833d91f` |
| `darwin-arm64` | [zigchaind-v5.0.0-rc.1-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-darwin-arm64.tar.gz) | `21fb6e315740b38af6d3f91979f39f7c780e8077428247930297dbf30915e899` |
| `darwin-amd64` | [zigchaind-v5.0.0-rc.1-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-darwin-amd64.tar.gz) | `819b8cc8009d3c46c95e6f9425c048efc3220390e777626ec11bf5c90da7ebc5` |

Full checksums: [`SHA256SUMS-v5.0.0-rc.1.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/SHA256SUMS-v5.0.0-rc.1.txt).

### QA build — `v5.0.0-rc.1-qa-m3off` (local testing only)

Identical to **`v5.0.0-rc.1`** except the `in-place-testnet`-incompatible check is disabled. **For local testing only** — see the [local test guide](local-test-guide.md). Do not run on production or mainnet.

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-linux-amd64.tar.gz) | `a9b029461ae5a456f6b40acd41ce5c90dbabe8aec2b5039ad8bb477aaefb4b5a` |
| `darwin-arm64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-arm64.tar.gz) | `0d7cb074fbd47672de410f77f642b32056ff9f0118f615b109135789bc165bba` |
| `darwin-amd64` | [zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-qa-m3off-darwin-amd64.tar.gz) | `81fc6b83210c6c4e07880b6522259fd7527aa4de9635c4e97d0185139f73099e` |

Full checksums: [`SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/archive/v5.0.0-rc.1/SHA256SUMS-v5.0.0-rc.1-qa-m3off.txt).

## Guides

- **[Local test guide](local-test-guide.md)** — for builders testing the upgrade locally against mainnet data.
