# Methodology

## Source

Entries come from attacks observed on live production Palo Alto GlobalProtect VPN portals in the US, operated by the maintainer. Entries are single addresses or CIDR ranges; a range covers addresses that did not each attack.

## Production

1. The list is checked for new attackers every 6 hours. Each check validates every line as an IPv4 address or IPv4 CIDR, removes duplicates, and keeps the previous list if the new one looks incomplete (fewer than 100 entries).
2. The free sample is 200 entries taken from the full list in list order, skipping any entry that would take the sample past 500,000 addresses (so most large ranges are left out). It is served at `https://api.threatlistpro.com/v1/sample/blocklist.txt`.
3. Once per UTC day, at the first check after midnight UTC, the service records a snapshot of the full list: the number of entries, unique IPv4 addresses covered (overlapping ranges merged), how many entries were added and removed since the previous snapshot, and a SHA-256 hash of the list. Only these totals are published, at `https://api.threatlistpro.com/v1/history.csv`; the added and removed entries themselves are not.
4. A GitHub Action in this repository copies the sample and the daily totals once a day, validates them (exactly 200 valid, unique entries; the expected CSV header), and commits only when something changed.

The first snapshot, on 2026-10-20, is a baseline (`baseline=true`): it has no previous list to compare with, so its added and removed counts are 0.

## Reading the daily totals

- New entries arrive irregularly. Attackers pause, return from new addresses, and attempt less once a site blocks them, so a day with no change is normal.
- `added` and `removed` count entries, not addresses. Replacing several single addresses with one CIDR range shows as several removals and one addition.
- `list_sha256` shows whether the full list changed between two days without revealing it.
