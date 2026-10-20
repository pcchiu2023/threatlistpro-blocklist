# File format

`data/sample/blocklist.txt` is ASCII plain text with LF line endings.

- Lines that start with `#` are comments. They appear only at the top of the file.
- Every other line is one entry: an IPv4 address (`203.0.113.7`) or an IPv4 CIDR block (`198.51.100.0/24`).
- Single addresses may be written with or without `/32`.
- No blank lines, no repeated lines, no IPv6, no ports, no hostnames, no trailing comments on entry lines. Ranges can overlap, and a CIDR can have host bits set (for example `31.77.251.21/24`, which firewalls treat as `31.77.251.0/24`).

Validation pattern used by the refresh workflow:

```
^([0-9]{1,3}\.){3}[0-9]{1,3}(/[0-9]{1,2})?$
```

## Firewall compatibility

This is the plain "IP list" format that firewalls accept for URL-fetched address lists, such as Palo Alto Networks External Dynamic Lists (type IP List) and FortiGate threat feeds (IP address). Step-by-step setup for six platforms is in [examples/](../examples/).

If a parser rejects the `#` comment lines, strip them before import:

```
grep -v '^#' blocklist.txt
```

## Entry count vs address count

Firewall capacity limits usually count entries, not addresses. A `/24` is one entry that covers 256 addresses. `unique_ipv4_addresses` in `daily.csv` merges overlapping ranges, so no address is counted twice.

## Hashes

`entries_sha256` in `metadata.json` is the SHA-256 of the entry lines (comments removed), each ending in LF: `grep -v '^#' blocklist.txt | sha256sum`. `list_sha256` in `daily.csv` hashes the full feed's entries joined by LF with no trailing newline.
