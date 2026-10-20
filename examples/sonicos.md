# SonicOS 7: Dynamic External Address Group

Load the blocklist into a SonicOS Dynamic External Address Group (DEAG) and deny it in an access rule placed above the default SSL VPN rule.

## Feed URLs

```text
# Free sample (this repo, raw GitHub)
https://raw.githubusercontent.com/pcchiu2023/threatlistpro-blocklist/main/data/sample/blocklist.txt

# Same sample from the API
https://api.threatlistpro.com/v1/sample/blocklist.txt

# Full commercial feed (subscribers)
https://api.threatlistpro.com/v1/YOUR_API_KEY/blocklist.txt
```

## About the list

- 200-entry sample; the full feed covers 5.4M+ IPv4 addresses. Built from attacks observed on live production Palo Alto GlobalProtect VPN portals in the US.
- IPv4 only. One IPv4 address or IPv4 CIDR per line. A CIDR range covers addresses that did not each attack.
- SonicOS accepts IP addresses one per line, including subnets in CIDR format.
- The file starts with `#` comment lines. The SonicOS documentation does not say whether comment lines are accepted. If the download shows an error (step 2), host a copy without the comment header.
- The subscriber URL contains your API key. Treat it as a secret.

## 1. Create the Dynamic External Address Group

SonicOS 7.1 path: **OBJECT | Match Objects > Dynamic Group**, then click **Add**. (The SonicOS 7.X knowledge-base article names the page **Dynamic External Objects**.)

1. **Name**: letters and numbers only, no spaces, for example `TLPblocklist`. SonicOS adds the prefix `DEAG_` when you save.
2. **Zone Assignment**: the zone for the group, normally WAN.
3. Leave **FQDN** disabled. This list holds IP addresses only, and an IP-type group does not accept FQDNs.
4. Enable **Periodic Download** and choose a **Download Interval**: 5 minutes, 15 minutes, 1 hour or 24 hours.
5. **Protocol**: **HTTPS**. **URL Name**: one of the feed URLs above (it must start with `https://`).
6. Click **Save**.

The firewall then creates a read-only address group plus a read-only address object for each valid, unique entry. You cannot edit or delete these objects, and you cannot change the group's name or zone later.

## 2. Verify the list loaded

- On the Dynamic Group / Dynamic External Objects page, check the **Comments** column: a green circle means the file downloaded successfully, a red circle means an error.

## 3. Deny inbound traffic to SSL VPN

SonicOS adds a default **WAN to WAN** access rule for the SSLVPN service. SonicWall's guidance for blocking specific public addresses from SSL VPN is to leave that default rule in place and add a Deny rule with a higher priority. The Deny rule blocks those addresses from NetExtender, Mobile Connect and Virtual Office.

Classic mode:

1. Go to **POLICY | Rules and Policies | Access Rules** and open the **WAN to WAN** rules.
2. Note the destination and service of the default SSLVPN rule.
3. Click **Add**. Set **Action** to **Deny** (or **Discard**).
4. Source and destination **Zone/Interface**: WAN to WAN. **Source**: the DEAG. **Destination** and **Service**: the same as the default SSLVPN rule.
5. On the **Logging** tab, enable logging for the rule.
6. Save. Make sure the new rule has a higher priority than the default SSLVPN rule.

Policy mode: a DEAG can also be used in security policies.

The same DEAG can also be the source of a Deny rule from WAN to other zones.

### Testing first

SonicOS access rule actions are Allow, Deny and Discard; there is no log-only action. Check the group's contents (step 2) before you save the rule, and review the logs right after enabling it.

## Capacity

- IP-type DEAOs (the objects a DEAG creates): at most 25% of the total address objects the device supports.
- DEAGs (IP and FQDN types together): at most 25% of the total address groups the device supports.
- Each DEAG automatically gets one DEAO for `0.0.0.0`, so with a single DEAG the file can hold one fewer IP address than the DEAO maximum.

## Vendor docs

- [SonicOS 7.1 Objects: Adding a Dynamic External Address Group](https://www.sonicwall.com/support/technical-documentation/docs/sonicos-7-1-objects/Content/Match_Objects/Dynamic_Group/adding-object.htm)
- [SonicOS 7.1 Objects: About the DEAG file](https://www.sonicwall.com/support/technical-documentation/docs/sonicos-7-1-objects/Content/Match_Objects/Dynamic_Group/about-deag-file.htm)
- [SonicOS 7.1 Objects: DEAG and DEAO Maximums](https://www.sonicwall.com/support/technical-documentation/docs/sonicos-7-1-objects/Content/Match_Objects/Dynamic_Group/deag-deao-maximums.htm)
- [KB: What are dynamic external objects/groups and how can we configure it?](https://www.sonicwall.com/support/knowledge-base/what-are-dynamic-external-objects-groups-and-how-can-we-configure-it/200507105852280)
- [KB: How to block certain public IP addresses from accessing SSL VPN?](https://www.sonicwall.com/support/knowledge-base/how-to-block-certain-public-ip-addresses-from-accessing-ssl-vpn/kA1VN0000000KRs0AM)
- [KB: How to restrict SSLVPN access based on source WAN IPs](https://www.sonicwall.com/support/knowledge-base/how-to-restrict-sslvpn-access-to-the-sonicwall-firewall-based-on-source-wan-ip-s/kA1VN0000000Fs00AE)
- [SonicOS 7.0 Rules and Policies (Classic Mode): Creating access rules](https://www.sonicwall.com/support/technical-documentation/docs/sonicos-7-0-0-0-rules-and-policies-classic-mode/Content/Access_Rules/access_rules_create.htm)

Verified against vendor documentation on 2026-10-20.
