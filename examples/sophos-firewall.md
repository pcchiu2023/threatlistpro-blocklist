# Sophos Firewall: Third-Party Threat Feed

Add the blocklist to Sophos Firewall as a third-party threat feed (IPv4 indicator type). This page also covers the documented way to drop traffic to the firewall's own VPN services.

## Feed URLs (Sophos format)

Sophos needs one bare IPv4 address per line, so ThreatListPro serves a Sophos-format copy of the list.
Use these URLs, not `blocklist.txt`:

```text
# Free sample, Sophos format (no signup)
https://api.threatlistpro.com/v1/sample/sophos.txt

# Full feed, Sophos format (subscribers)
https://api.threatlistpro.com/v1/YOUR_API_KEY/sophos.txt
```

## About the list, and two limits that matter on Sophos

- 200-entry sample; the full feed covers 5.4M+ IPv4 addresses. Built from attacks observed on live production Palo Alto GlobalProtect VPN portals in the US.
- IPv4 only. One IPv4 address or IPv4 CIDR per line. A CIDR range covers addresses that did not each attack.
- **No CIDR on Sophos:** Sophos documents that third-party threat feeds do not support IP address ranges, IPv6 addresses or network addresses. The `sophos.txt` URLs list bare addresses only: single IPs, plus ranges of /22 or smaller expanded into their host addresses. Larger ranges are left out. They hold most of the 5.4M+ addresses, so on Sophos the feed blocks only a small share of them. Step 4 covers the larger ranges.
- **No comment lines:** the feed file must be plain text with one indicator per line, so `sophos.txt` has no `#` header.
- **Traffic direction:** on SFOS 21.0 and later, the [Active threat response](https://docs.sophos.com/nsg/sophos-firewall/21.5/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/index.html) page lists system-destined traffic to services under Administration > Device access (including the VPN portal and VPN) as matched against third-party feed source IPs.
- Third-party threat feeds need the **Xstream Protection** bundle license.
- The subscriber URL contains your API key. Treat it as a secret.

## 1. Add the threat feed

1. Go to **Active threat response > Third-party threat feeds** and click **Add**.
2. **Name**, and an optional **Description**.
3. **Action**: start with **Monitor** ("Only logs threats"). Switch to **Block** ("Logs and blocks threats") after reviewing the logs.
4. Position: **Top** or **Bottom** of the list. The firewall evaluates blocked and monitored feeds in the order shown and logs the first match.
5. **Indicator type**: **IPv4 address**. The firewall evaluates only the type you select.
6. **External URL**: one of the feed URLs above.
7. **Authorization**: **No authentication**. The subscriber URL carries the key in its path.
8. Select **Validate server certificate**. For a public certificate, the issuing CA must be listed under **Certificates > Certificate authorities**.
9. **Polling interval**: the API reference (SFOS 21.0) lists 5 min, 15 min, 30 min, 1 h, 6 h, 24 h, 7 days and 30 days. XGS 87/87w, 88/88w and 107/107w support only 24 hours, 7 days and 30 days. Choose 1 h, or 24 h on those models.
10. Click **Test connection**, then **Save**.

## 2. Turn on logging

Go to **System services > Log settings**, select **Active threat response** for local reporting (and syslog or central reporting if you use them), and click **Apply**.

## 3. Verify

- **Log viewer**, then select **Active threat response**, shows blocked indicators of compromise (IoCs).
- The Control Center **Active threat response** widget shows configured third-party feeds, sync status and the number of blocked threats.
- If an expected address is not acted on, check the threat exclusions list.

## 4. Drop traffic to the firewall's VPN services

Sophos documents traffic to the firewall's own services under **Administration > Device access**, using a **Local service ACL exception rule** whose action can be **Drop**:

1. Go to **Administration > Device access** and click **Add** under **Local service ACL exception rule**.
2. **Name**, **Rule position**, optional **Description**. **IP version**: IPv4. **Source zone**: WAN.
3. **Source network/host**: Network or IP range host objects for the larger ranges in the standard `blocklist.txt` that `sophos.txt` leaves out. These objects do not update from a URL, so maintain them yourself. List the ranges with `grep -v '^#' blocklist.txt | awk -F/ '$2 < 22'`.
4. **Destination host** (optional): the IP address or interface the service listens on.
5. **Services**: **SSL VPN**, **VPN portal** and/or **IPsec**, matching what you expose.
6. **Action**: **Drop**. **Save**.

The Sophos documentation describes these source objects as host objects (IP address, IP range, Network, IP list and others). It does not describe filling them from a URL, so this rule does not update itself when the feed changes.

## Capacity

- Sophos states that a feed can have any number of IoCs, but the disk space available for feeds depends on the appliance model.
- After restoring a backup, feed configurations are restored but their lists are not; the firewall polls the source again.

## Vendor docs

- [Third-party threat feeds (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/ConfigureFeeds/ThirdPartyThreatFeeds/index.html)
- [Configure third-party threat feeds (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/ConfigureFeeds/ThirdPartyThreatFeeds/ConfigureThirdPartyThreatFeeds/index.html)
- [Add/Edit Third-party threat feed, API (SFOS 21.0, polling interval values)](https://docs.sophos.com/nsg/sophos-firewall/21.0/api/protect/active%20threat%20response/thirdpartyfeeds/operations/AddThird-partythreatfeed&EditThird-partythreatfeed.html)
- [Firewall configurations for threat feeds (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/Requirements/OtherConfigurations/index.html)
- [Licenses for threat feed modules (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/Requirements/Licenses/index.html)
- [Logs and alerts for Active threat response (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/Requirements/ConfigureLogs/index.html)
- [Active threat response widget (SFOS 21.5)](https://docs.sophos.com/nsg/sophos-firewall/21.5/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ControlCenter/ControlCenterATR/index.html)
- [Active threat response FAQ (SFOS 21.0)](https://docs.sophos.com/nsg/sophos-firewall/21.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/FAQ/index.html)
- [Active threat response, system-destined traffic (SFOS 21.5)](https://docs.sophos.com/nsg/sophos-firewall/21.5/Help/en-us/webhelp/onlinehelp/AdministratorHelp/ActiveThreatResponse/index.html)
- [Add local service ACL exception rule (SFOS 22.0)](https://docs.sophos.com/nsg/sophos-firewall/22.0/Help/en-us/webhelp/onlinehelp/AdministratorHelp/Administration/DeviceAccess/AdministrationLocalServiceACLExceptionRuleAdd/index.html)

Verified against vendor documentation on 2026-10-21.
