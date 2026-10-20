# Limitations

- **One kind of source.** Entries come from attacks on live production Palo Alto GlobalProtect VPN portals in the US. The list targets VPN brute force and credential attacks; it is not a general malware, spam, botnet or scanner list.
- **IPv4 only.** There are no IPv6 entries.
- **CIDR coverage.** Some entries are ranges. Not every address inside a listed range attacked.
- **No guarantee.** An address that is not listed may still be malicious, and a listed address can later be reassigned to someone else (cloud, hosting and residential addresses change hands). Report suspected false positives by opening an issue or emailing support@threatlistpro.com.
- **The sample is partial.** It is 200 entries covering at most 500,000 addresses, chosen in feed order, and does not rotate.
- **Inbound use.** The list is built for inbound traffic toward VPN portals and other exposed services, not for outbound filtering.

## Test before you block

1. Load the list and confirm the entry count your firewall reports (200 lines in the sample; firewalls that merge overlapping ranges may report slightly fewer).
2. Create the rule with a log or alert action first and watch matches for a few days.
3. Keep an allow list for your own users' and partners' addresses, ordered before the deny rule.
4. Switch the rule to deny.
