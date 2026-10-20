# PAN-OS: External Dynamic List (IP List)

Use the blocklist as a PAN-OS External Dynamic List (EDL) of type IP List and deny inbound traffic from it to a GlobalProtect portal or gateway.

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
- The file starts with `#` comment lines. PAN-OS documents comments only after an entry on the same line, and says the firewall skips entries that do not match the list type. Use the CLI check in step 5 to see skipped lines.
- The subscriber URL contains your API key. Treat it as a secret.

## 1. Create the EDL

1. Go to **Objects > External Dynamic Lists** and click **Add**.
2. Enter a **Name**, for example `tlp-blocklist`. Select **Shared** if several virtual systems will use it.
3. Set **Type** to **IP List**.
4. Set **Source** to one of the feed URLs above.
5. The URL is HTTPS, so select a **Certificate Profile** whose root and intermediate CA certificates match the server's certificate chain.
6. Click **Test Source URL**. This confirms the firewall can reach the server. (It is not available when the EDL uses authentication.)
7. Set **Check for updates**. Options are **Every Five Minutes**, **Hourly**, **Daily**, **Weekly** and **Monthly**. The documented default differs between doc versions (Every Five Minutes in the current field reference, hourly in the PAN-OS 11.1 admin guide), so set it explicitly.
8. Click **OK**, then **Commit**.

To fetch the list right away instead of waiting for the interval: select the list on **Objects > External Dynamic Lists** and click **Import Now**. The job appears in the Task Manager.

## 2. Test before you deny

PAN-OS rule actions are Allow or Deny; there is no separate monitor action for a Security policy rule. To see what would match before you block anything:

- Use **Global Find** to check whether a given address belongs to the EDL.
- Use **Device > Troubleshooting**, select **Security Policy Match**, and test a source address from the list against your portal address.
- Review **Monitor > Logs > Traffic** for sessions from listed addresses (rules log at session end by default).

## 3. Deny inbound traffic to the GlobalProtect portal/gateway

GlobalProtect portals and external gateways sit on an interface in an untrust-type zone (for example `l3-untrust`). When a client connects to the portal or gateway, the traffic terminates on the firewall's own interface address. PAN-OS defines the destination zone as "the zone at which the traffic terminates", so the rule's destination is the untrust zone and the portal/gateway interface address.

1. Go to **Policies > Security** and click **Add**. Enter a name, for example `deny-tlp-to-globalprotect`.
2. **Source** tab: Source Zone = your untrust zone. Source Address = the EDL (`tlp-blocklist`).
3. **Destination** tab: Destination Zone = the zone of the portal/gateway interface (your untrust zone). Destination Address = the portal/gateway interface IP address (an address object). If you use NAT in front of the firewall, use the pre-NAT address and the post-NAT zone.
4. **Actions** tab: Action Setting = **Deny**. Leave logging at its default (Log at Session End).
5. Click **OK** and move the rule above any rule that allows traffic to the portal/gateway.
6. **Commit**.

## 4. Use it for other inbound traffic

The same EDL can be the Source Address of any Security policy rule from the untrust zone to internal zones, with action Deny.

## 5. Verify the list loaded

- GUI: open the EDL and check the **List Entries and Exceptions** tab (**View External Dynamic List Entries**).
- CLI, which also shows entries the firewall skipped:

```text
request system external-list show type ip name tlp-blocklist
```

- If a refresh fails, or the list comes back empty, the firewall keeps using the last list it retrieved.

## Capacity

- The PAN-OS 11.1 admin guide states: PA-3200 Series, PA-5200 Series and PA-7000 Series support a maximum of 150,000 total IP addresses across EDLs; all other models support 50,000. Check the EDL page of the admin guide for your release and model.
- A subnet or range counts as one entry. No per-list limit is enforced; the limit is for the firewall as a whole.
- Only lists referenced in policy count toward the limit. Entries beyond the model maximum are ignored, and the firewall generates a syslog message when the limit is reached.
- Up to 30 custom EDLs with unique sources per firewall.

## Vendor docs

- [Configure the Firewall to Access an External Dynamic List (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/configure-the-firewall-to-access-an-external-dynamic-list)
- [Policy Object: External Dynamic Lists (field reference)](https://docs.paloaltonetworks.com/network-security/security-policy/administration/objects/external-dynamic-lists)
- [External Dynamic List (capacity, PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/external-dynamic-list)
- [IP Address List formatting (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/formatting-guidelines-for-an-external-dynamic-list/ip-address-list)
- [Enforce Policy on an External Dynamic List (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/enforce-policy-on-an-external-dynamic-list)
- [Retrieve an External Dynamic List from the Web Server (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/use-an-external-dynamic-list-in-policy/retrieve-an-external-dynamic-list-from-the-web-server)
- [Configure DNS Sinkholing for a List of Custom Domains (`external-list show ... name` syntax)](https://docs.paloaltonetworks.com/advanced-threat-prevention/administration/configure-threat-prevention/use-dns-queries-to-identify-infected-hosts-on-the-network/configure-dns-sinkholing-for-a-list-of-custom-domains)
- [Components of a Security Policy Rule (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/security-policy/components-of-a-security-policy-rule)
- [Create a Security Policy Rule (PAN-OS 11.1)](https://docs.paloaltonetworks.com/pan-os/11-1/pan-os-admin/policy/security-policy/create-a-security-policy-rule)
- [Create Interfaces and Zones for GlobalProtect](https://docs.paloaltonetworks.com/globalprotect/10-1/globalprotect-admin/get-started/create-interfaces-and-zones-for-globalprotect)

Verified against vendor documentation on 2026-10-20.
