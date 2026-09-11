# ElectrumX for Doichain Core 31.1 (branch `feat/doichain-31.1`)

## Important scope note

ElectrumX is an **indexing server**, not a validating node: it trusts `doichaind`
and indexes what the daemon serves. It does **no proof‑of‑work or difficulty
validation**. Therefore the 31.1 consensus changes — DigiShield DAA, the
re‑enabled PoW‑difficulty check, the reset window — require **no change here**.
What actually matters for ElectrumX is correct **block / transaction / name
parsing** and the **AuxPoW header handling**.

The Doichain coin class already carries the name layer, incl. `name_doi`:
`electrumx/lib/coins.py` → `class Doichain(NameIndexMixin, AuxPowMixin, Coin)`,
with `OP_NAME_DOI = OP_10` and `NAME_DOI_OPS` (mirrors `NAME_UPDATE_OPS`).

## To verify / update for 31.1 (against a real 31.1 node)

- [ ] **name_doi script parity** — confirm `NAME_DOI_OPS` = `[OP_10, "name", -1,
  OP_2DROP, OP_DROP]` matches the on‑chain output script that 31.1's
  `name_doi` produces (`buildNameDOI` in doichain-core). Index a real `name_doi`
  tx and check the name/value come out right.
- [ ] **AuxPoW + SegWit deserialization** — `DeserializerAuxPowSegWit` must parse
  31.1 blocks (Core 31 blocks are always SegWit). Index across a block with an
  AuxPoW header and one without.
- [ ] **Coin stats** — `TX_COUNT` / `TX_COUNT_HEIGHT` / `TX_PER_BLOCK` are
  placeholders (`1/1/10`); set to real values from `getchaintxstats` so sync
  progress is sane.
- [ ] **GENESIS_HASH** unchanged (`000006fdd8…`) — the relaunch keeps the chain,
  so this stays. ✓
- [ ] **AuxPoW checkpoint** (session.py truncates AuxPoW data below a checkpoint)
  — set/verify for the relaunch height if used.
- [ ] **PEERS** list is empty — add the mainnet ElectrumX peer(s) once deployed.

## Test

1. Build ElectrumX (this branch) and point `DAEMON_URL` at a 31.1 `doichaind`
   (RPC 8339).
2. Index a height range that contains name_new / name_firstupdate / name_update /
   **name_doi** and normal address txs.
3. Verify with an Electrum client: address balances/history and `name_show`‑style
   lookups match the daemon.

## Deployment

Will be wired as a service in `doichain-install` (`feat/doichain-31.1`) alongside
`doichaind` 31.1 and p2pool.
