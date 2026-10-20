# OPNsense: URL Table Alias

Load the blocklist into an OPNsense URL Table alias and block inbound traffic from it on the WAN.

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
- The `#` comment lines at the top are fine: OPNsense ignores fetched lines that start with whitespace, `,`, `;`, `|` or `#`.
- The OPNsense docs describe URL table content as one address per line and do not mention CIDR lines explicitly; after the first load, check that CIDR entries appear in **Firewall ‣ Diagnostics ‣ Aliases**.
- The subscriber URL contains your API key. Treat it as a secret.

## 1. Create the alias

1. Go to **Firewall ‣ Aliases** and add a new alias.
2. **Name**: `tlp_blocklist`. **Description**: optional.
3. **Type**: **URL Tables (IPs)** (fetched at regular intervals). Do not use **URL (IPs)**, which is fetched only once.
4. **Content**: one of the feed URLs above.
5. **Refresh frequency**: how often the list is fetched again. The OPNsense Spamhaus how-to sets it to `1` for once a day.
6. **Save**, then **Apply**.

Optional: **Firewall ‣ Settings ‣ Advanced**, **Check certificate of aliases URLs** refuses to download from an HTTPS address whose certificate is invalid or revoked.

To refresh URL tables now, from the shell:

```text
configctl filter refresh_aliases
```

This is the command the "Update and reload firewall aliases" cron job runs. It updates DNS and MAC aliases as well as URL tables.

## 2. Verify the list loaded

- **Firewall ‣ Diagnostics ‣ Aliases**: select `tlp_blocklist` to see the addresses currently in the table.
- **Firewall ‣ Diagnostics ‣ pfTable**: check whether a specific address is in the alias.

## 3. Block inbound traffic to the VPN/portal

OPNsense filters on the incoming direction by default, so the rule goes on the interface the traffic arrives on: WAN.

1. Go to **Firewall ‣ Rules**, open the **WAN** tab and add a rule.
2. **Action**: **Block** (or **Reject**, which also answers the sender).
3. **Interface**: WAN. **TCP/IP Version**: IPv4.
4. **Source**: `tlp_blocklist`.
5. **Destination**: the address of your VPN or portal, or any to cover all inbound traffic on WAN.
6. **Log**: tick "Log packets that are handled by this rule".
7. Keep **Quick** set (first matching rule wins) and place the rule above any rule that passes traffic to the VPN/portal.
8. **Description**, for example `Block tlp_blocklist`. **Save**, then apply the changes.

If clients from listed addresses already have connections open, you may need to remove those states before the block takes effect (**Firewall ‣ Diagnostics ‣ States Dump**).

### Testing first

OPNsense has no log-only rule action. You can:

- create the rule with **Disabled** ticked, check the alias contents (step 2), then enable it;
- watch **Firewall ‣ Log Files ‣ Live View** to confirm the rule matches what you expect.

## Capacity

- **Firewall ‣ Settings ‣ Advanced ‣ Firewall Maximum Table Entries** is the combined limit for aliases, sshlockout, bogons and other tables. The default is shown in the field's help text. Raise it when you load large aliases such as the full feed.

## Vendor docs

- [Aliases (URL Tables, file format, diagnostics)](https://docs.opnsense.org/manual/aliases.html)
- [Configure Spamhaus DROP (URL table alias + WAN block rule how-to)](https://docs.opnsense.org/manual/how-tos/drop.html)
- [Rules (direction, Quick, Log, Disabled)](https://docs.opnsense.org/manual/firewall.html)
- [(Advanced) Settings (Maximum Table Entries, alias URL certificate check)](https://docs.opnsense.org/manual/firewall_settings.html)
- [Settings: Cron (`configctl filter refresh_aliases`)](https://docs.opnsense.org/manual/settingsmenu.html)

Verified against vendor documentation on 2026-10-20.
