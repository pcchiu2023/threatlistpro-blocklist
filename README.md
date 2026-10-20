# ThreatListPro free IPv4 blocklist sample

ThreatListPro publishes a free 200-entry sample of an IPv4/CIDR blocklist built from attacks observed on live production Palo Alto GlobalProtect VPN portals in the US. It is intended for testing External Dynamic List (EDL) and external threat-feed integrations; the commercial feed remains the full product. At the 2026-10-20 baseline, that feed covered 6,615,277 unique IPv4 addresses; CIDR coverage does not mean every covered address attacked.

## Files

| Path | What it is |
|---|---|
| [`data/sample/blocklist.txt`](data/sample/blocklist.txt) | The 200-entry sample: one IPv4 address or IPv4 CIDR per line, `#` comment lines at the top |
| [`data/sample/metadata.json`](data/sample/metadata.json) | Entry count, SHA-256 of the entries, time of the last change, full-feed totals |
| [`data/history/daily.csv`](data/history/daily.csv) | One row per UTC day for the full feed: entries, unique IPv4 addresses covered, entries added and removed (counts only) |
| [`schema/history.schema.json`](schema/history.schema.json) | Column definitions for `daily.csv` |
| [`docs/FORMAT.md`](docs/FORMAT.md) | File format and how firewalls parse it |
| [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) | Where entries come from and how the files are produced |
| [`docs/LIMITATIONS.md`](docs/LIMITATIONS.md) | What the list is not, and how to test it safely |
| [`examples/`](examples/) | Setup for PAN-OS, FortiOS, pfSense (pfBlockerNG), OPNsense, SonicOS and Sophos Firewall |

A GitHub Action refreshes the sample and the daily totals once a day and commits only when something changed.

## Use the sample

Raw URL for a firewall (plain text over HTTPS):

```
https://raw.githubusercontent.com/pcchiu2023/threatlistpro-blocklist/main/data/sample/blocklist.txt
```

The same 200 entries are served at `https://api.threatlistpro.com/v1/sample/blocklist.txt`. Test in a log-only or alert rule before you deny traffic; see [docs/LIMITATIONS.md](docs/LIMITATIONS.md).

## The sample and the full feed

| | Sample (this repository) | Full feed |
|---|---|---|
| IPv4 addresses covered | at most 500,000 (467,360 on 2026-10-20) | 5.4M+ (6,615,277 at the 2026-10-20 baseline) |
| Entries | 200, taken from the full feed in feed order | 2,763 at the 2026-10-20 baseline |
| Source | attacks on live production Palo Alto GlobalProtect VPN portals in the US | same |
| Checked for new attackers | taken from the full feed, which is checked every 6 hours; copied here daily | every 6 hours |
| Access | free, no signup | subscription, [threatlistpro.com](https://threatlistpro.com) |

The sample is chosen deterministically, not rotated: it changes only when the start of the full feed changes, and each change is kept in this repository's history.

## License

The data in `data/` is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); everything outside `data/` (workflow, schema, docs and examples) is under the MIT License. Neither license grants access to the full commercial feed. See [LICENSE.md](LICENSE.md). To cite this repository, use GitHub's "Cite this repository" button ([CITATION.cff](CITATION.cff)).

## Contact

Questions, corrections or false-positive reports: open an issue or email support@threatlistpro.com. Maintained by Phil Chiu, founder of [ThreatListPro](https://threatlistpro.com/about).
