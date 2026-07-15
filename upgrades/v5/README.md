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
| Binary version | pending | pending |
| Upgrade height | pending | pending |
| Cosmovisor height (`height − 1`) | pending | pending |
| Proposal | pending | pending |
| Status | 🧪 RC available | ⏳ pending |

> Heights, proposal links, and binary versions are filled in as each stage is reached — never guessed ahead of time.

## Binaries

### Release candidate — `v5.0.0-rc.1`

These are the **`v5.0.0-rc.1` release-candidate** builds, for testing the upgrade. **Final release binaries are still pending.**

| Platform | Download | SHA-256 |
|----------|----------|---------|
| `linux-amd64` | [zigchaind-v5.0.0-rc.1-linux-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-linux-amd64.tar.gz) | `58c939364c9d390d21c69d114917f659b6648e437ad2a942df0929f2f833d91f` |
| `darwin-arm64` | [zigchaind-v5.0.0-rc.1-darwin-arm64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-darwin-arm64.tar.gz) | `21fb6e315740b38af6d3f91979f39f7c780e8077428247930297dbf30915e899` |
| `darwin-amd64` | [zigchaind-v5.0.0-rc.1-darwin-amd64.tar.gz](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/zigchaind-v5.0.0-rc.1-darwin-amd64.tar.gz) | `819b8cc8009d3c46c95e6f9425c048efc3220390e777626ec11bf5c90da7ebc5` |

Full checksums: [`SHA256SUMS-v5.0.0-rc.1.txt`](https://github.com/ZIGChain/networks/raw/main/binaries/v5.0.0-rc.1/SHA256SUMS-v5.0.0-rc.1.txt). Once the final release is cut, the **authoritative** download URLs + checksums cosmovisor uses for auto-download will live on-chain in the proposal's `plan.info`.

> This release candidate ships as two builds: the standard **`v5.0.0-rc.1`** build above (the release candidate for the upgrade itself), and a **`v5.0.0-rc.1-qa-m3off`** build — the **QA release candidate**, identical except the `in-place-testnet`-incompatible check is disabled, **for local testing only** (see the [local test guide](local-test-guide.md)). Neither is a final release; do not run either on production or mainnet.

## Guides

- **[Local test guide](local-test-guide.md)** — for builders testing the upgrade locally against mainnet data.
