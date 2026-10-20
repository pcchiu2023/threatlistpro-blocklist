# FortiOS: IP Address External Feed

Use the blocklist as a FortiGate IP Address external feed (threat feed) and deny traffic from it to the FortiGate's own VPN service with a local-in policy.

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
- The `#` comment lines at the top are supported: FortiOS accepts comments that start with the number sign.
- The subscriber URL contains your API key. Treat it as a secret.

## 1. Create the external feed

GUI (FortiOS 7.6 calls it an "external feed"; older releases say "threat feed"):

1. Go to **Security Fabric > External Connectors** and click **Create New**.
2. Under **External Feeds**, click **IP Address**.
3. **Name**: `tlp-blocklist`. **Update method**: **External Feed**.
4. **URL of external resource**: one of the feed URLs above.
5. **Refresh Rate**: in minutes, 1 to 43200, default 5.
6. Click **OK**.

CLI:

```text
config system external-resource
    edit "tlp-blocklist"
        set type address
        set resource "https://api.threatlistpro.com/v1/sample/blocklist.txt"
        set refresh-rate 60
        set server-identity-check full
    next
end
```

`server-identity-check` defaults to `none`; Fortinet recommends `basic` or `full`.

## 2. Verify the list loaded

- GUI: edit the connector and click **View Entries**.
- CLI: shows total, valid and error lines for each feed:

```text
diagnose sys external-resource stats
```

## 3. Deny inbound traffic to the VPN service (local-in policy)

Local-in policies control traffic going to a FortiGate interface, including VPN services. An IP Address external feed can be used as the source.

GUI (FortiOS 7.6): **Policy & Objects > Local-In Policy** > **Local-In Policy** tab > **Create new**.

CLI:

```text
config firewall local-in-policy
    edit 1
        set intf "port1"
        set srcaddr "tlp-blocklist"
        set dstaddr "all"
        set action deny
        set service "<vpn-service>"
        set schedule "always"
        set logtraffic enable
        set comments "Deny tlp-blocklist to VPN"
    next
end
```

- `edit 1`: use a free policy ID. `intf`: the interface your VPN listens on (`port1` is an example).
- `service`: a service object matching the protocol and port your VPN listens on.
- `action` defaults to `deny` if unset. Local-in policies have no implicit deny, so this rule blocks only what it matches.

Test it with the debug flow commands from the Fortinet guide; a match shows `iprope_in_check() check failed on policy <id>, drop`:

```text
diagnose debug flow filter addr <listed IP>
diagnose debug enable
diagnose debug flow trace start 10
```

### SSL VPN changes in FortiOS 7.6

- From FortiOS 7.6.3, SSL VPN tunnel mode is replaced by IPsec VPN, which can be configured to use TCP port 443. Tunnel-mode settings are not upgraded, so migrate before upgrading.
- SSL VPN web mode is now called Agentless VPN.
- From 7.6.0, FortiGate models with 2 GB of RAM or less (e.g. FGT-40F, 60F, 61F) no longer have SSL VPN web or tunnel mode.

**Other uses:** the feed can also be the Source of a firewall policy (**Policy & Objects > Firewall Policy**, IP ADDRESS FEED section) with **Action** = **DENY**.

**Testing first:** local-in actions are only `accept` and `deny`. Create the rule with `set status disable`, confirm the entries in step 2, then enable it and watch the logs.

## Capacity

- The FortiOS 7.6.5 guide lists IP address entry limits of 5,000,000 (high-end), 1,000,000 (mid-range) and 300,000 (entry-level), with file size limits of 128 MB, 64 MB and 32 MB.
- The limit is global. Entries over the limit are not loaded, and the most recently added entries are truncated first.
- `print tablesize` shows the feed (table) count limit for your model (`system.external-resource`).

## Vendor docs

- [IP address external feed (FortiOS 7.6.4)](https://docs.fortinet.com/document/fortigate/7.6.4/administration-guide/891236/ip-address-external-feed)
- [Configuring an external feed (FortiOS 7.6.4)](https://docs.fortinet.com/document/fortigate/7.6.4/administration-guide/379433/configuring-an-external-feed)
- [External feeds: file format and entry limits (FortiOS 7.6.5)](https://docs.fortinet.com/document/fortigate/7.6.5/administration-guide/9463/external-feeds)
- [Local-in policy (FortiOS 7.6.6)](https://docs.fortinet.com/document/fortigate/7.6.6/administration-guide/363127/local-in-policy)
- [config firewall local-in-policy (CLI reference 7.6.6)](https://docs.fortinet.com/document/fortigate/7.6.6/cli-reference/185227842/config-firewall-local-in-policy)
- [SSL VPN tunnel mode replaced with IPsec VPN (7.6.6 release notes)](https://docs.fortinet.com/document/fortigate/7.6.6/fortios-release-notes/173430/ssl-vpn-tunnel-mode-replaced-with-ipsec-vpn)
- [SSL VPN removed from 2GB RAM models (7.6.0 release notes)](https://docs.fortinet.com/document/fortigate/7.6.0/fortios-release-notes/877104/ssl-vpn-removed-from-2gb-ram-models-for-tunnel-and-web-mode)

Verified against vendor documentation on 2026-10-20.
