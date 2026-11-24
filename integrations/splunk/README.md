# Splunk IP Enrichment with IPGeolocation.io: Geolocation, VPN Detection and Threat Scores

## Overview

The **IPGeolocation.io App for Splunk** adds [IP geolocation](https://ipgeolocation.io/guides/what-is-ip-geolocation-how-it-works) and [IP security data](https://ipgeolocation.io/guides/what-is-ip-threat-intelligence) to your Splunk searches. Pipe any field that holds an IP address through one SPL command, and each event gets the country, city, [ISP](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn), [company](https://ipgeolocation.io/guides/who-owns-an-ip-address), [ASN](https://ipgeolocation.io/guides/what-is-an-asn), time zone and [abuse contact](https://ipgeolocation.io/ip-abuse-contact-api.html) of that IP, plus security data: [VPN](https://ipgeolocation.io/blog/what-is-vpn-detection-and-how-does-it-work), [proxy](https://ipgeolocation.io/guides/types-of-proxies), [residential proxy](https://ipgeolocation.io/blog/how-residential-proxy-detection-works) and Tor detection, a [threat score](https://ipgeolocation.io/blog/how-to-use-ip-threat-score) and a known attacker flag.

Version 4.2.0 also gives you a **Security Overview dashboard** and four ready-made **security searches** that flag logins from VPNs, proxies, Tor and high-risk IPs, accounts that log in from several countries, and outbound connections to risky IPs.

The app ships four custom SPL commands (`ipgeolocation`, `ipgeolocationbatch`, `ipsecurity`, `ipsecuritybatch`) and two lookup methods:

- **API mode** queries the [IPGeolocation.io IP Location API](https://ipgeolocation.io/documentation/ip-location-api.html) and [IP Security API](https://ipgeolocation.io/documentation/ip-security-api.html) over HTTPS. Each lookup uses API credits.
- **MMDB mode** reads IPGeolocation.io databases (`.mmdb` files) that the app downloads to your Splunk server. Lookups make no network call and use no API credits. You can use the [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html), [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html), [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP Security Database](https://ipgeolocation.io/ip-security-database.html), [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and the combined databases listed in [Database reference](#database-reference).

Both methods are set up on one setup page inside Splunk. No scripting is needed. Get the app from [Splunkbase](https://splunkbase.splunk.com/app/7421).

> [!TIP]
> Use API mode to start quickly or to look up a few IPs during an investigation. Use MMDB mode for scheduled searches, dashboards and large volumes: lookups are local and cost no API credits.

---

## What's New in 4.2.0

- **Security Overview dashboard**: logins and outbound connections from risky IPs, with counts, charts, a map and tables. See [Dashboards](#dashboards).
- **Four security searches** you can schedule as alerts, and macros that point them at your own data. See [Security Searches and Alerts](#security-searches-and-alerts).
- `ipsecurity` and `ipsecuritybatch` now work in **MMDB mode** with an IP Security, Residential Proxy or IP Hosting database.
- Several databases can now be enabled together: their fields are combined in one result.
- API mode returns 11 more security fields: proxy activity (days seen in the last 7 and 30 days, provider count), bot and corporate gateway fields.
- MMDB mode reads the new IP Security database fields (bot, corporate gateway, proxy activity) and the new provider list of the Residential Proxy database.
- Database refresh now needs the admin role, and keys no longer appear in log files.
- Removed from the setup page: **Download MMDB on each Search Head**, **Bearer Token** and **API URL**. Each database now uses only its own key.

Upgrading from 4.0 or 4.1? See [Upgrade from 4.0 or 4.1](#upgrade-from-40-or-41).

---

## Supported Splunk Version

| Splunk |
| --- |
| Splunk 10.7 |
| Splunk 10.6 |
| Splunk 10.5 |
| Splunk 10.4 |
| Splunk 10.3 |
| Splunk 10.2 |
| Splunk 10.1 |
| Splunk 10.0 |
| Splunk 9.4 |
| Splunk 9.3 |
| Splunk 9.2 |
| Splunk 9.1 |
| Splunk 9.0 |


## Prerequisites

Before installing the app, make sure you have the following:

- A running **Splunk Enterprise** or **Splunk Cloud** instance on a version listed in the table above.
- Use an app release tested for your Splunk version. Run `splunk cmd python3 --version` to check the Python runtime your Splunk instance uses.
- An active **IPGeolocation.io account**. [Sign up free](https://app.ipgeolocation.io/signup) -- no credit card required for the Free plan.
- For **API mode**: your **API key** from the [IPGeolocation.io dashboard](https://app.ipgeolocation.io). Security data needs a Paid API plan.
- For **MMDB mode**: a database subscription and the **database key** for each database, from the Databases section of your IPGeolocation.io account.

---

## Installation

Install the app on your **search heads**. Forwarders don't need it. You can install it from Splunk Web or from the command line.

### Splunk Web (Standalone)

1. On the Splunk home page, click **Apps > Manage Apps**.

![Manage Apps in the Splunk Apps menu](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/manage_apps.png)

2. Click **Install app from file**.

![Install app from file button on the Splunk Apps page](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/install_app_from_file.png)

3. Choose the IPGeolocation.io app package you downloaded from Splunkbase and click **Upload**.

![Install App From File dialog with the IPGeolocation.io app package selected](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/upload_app.png)

4. Restart Splunk when it asks you to.

### Command Line (Standalone)

1. Extract the package you downloaded from Splunkbase: `tar -xzf <package>.tgz`. It contains the `ipgeolocation_app` folder.
2. Copy the `ipgeolocation_app` folder to `$SPLUNK_HOME/etc/apps/`.
3. Restart Splunk: `$SPLUNK_HOME/bin/splunk restart`.

### Deployment Server (Non-Clustered Search Heads)

Use this for one or more search heads that are not in a cluster.

1. Extract the package and copy the `ipgeolocation_app` folder to `$SPLUNK_HOME/etc/deployment-apps/` on the deployment server.
2. Add the app to the server class of your search heads in `serverclass.conf`:

    ```
    [serverClass:<SEARCHHEAD_SERVERCLASS>:app:ipgeolocation_app]
    stateOnClient = enabled
    restartSplunkd = true
    ```

3. Reload the deployment server: `$SPLUNK_HOME/bin/splunk reload deploy-server`.

### Search Head Cluster (Deployer)

1. Extract the package and copy the `ipgeolocation_app` folder to `$SPLUNK_HOME/etc/shcluster/apps/` on the deployer.
2. Push it to the cluster members:

    ```
    $SPLUNK_HOME/bin/splunk apply shcluster-bundle -target <URI>:<management_port> -auth <username>:<password>
    ```

---

## Upgrade from 4.0 or 4.1

Upgrading keeps your settings, stored keys, downloaded databases and schedules.

**Splunk Web:** go to **Apps > Manage Apps > Install app from file**, choose the new package, tick **Upgrade app**, click **Upload** and restart Splunk.

![Install App From File dialog with the 4.2.0 package selected and Upgrade app ticked](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/upgrade_app_v420.png)

**Command line:**

```
$SPLUNK_HOME/bin/splunk install app <package>.tgz -update 1 -auth <username>:<password>
$SPLUNK_HOME/bin/splunk restart
```

**Search head cluster:** replace the `ipgeolocation_app` folder in `$SPLUNK_HOME/etc/shcluster/apps/` on the deployer and run `splunk apply shcluster-bundle` again.

After the upgrade there is nothing to change: the removed setup fields (Download MMDB on each Search Head, Bearer Token, API URL) were not needed anymore. To use the new security searches, follow [Security Searches and Alerts](#security-searches-and-alerts).

---

## Configuration

1. Log in to Splunk Web and go to **Apps > Manage Apps**.
2. Find **ipgeolocation.io** and click **Set up**.

![Splunk Apps page filtered to ipgeolocation.io 4.2.0, with the Set up link highlighted](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/manage_setup_v420.png)

3. Choose a lookup method, fill in its settings as described below and click **Submit**.

### API Mode

1. Under **Select Lookup Method**, choose **Fetch Details via REST API**.
2. Paste your API key in **Key for API Subscription**.
3. Choose your **API Subscription Plan**: **Paid** or **Free**.

![Setup page with REST API selected, the API key field and the Paid plan selected](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/setup_api_v420.png)

> [!IMPORTANT]
> The app requests data based on the plan you choose here, not on your key. If you have a Paid key but choose **Free**, you get Free plan data only: no security data, and `ipsecurity` returns nothing. Choose the plan that matches your key.

### Proxy Settings

All proxy fields are optional. To send the app's requests to IPGeolocation.io through a proxy, set **Proxy Enable** to **Yes** and fill in the proxy type (`http` or `https`), host, port and, if your proxy needs them, username and password. The proxy is used for API lookups and database downloads.

![Proxy settings on the setup page](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/proxy_setting.png)

### MMDB Mode

1. Under **Select Lookup Method**, choose **Use MMDB**.
2. For each database you subscribe to:
    - set **Enable** to **Yes**,
    - paste the database key in **Key**,
    - choose the **Update Interval** of your subscription: **Daily** or **Weekly**.
3. Click **Submit**.

For example, a security setup with the IP Security, Residential Proxy and IP Hosting databases:

![Setup page with the IP Security, Residential Proxy and IP Hosting databases enabled, each with a key and a daily update interval](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/setup_mmdb_security_v420.png)

The app downloads each enabled database on its schedule: **Daily** at 01:01 every day, **Weekly** at 01:01 every Monday (Splunk server time). To download right away, see [Database Downloads](#database-downloads).

You can enable several databases at once, for example IP to Location, IP to ASN and IP Security. The app combines their fields in one result. Some databases save to the same file, so enable only one database of each group in [Databases That Share a File](#databases-that-share-a-file).

#### Replicate MMDB on Indexers

Set **Replicate MMDB on Indexers** to **Yes** to send the database files to your indexers in the search bundle, so `ipgeolocation` and `ipsecurity` run on the indexers. This can make large searches faster, but it makes the search bundle bigger. It applies when you search a distributed deployment with indexers.

> [!NOTE]
> The databases are saved in the `lookups` folder of the app: `$SPLUNK_HOME/etc/apps/ipgeolocation_app/lookups`. They don't replace Splunk's own GeoIP database used by the `iplocation` command.

> [!NOTE]
> In a search head cluster, every search head downloads its own copy of each enabled database.

### Switching Between API and MMDB

When you choose **Fetch Details via REST API** and click **Submit**, the app switches off all database settings and their scheduled downloads. Your downloaded files and database keys are kept. When you switch back to **Use MMDB**, set **Enable** to **Yes** again for each database.

---

## Database Reference

The **Database ID** is the name you use with `refreshsinglemmdb`. The files are saved in the app's `lookups` folder.

| Name on the setup page | Database ID | MMDB file |
|---|---|---|
| Standard: IP to Country | `db_std_ip_country` | `db-ip-country.mmdb` |
| Standard: IP to Location | `db_std_ip_city` | `db-ip-location.mmdb` |
| Standard: IP to ISP | `db_std_ip_isp` | `db-ip-isp.mmdb` |
| Standard: IP to City & ISP | `db_std_ip_city_isp` | `db-ip-city-isp.mmdb` |
| Advance: IP to Location | `db_advanced_ip_city` | `db-ip-location.mmdb` |
| Advance: IP Abuse Contact | `db_advanced_ip_abuse` | `db-ip-abuse.mmdb` |
| Advance: IP to ASN | `db_advanced_ip_asn` | `db-ip-asn.mmdb` |
| Advance: IP to ASN (Extended) | `db_advanced_ip_asn_ext` | `db-ip-asn.mmdb` |
| Advance: IP to Company | `db_advanced_ip_company` | `db-ip-company.mmdb` |
| Advance: IP WHOIS | `db_advanced_ip_whois` | `db-ip-whois.mmdb` |
| Advance: IP Location, Company & ASN | `db_advanced_ip_city_company_asn` | `db-ip-city-company-asn.mmdb` |
| Advance: IP Location, Company, ASN & Abuse Contact | `db_advanced_ip_city_company_asn_abuse` | `db-ip-city-company-asn-abuse.mmdb` |
| Advance: IP to ASN & Company | `db_advanced_ip_company_asn` | `db-ip-company-asn.mmdb` |
| Advance: IP to City, Company, ASN, Abuse, Security | `db_advanced_ip_city_company_asn_abuse_security` | `db-ip-city-company-asn-abuse.mmdb` + `db-ip-security.mmdb` |
| Security Pro: IP Security | `db_sec_pro_ip_security` | `db-ip-security.mmdb` |
| Security Pro: IP Residential Proxy | `db_sec_pro_ip_residential_proxy` | `db-residential-proxy.mmdb` |
| Security Pro: IP Hosting Provider | `db_sec_pro_ip_hosting` | `db-ip-hosting.mmdb` |
| Security Pro: IP to Security & Location | `db_sec_pro_ip_city_security` | `db-ip-city.mmdb` + `db-ip-security.mmdb` |
| Security Pro: IP to Security, Location & ISP | `db_sec_pro_ip_city_isp_security` | `db-ip-city-isp.mmdb` + `db-ip-security.mmdb` |

> IP WHOIS: the standard IP WHOIS download is Gzip-compressed NDJSON. To use it in this app, request the IP WHOIS MMDB format from support first.

### Databases That Share a File

These databases save to the same file name. If you enable two of the same group, each download overwrites the other one's file, and your results change with every download. Enable only one database from each group:

| File | Enable only one of |
|---|---|
| `db-ip-location.mmdb` | Standard: IP to Location, Advance: IP to Location |
| `db-ip-asn.mmdb` | Advance: IP to ASN, Advance: IP to ASN (Extended) |
| `db-ip-city-isp.mmdb` | Standard: IP to City & ISP, Security Pro: IP to Security, Location & ISP |
| `db-ip-city-company-asn-abuse.mmdb` | Advance: IP Location, Company, ASN & Abuse Contact, Advance: IP to City, Company, ASN, Abuse, Security |
| `db-ip-security.mmdb` | Security Pro: IP Security, Security Pro: IP to Security & Location, Security Pro: IP to Security, Location & ISP, Advance: IP to City, Company, ASN, Abuse, Security |

---

## Database Downloads

Enabled databases download on their schedule (see [MMDB Mode](#mmdb-mode)). The first download happens at the next scheduled time, so after the setup, download them once by hand.

**Force Refresh page:** in the app menu, open **Force Refresh**, set **Yes** for each database to download now and submit.

**Search bar:** run `refreshsinglemmdb` with the [database ID](#database-reference):

```spl
| refreshsinglemmdb MMDB="db_sec_pro_ip_security"
```

Run it once for each database you want to refresh. Both ways need the Splunk **admin** role; other users get the message "Only admins can update the databases."

To change the download time, go to **Settings > Data inputs > Scripts**, open the app's script for that database (for example `db_sec_pro_ip_security_mmdb.py`) and change its interval (a cron schedule). Clicking **Submit** on the setup page sets the schedule back to Daily or Weekly at 01:01.

---

## Usage

### `ipgeolocation` Command

**Type:** Streaming (adds fields to each event)  
**Methods:** API (Free and Paid), MMDB  
**Plan:** Free API plan, or any database

The `ipgeolocation` command is the main lookup command. Pipe your search results through it and name one or more fields that hold IP addresses. Each event gets the location, ISP, ASN, time zone and other fields of its IP.

**Syntax:**

```spl
... | ipgeolocation <field> [field2 ...] [options]
```

**Options:**

| Option | Type | Default | Description |
|---|---|---|---|
| `liveHostname` | bool | `false` | Perform a live reverse DNS lookup for the IP. PAID API only. |
| `hostnameFallbackLive` | bool | `false` | Try hostname from database first; fall back to live DNS if not found. PAID API only. |
| `abuseContact` | bool | `false` | Include abuse contact details (name, email, address). PAID API only. |
| `dma` | bool | `false` | Include the DMA (Designated Market Area) code. US-centric. PAID API only. |
| `geoAccuracy` | bool | `false` | Include accuracy radius and confidence level for the geolocation. PAID API only. |
| `security` | bool | `false` | Include security data: VPN, proxy, residential proxy, Tor, bot, spam, known attacker, threat score. Needs a Paid API plan, or in MMDB mode an IP Security, Residential Proxy or IP Hosting database (or a combined database with security). |
| `allinfo` | bool | `false` | Shortcut that enables `liveHostname`, `abuseContact`, `dma`, `geoAccuracy`, and `security` simultaneously. |
| `prefix` | bool | `false` | Prefix all output field names with the source field name. Automatically set to `true` when multiple IP fields are specified. |
| `language` | string | `en` | Response language for place names. Supported: `en`, `de`, `ru`, `ja`, `fr`, `cn`, `es`, `cs`, `it`, `ko`, `fa`, `pt`, `ar`. `ar` (Arabic) works in API mode only; in MMDB mode it falls back to English with a warning. |

**Examples:**

Basic geolocation lookup on a field named `src_ip`:

```spl
index=firewall action=blocked
| ipgeolocation src_ip
| table _time, src_ip, location.city, location.country_name, isp, location.latitude, location.longitude
```

Lookup two IP fields simultaneously (auto-prefixes output fields):

```spl
index=web_proxy
| ipgeolocation src_ip dst_ip
| table src_ip_location.country_name, dst_ip_location.country_name
```

Full enrichment for a threat-hunting search:

```spl
index=sysmon EventCode=3
| rename SourceIp as src_ip
| ipgeolocation src_ip allinfo=true
| where `ipgeolocation_is_true(security.is_tor)` OR `ipgeolocation_is_true(security.is_known_attacker)`
| table _time, src_ip, location.country_name, security.is_tor, security.is_known_attacker, security.threat_score, abuse.emails
```

Localized response in French:

```spl
index=access_combined
| rex field=_raw "(?<client_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
| ipgeolocation client_ip language=fr
| table client_ip, location.country_name, location.city
```

Enrich with DMA code for US ad-targeting analysis:

```spl
index=cdn_logs country_code=US
| ipgeolocation ip dma=true
| stats count by location.dma_code, location.city
| sort -count
```

#### Security Fields in API and MMDB Mode

- **Yes/no flags** such as `security.is_vpn` come back as `1` or `0` in API mode and as `true` or `false` in MMDB mode. To write one search for both, use the app's macro: `` where `ipgeolocation_is_true(security.is_vpn)` ``. The macro is shared with all apps.
- In MMDB mode, `ipgeolocation` returns six proxy and VPN detail fields without the `security.` prefix: `proxy_confidence_score`, `proxy_last_seen`, `proxy_provider_names`, `vpn_confidence_score`, `vpn_last_seen` and `vpn_provider_names`.
- API mode also returns proxy activity (`security.proxy_days_seen_7d`, `security.proxy_days_seen_30d`, `security.proxy_provider_count_30d`), bot fields (`security.is_known_good_bot`, `security.bot_type`, `security.bot_operator_name`, `security.bot_confidence_score`, `security.bot_last_seen`) and corporate gateway fields (`security.is_corporate_gateway`, `security.corporate_gateway_type`, `security.corporate_gateway_provider_name`). In MMDB mode these come from the IP Security database.

For the full list of database fields, see the [MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md).

---

### `ipgeolocationbatch` Command

**Type:** Generating (produces its own results, does not require piped input)  
**Methods:** API (Free and Paid), MMDB  
**Plan:** Free API plan, or any database

The `ipgeolocationbatch` command is used when you have a static list of IPs and want to enrich them without a preceding search that generates records. Start your SPL with a `|` (pipe) to invoke a generating command.

**Syntax:**

```spl
| ipgeolocationbatch ips="<ip1>,<ip2>,..." [options]
```

**Options:** Identical to `ipgeolocation` (`liveHostname`, `hostnameFallbackLive`, `abuseContact`, `dma`, `geoAccuracy`, `security`, `allinfo`, `language`).

**The `ips` parameter is required.** Pass a comma-separated list of IP addresses.

**Examples:**

Quick ad-hoc lookup for a known list of suspicious IPs from a threat intel report:

```spl
| ipgeolocationbatch ips="185.220.101.1, 162.247.72.199, 198.98.54.11, 45.142.212.100" security=true
| table ip, location.country_name, location.city, isp, security.is_tor, security.is_proxy
```

Security data in API mode requires a paid API plan.

Combine with a lookup table of IOCs:

```spl
| inputlookup threat_iocs.csv
| fields ip_address
| ipgeolocation ip_address security=true
| table ip, location.country_name, security.is_known_attacker, security.threat_score
```

Generate a geolocation report for a list of public DNS resolvers:

```spl
| ipgeolocationbatch ips="8.8.8.8, 8.8.4.4, 1.1.1.1, 208.67.222.222" language=de
| table ip, location.country_name, location.city, asn.as_number, asn.organization
```

---

### `ipsecurity` Command

**Type:** Streaming  
**Methods:** API (Paid), MMDB  
**Plan:** Paid API plan, or an IP Security, Residential Proxy or IP Hosting database

The `ipsecurity` command returns only IP security data: VPN, proxy, residential proxy, Tor, bot, spam and cloud provider flags, and a threat score from 0 to 100. Use it when you want security signals without the full geolocation data.

In MMDB mode it reads the enabled IP Security, Residential Proxy and IP Hosting databases. If none is enabled, it shows the warning "No IP Security, Residential Proxy or IP Hosting database is enabled".

**Syntax:**

```spl
... | ipsecurity <field> [field2 ...] [prefix=true]
```

**Options:**

| Option | Type | Default | Description |
|---|---|---|---|
| `prefix` | bool | `false` | Prefix output field names with the source field name. Auto-enabled when multiple IP fields are provided. |

**Examples:**

Identify VPN/proxy users in your authentication logs:

```spl
index=auth sourcetype=okta action=success
| ipsecurity src_ip
| where `ipgeolocation_is_true(security.is_vpn)` OR `ipgeolocation_is_true(security.is_proxy)`
| stats count by user, src_ip, security.is_vpn, security.is_proxy
| sort -count
```

Alert on logins from Tor exit nodes:

```spl
index=auth action=login
| ipsecurity client_ip
| where `ipgeolocation_is_true(security.is_tor)`
| eval alert_message="Tor login attempt from " . client_ip
| table _time, user, client_ip, alert_message
```

Score incoming webhook requests by threat level:

```spl
index=api_gateway path="/webhook/payment"
| rex field=_raw "\"x-forwarded-for\":\"(?<req_ip>[^\"]+)\""
| ipsecurity req_ip
| where tonumber('security.threat_score') > 70
| table _time, req_ip, security.threat_score, security.is_bot, security.is_spam
```

---

### `ipsecuritybatch` Command

**Type:** Generating  
**Methods:** API (Paid), MMDB  
**Plan:** Paid API plan, or an IP Security, Residential Proxy or IP Hosting database

The `ipsecuritybatch` command lets you look up security information for a static list of IPs. Like `ipgeolocationbatch`, it is a generating command (starts with `|`).

**Syntax:**

```spl
| ipsecuritybatch ips="<ip1>,<ip2>,..."
```

**Examples:**

Check a batch of IPs from a threat intelligence feed:

```spl
| ipsecuritybatch ips="91.108.4.1, 185.220.101.45, 176.10.99.200, 194.165.16.87"
| table ip, security.is_vpn, security.is_proxy, security.is_tor, security.is_bot, security.threat_score
| sort -security.threat_score
```

Weekly automated threat scoring of your top talkers:

```spl
index=firewall earliest=-7d@d
| top limit=50 src_ip
| rename src_ip as ip
| table ip
| map maxsearches=50 search="| ipsecuritybatch ips=\"$ip$\""
| where tonumber('security.threat_score') > 50
| outputlookup high_risk_ips.csv
```

---

## Dashboards

The app has two dashboards. Open them from the app menu.

### IP Lookup Dashboard

The **ipgeolocation.io** dashboard is the first item in the app menu. Type an IP address in the box at the top to see everything about it: location and map, threat score, and panels for location, security, ASN, abuse contact, company, country, network, currency and time zone. Panels without data for that IP are hidden.

![ipgeolocation.io dashboard looking up 5.255.111.52: city, state, country, postal code, threat score 90 and a map](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/dashboard_lookup_v420.png)

The **Security** panel lists all security fields of the IP, including the proxy activity, bot and corporate gateway fields. This example is a Tor exit node and known attacker with a threat score of 90:

![Security panel of the ipgeolocation.io dashboard showing threat score, Tor, proxy, residential proxy, known attacker, bot and corporate gateway fields](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/dashboard_security_panel_v420.png)

### Security Overview Dashboard

The **Security Overview** dashboard shows which logins and outbound connections in your data come from risky IPs:

- counts of login IPs from anonymizers (Tor, VPN, residential proxy, proxy, relay), with a high threat score, from known attackers, and without security data,
- logins by anonymizer type and login IPs by threat score,
- a map of where risky logins come from,
- a table of risky login IPs with their accounts, and a table of outbound connections to risky IPs.

![Security Overview dashboard with login IP counts, anonymizer and threat score charts, a map, and tables of risky login IPs and outbound connections](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/security_overview_v420.png)

Choose the **Time range** and how many **IPs to look up** (the busiest first: 100, 1,000 or 10,000). Each time the dashboard loads, every public login IP in that range is looked up once. In API mode this uses API credits; in MMDB mode it doesn't.

The dashboard reads your data through the `ipgeolocation_auth_events` and `ipgeolocation_network_events` macros. If it stays empty, point them at your data: see [Point the Searches at Your Data](#point-the-searches-at-your-data).

> [!NOTE]
> Security data needs a Paid API plan or an enabled IP Security database. On the Free API plan, every IP is counted under **No security data**. No data does not mean the IP is safe.

---

## Security Searches and Alerts

The app ships four searches you can schedule as alerts. They are **not scheduled** after installation, so nothing runs until you turn them on.

| Search | Finds | Runs every | Default time range |
|---|---|---|---|
| IPGeolocation - Login From Anonymizer | Successful logins from Tor, VPN, residential proxy, proxy or relay IPs | Hour, at :07 | Last hour |
| IPGeolocation - Login From High Threat IP | Successful logins from IPs with a threat score at or above `ipgeolocation_threat_score_threshold` (default 80) | Hour, at :07 | Last hour |
| IPGeolocation - Connections To Risky IPs | Network connections to known attackers, Tor, or IPs with a high threat score | Hour, at :07 | Last hour |
| IPGeolocation - Account Logins From Several Countries | Accounts with successful logins from at least `ipgeolocation_min_countries` countries (default 2) | Hour, at :17 | Last 24 hours |

Each search alerts when it finds results, and lists the alert under **Activity > Triggered Alerts**. The same IP (or the same account, for the countries search) alerts at most once every 24 hours.

The searches use the same lookup method as the rest of the app. In API mode each IP they look up uses API credits.

**To turn a search on:**

1. Set the macros for your data first (next section).
2. Go to **Settings > Searches, reports, and alerts**, choose the **ipgeolocation.io** app and set the owner filter to **All**.
3. Next to the search, click **Edit > Edit Schedule**, tick **Schedule Report** and click **Save**.
4. Optional: add a trigger action such as email or webhook in the alert's settings.

### Point the Searches at Your Data

The searches and the Security Overview dashboard use these macros. Change them in **Settings > Advanced search > Search macros** (choose the **ipgeolocation.io** app).

| Macro | Default | What to set |
|---|---|---|
| `ipgeolocation_auth_events` | `index=* tag=authentication action=success` | Your successful logins. The events need the `src` (source IP) and `user` fields. |
| `ipgeolocation_network_events` | `index=* tag=network tag=communicate` | Your firewall or proxy connections. The events need `dest` (remote IP) and `src` (internal host). |
| `ipgeolocation_threat_score_threshold` | `80` | Lowest threat score that counts as high. IPGeolocation.io suggests 80-100 for blocking or manual review, and 45-79 for added friction. |
| `ipgeolocation_min_countries` | `2` | Number of countries one account must log in from to be listed. |

The defaults use the Splunk [Common Information Model (CIM)](https://help.splunk.com/en/splunk-enterprise/common-information-model) tags, which most Splunk add-ons for VPNs, identity providers and firewalls set. Replace `index=*` with your own indexes so the searches run faster.

![Search macros page filtered to the ipgeolocation.io app, with the four macros you can change highlighted](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/search_macros_v420.png)

For example, if your VPN logins are in the `vpn` index:

![Editing the ipgeolocation_auth_events macro with the definition index=vpn tag=authentication action=success](https://static.ipgeolocation.io/web-assets/images/integrations/splunk/macro_edit_auth_events_v420.png)

The app also has three helper macros you don't need to change: `ipgeolocation_is_true(1)`, `ipgeolocation_is_public_ip(1)` and `ipgeolocation_anonymizer_types`. They are shared with all apps, so you can use them in your own searches:

```spl
index=vpn action=success
| where `ipgeolocation_is_public_ip(src)`
| ipgeolocation src security=true
| `ipgeolocation_anonymizer_types`
| where isnotnull(anonymizer)
| stats count by user, src, anonymizer
```

---

## Practical SPL Examples

### Use Case 1: Geo-Enrich Firewall Blocked Traffic for a Map Dashboard

```spl
index=firewall action=blocked earliest=-1h
| stats count as blocked_events by src_ip
| ipgeolocation src_ip
| geostats latfield=location.latitude longfield=location.longitude sum(blocked_events) by location.country_name
```

This feeds directly into Splunk's Cluster Map visualization. The result shows blocked connection volume by geographic origin.

### Use Case 2: Detect Impossible Travel (Auth Log Analysis)

```spl
index=auth action=success
| sort 0 _time
| ipgeolocation src_ip
| streamstats window=2 current=true global=false values(location.country_name) as countries values(src_ip) as ips range(_time) as time_diff_secs by user
| where mvcount(countries) > 1 AND time_diff_secs < 3600
| eval alert="Impossible travel: " . mvjoin(countries, " -> ") . " within " . round(time_diff_secs/60, 0) . " minutes"
| table _time, user, ips, countries, time_diff_secs, alert
```

### Use Case 3: Identify High-Risk API Consumers

```spl
index=api_gateway method=POST uri="/api/v1/checkout" status=200 earliest=-24h
| stats count as requests, sum(response_bytes) as bytes_transferred by src_ip
| ipsecurity src_ip
| where tonumber('security.threat_score') > 60 OR `ipgeolocation_is_true(security.is_proxy)` OR `ipgeolocation_is_true(security.is_vpn)`
| table src_ip, requests, bytes_transferred, security.threat_score, security.is_vpn, security.is_proxy, security.is_tor
| sort -security.threat_score
```

### Use Case 4: Country-Based Access Control Audit

```spl
index=web_access
| ipgeolocation clientip
| search NOT location.country_code2 IN ("US", "CA", "GB", "AU", "DE")
| stats count by location.country_code2, location.country_name, clientip
| sort -count
| head 50
```

### Use Case 5: ISP-Aware Rate Limit Breach Detection

```spl
index=nginx status=429 earliest=-15m
| ipgeolocation client_ip
| stats count as rate_limit_hits by client_ip, isp, asn.as_number, location.country_name
| where rate_limit_hits > 100
| table client_ip, isp, asn.as_number, location.country_name, rate_limit_hits
```

### Use Case 6: Enrich Threat Intel IOC List with Full Profile

```spl
| inputlookup threat_intel_ips.csv where is_active=true
| rename ip_indicator as ip
| ipgeolocation ip allinfo=true
| eval risk_label=case(
    `ipgeolocation_is_true(security.is_known_attacker)`, "High - Known Attacker",
    `ipgeolocation_is_true(security.is_tor)`, "High - Tor",
    tonumber('security.threat_score') > 70, "High - Threat Score",
    `ipgeolocation_is_true(security.is_vpn)` OR `ipgeolocation_is_true(security.is_proxy)`, "Medium - Anonymizer",
    true(), "Low"
  )
| table ip, location.country_name, isp, asn.as_number, security.threat_score, risk_label, abuse.emails
| sort risk_label
```

### Use Case 7: Timezone-Aware Business Hours Alerting

```spl
index=auth action=success earliest=-1h
| ipgeolocation src_ip
| eval local_hour=floor(((_time + ('timezone.offset_with_dst' * 3600)) % 86400) / 3600)
| where isnotnull(local_hour) AND (local_hour < 6 OR local_hour > 22)
| eval after_hours_login="Login at local time " . local_hour . ":00 in " . 'timezone.name'
| table _time, user, src_ip, location.city, location.country_name, timezone.name, after_hours_login
```

The local hour is calculated from each event's `_time`, so the result does not depend on when the search runs or on your Splunk user time zone. `timezone.offset_with_dst` is the location's UTC offset in hours at lookup time, which is correct for recent events like the last hour searched here. For older events that span a daylight-saving change, use the offset for the event's date. The offset is returned in API mode; MMDB lookups return `timezone.name` only.

---

## Troubleshooting

### Check the App Log

The app writes detailed logs to:

```
$SPLUNK_HOME/var/log/splunk/ipgeolocation.log
```

You can search this log from Splunk itself:

```spl
index=_internal source=*ipgeolocation.log
| table _time, log_level, message
| sort -_time
```

Common log entries include API request counts, MMDB file paths opened, errors during parsing, and API usage stats tracked before and after each command run.

### "Your subscription plan must be..." Warning

You see a warning like:

```
Your subscription plan must be 'PAID' to search IP details through `ipsecurity` command.
```

**Cause:** In API mode, `ipsecurity` and `ipsecuritybatch` need the Paid plan, and so do the `allinfo`, `security`, `abuseContact`, `geoAccuracy`, `dma` and `liveHostname` options. The app checks the **API Subscription Plan** chosen on the setup page, not your key.

**Fix:** Log in to [app.ipgeolocation.io](https://app.ipgeolocation.io), confirm your plan and choose the same plan on the setup page. With **Free** chosen, `ipsecurity` passes your events through unchanged.

### "No IP Security, Residential Proxy or IP Hosting database is enabled"

**Cause:** The lookup method is MMDB and you run `ipsecurity` or `ipsecuritybatch`, but none of these databases is enabled.

**Fix:** Enable an IP Security, Residential Proxy or IP Hosting database on the setup page and download it (see [Database Downloads](#database-downloads)), or switch to API mode with a Paid plan.

### "Only admins can update the databases."

**Cause:** `refreshsinglemmdb` and the Force Refresh page need the Splunk **admin** role.

**Fix:** Run the refresh as an admin, or let the scheduled downloads update the databases.

### Security Overview Shows Everything Under "No security data"

**Cause:** There is no security data: the API plan on the setup page is **Free**, or in MMDB mode no database with IP Security is enabled.

**Fix:** Choose the Paid plan (with a Paid key), or enable and download an IP Security database.

### Security Overview or Security Searches Show No Results

**Cause:** The macros don't match your data, or the time range has no events.

**Fix:** Run `` `ipgeolocation_auth_events` | head 10 `` in the search bar. If it returns nothing, or the events have no `src` and `user` fields, change the macro (see [Point the Searches at Your Data](#point-the-searches-at-your-data)).

### No Fields Returned / All Fields Empty

**Cause:** Common causes include an invalid or expired API key, the IP field value being empty or whitespace, or a network error reaching the API endpoint.

**Fix:**
1. Confirm your API key is valid by testing it directly: `curl "https://api.ipgeolocation.io/v3/ipgeo?apiKey=YOUR_KEY&ip=8.8.8.8"`
2. Inspect `ipgeolocation.log` for `Error during fetching data` messages.
3. Ensure the search field actually contains a valid IP string. Run `| table <your_ip_field>` before the `ipgeolocation` command to verify.

### MMDB File Not Found

**Cause:** The database has not been downloaded yet, or the download failed.

**Fix:** Download it now (as an admin):

```spl
| refreshsinglemmdb MMDB="<database_id>"
```

Check `ipgeolocation.log` for download errors. Check that the database key on the setup page is correct and that `database.ipgeolocation.io` is reachable from the Splunk server.

---

## Support and Resources

| Resource | Link |
|---|---|
| IPGeolocation.io App on Splunkbase | https://splunkbase.splunk.com/app/7421 |
| IPGeolocation.io Homepage | https://ipgeolocation.io |
| API Documentation | https://ipgeolocation.io/documentation/ip-location-api.html |
| IP Security API Docs | https://ipgeolocation.io/documentation/ip-security-api.html |
| Database Pricing | https://ipgeolocation.io/db-pricing.html |
| API Pricing | https://ipgeolocation.io/pricing.html |
| Account Dashboard | https://app.ipgeolocation.io |
| All Integrations | https://ipgeolocation.io/integrations.html |
| Contact Support | https://ipgeolocation.io/contact.html |
