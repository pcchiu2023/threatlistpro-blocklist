# pfSense: pfBlockerNG or URL Table Alias

Load the blocklist into pfSense, with the pfBlockerNG package or a built-in URL Table (IPs) alias, and block inbound traffic from it on the WAN.

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
- The file starts with `#` comment lines. The Netgate alias documentation does not say how comment lines are handled, so check the loaded table (Verify, below) after the first download.
- The subscriber URL contains your API key. Treat it as a secret.

## Option A: pfBlockerNG

Netgate's pfBlocker-NG package page covers the concepts below but not the package's current menu paths, so the steps follow its section names.

1. Install the pfBlockerNG package.
2. **General Setup**: set the interfaces pfBlockerNG monitors. The inbound interface is the Internet connection (WAN).
3. Add an IP list that uses one of the feed URLs above. pfBlockerNG takes IP addresses and ranges in file or CIDR format.
4. Set the list action to **Deny Inbound** (blocks access from the list to your network), or **Alias Only** to create an alias that you use in your own rule (see the WAN rule below).
5. pfBlockerNG needs at least one firewall entry on some interface to be active; the dashboard widget shows whether it is.

pfBlockerNG creates URL Table aliases behind the scenes, so the Verify and Capacity notes below apply to it too.

## Option B: URL Table (IPs) alias, no package

1. Go to **Firewall > Aliases** and click **Add**.
2. **Name**: `tlp_blocklist` (letters, digits and underscores only). **Description**: optional.
3. **Type**: **URL Table (IPs)**.
4. Enter a feed URL. The drop-down after the `/` is the update frequency, in **days**. The firewall checks once per day whether a URL Table alias is due for an update.
5. Click **Save**, then apply the changes.
6. Optional: **System > Advanced**, **Firewall & NAT** tab, **Verify HTTPS certificates when downloading alias URLs** makes the firewall require a valid HTTPS certificate.

## Block inbound traffic to the VPN/portal

1. Go to **Firewall > Rules** and open the **WAN** tab. Rules on an interface tab filter packets as they enter on that interface, so WAN rules see traffic arriving from the Internet.
2. Add a rule: **Action** = **Block** (or **Reject**). **Source**: choose **Address or Alias**, type the first letter of the alias name and pick `tlp_blocklist` (with Option A, the pfBlockerNG alias).
3. **Destination**: the address of your VPN or portal, or any to cover all inbound traffic on WAN.
4. Tick **Log** ("Log packets that are handled by this rule").
5. Place the rule above the rule that passes traffic to the VPN/portal. Save and apply.

### Testing first

pfSense rules have no log-only action. To stage the rule, add a Pass rule with the same source and destination and **Log** ticked above the existing pass rule, review the firewall log, then change the action to Block.

## Verify the list loaded

- **Diagnostics > Tables**: pick the alias from the **Table** drop-down to list its current contents.
- Hover over the alias on **Firewall > Rules** to see a tooltip of its contents.

## Capacity

- **System > Advanced > Firewall & NAT > Firewall Maximum Table Entries** defaults to 400,000. Netgate advises setting it to at least double the total entries of all aliases combined when you use URL Table aliases or pfBlockerNG; if it is too small the ruleset may fail to load.
- Manually created aliases are limited to 5,000 members; larger lists belong in URL Table aliases.

## Vendor docs

- [pfBlocker-NG Package](https://docs.netgate.com/pfsense/en/latest/packages/pfblocker.html)
- [Aliases](https://docs.netgate.com/pfsense/en/latest/firewall/aliases.html)
- [Alias Types (URL Table, update frequency)](https://docs.netgate.com/pfsense/en/latest/firewall/aliases-types.html)
- [Alias Configuration](https://docs.netgate.com/pfsense/en/latest/firewall/aliases-config.html)
- [Alias Usage](https://docs.netgate.com/pfsense/en/latest/firewall/aliases-usage.html)
- [Firewall Rule Configuration](https://docs.netgate.com/pfsense/en/latest/firewall/configure.html)
- [Firewall & NAT advanced settings (Maximum Table Entries)](https://docs.netgate.com/pfsense/en/latest/config/advanced-firewall-nat.html)
- [Firewall Table Contents (Diagnostics > Tables)](https://docs.netgate.com/pfsense/en/latest/monitoring/status/firewall-tables.html)

Verified against vendor documentation on 2026-10-20.
