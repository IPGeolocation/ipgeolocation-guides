# IPGeolocation.io CloudQuery Source Plugin

## Overview

Load IP geolocation, threat intelligence, ASN ownership and abuse contact data straight into your data warehouse. The IPGeolocation.io source plugin for [CloudQuery](https://cloudquery.io) turns the [IPGeolocation.io v3 API](https://ipgeolocation.io/documentation.html) into a set of ordinary database tables, so you can join IP intelligence against your own logs, inventories and access records using plain SQL.

There is no pipeline code to write. You list the IP addresses, AS numbers or user agent strings you care about in a YAML file, run one command, and the data appears in PostgreSQL, BigQuery, Snowflake, SQLite, S3 or any other CloudQuery destination.

---

## What is CloudQuery?

[CloudQuery](https://cloudquery.io) is an open source data movement tool. It reads from APIs through source plugins and writes to databases and object stores through destination plugins, handling schema creation, batching, retries and incremental state along the way.

The split matters in practice: the same source plugin feeds SQLite on a laptop and Snowflake in production, and switching between them is a change to one block of YAML.

This makes CloudQuery a good fit for teams that want IP intelligence sitting next to their existing data rather than behind an API call:

- Security engineers building enrichment tables their SIEM can join against
- Data teams who need IP attribution in the warehouse alongside product analytics
- Compliance teams producing repeatable, auditable geographic reports
- Network and platform teams tracking ASN ownership across their address space

---

## What Does This Plugin Do?

It exposes five tables mapped to the IPGeolocation.io v3 endpoints.

| Table | API Endpoint | What It Returns |
|---|---|---|
| `ipgeolocation_ip_geolocation` | `/v3/ipgeo` | Country, state, city, coordinates, timezone, currency, ASN, company, network and CDN data, with optional security, abuse, accuracy, DMA code and hostname |
| `ipgeolocation_ip_security` | `/v3/security` | Threat score, VPN, proxy, residential proxy, Tor, relay, bot, spam, cloud and corporate gateway detection |
| `ipgeolocation_abuse_contact` | `/v3/abuse` | Abuse team name, emails, phone numbers and postal address for the network block |
| `ipgeolocation_asn` | `/v3/asn` | AS name, organization, RIR, allocation status, route counts, with peers, upstreams, downstreams, routes and WHOIS on request |
| `ipgeolocation_user_agent` | `/v3/user-agent` | Browser, version, device, brand, rendering engine and operating system |

Nested API responses are flattened into columns, so `security.vpn_provider_names` becomes `vpn_provider_names` and you never write a JSON path in a query. Array values are stored as comma separated text, which every destination handles the same way.

---

## Prerequisites

- [CloudQuery CLI](https://docs.cloudquery.io/docs/quickstart) v5 or later
- [Go](https://go.dev/dl/) 1.22.7 or later, to build the plugin binary
- An IPGeolocation.io API key. The free Developer plan allows 1,000 requests per day and covers the geolocation table. Get one from [signing up here](https://app.ipgeolocation.io/signup), no card required.
- A destination you can write to. SQLite needs nothing beyond a file path, which makes it the easiest way to try this out.

The security, abuse, ASN and user agent endpoints require a paid plan. See the [pricing page](https://ipgeolocation.io/pricing.html) for current limits.

---

## Installation

Clone the repository and build the binary:

```bash
git clone https://github.com/IPGeolocation/cq-source-ipgeolocation
cd cq-source-ipgeolocation
go build -o cq-source-ipgeolocation
```

Start the plugin. It listens on `localhost:7777` and stays in the foreground, so give it its own terminal window.

```bash
./cq-source-ipgeolocation serve
```

CloudQuery connects to that address through the `grpc` registry, which is how every example below is wired up.

---

## Configuration

A CloudQuery config is one or more YAML documents. You need a `source` block describing the plugin and a `destination` block describing where rows go.

Create `ipgeo.yml`:

```yaml
kind: source
spec:
  name: ipgeolocation
  registry: grpc
  path: "localhost:7777"
  tables: ["ipgeolocation_ip_geolocation"]
  destinations: ["sqlite"]
  spec:
    api_key: "${IPGEOLOCATION_API_KEY}"
    ips:
      - "8.8.8.8"
      - "1.1.1.1"
---
kind: destination
spec:
  name: sqlite
  path: cloudquery/sqlite
  version: "v2.9.3"
  spec:
    connection_string: "./ipgeo.db"
```

Export your key and sync:

```bash
export IPGEOLOCATION_API_KEY="your_api_key_here"
cloudquery sync ipgeo.yml
```

Check the result:

```bash
sqlite3 ipgeo.db "SELECT ip, country_name, city, company_name FROM ipgeolocation_ip_geolocation;"
```

Two rows naming the United States, Google and Cloudflare mean the key, the plugin and the destination are all working.

### Spec Reference

| Field | Type | Default | Description |
|---|---|---|---|
| `api_key` | string | required | Your IPGeolocation.io API key |
| `ips` | list of strings | `[]` | IP addresses to look up. Empty means the caller's own public IP |
| `asns` | list of strings | `[]` | AS numbers for the ASN table, for example `["15169"]` |
| `user_agents` | list of strings | `[]` | User agent strings for the user agent table |
| `include_security` | bool | `false` | Adds the security object to geolocation rows. Paid plans |
| `include_abuse` | bool | `false` | Adds the abuse contact object to geolocation rows. Paid plans |
| `include_geo_accuracy` | bool | `false` | Adds `locality`, `accuracy_radius` and `confidence`. Paid plans |
| `include_dma_code` | bool | `false` | Adds `dma_code`, US locations only. Paid plans |
| `include_hostname` | bool | `false` | Adds the reverse DNS `hostname`. Paid plans |
| `hostname_lookup` | string | `fallback_live` | Resolution mode: `database`, `live` or `fallback_live` |
| `asn_include` | list of strings | `[]` | Optional ASN modules: `peers`, `upstreams`, `downstreams`, `routes`, `whois_response`, or `"*"` |
| `endpoint` | string | `https://api.ipgeolocation.io` | API base URL |
| `timeout` | string | `30s` | Per request HTTP timeout |
| `retry_attempts` | int | `3` | Retries for 429 and 5xx responses |
| `concurrency` | int | `10` | Tables synced in parallel |
| `rate_limit` | int | `15` | Maximum API requests per second |
| `user_agent` | string | `cq-source-ipgeolocation/1.0` | User-Agent header sent with each request |

The spec is validated before the first request, so a bad string or an unknown `asn_include` value fails immediately and costs no credits.

### Keeping Your API Key Out of Git

Reference an environment variable with `${VAR_NAME}` and export it at runtime. CloudQuery expands it when the config loads, and the key never touches a tracked file.

---

## Available Tables

### `ipgeolocation_ip_geolocation`

The main table. One row per entry in `ips`, or a single row for the caller's public IP when `ips` is empty. Works on the free plan.

Key columns:

| Column | Type | Description |
|---|---|---|
| `ip` | text | The IP that was looked up |
| `continent_code` | text | Two-letters continent code |
| `country_code2` | text | ISO alpha-2 country code |
| `country_code3` | text | ISO alpha-3 country code |
| `country_name` | text | Country name |
| `country_capital` | text | Capital city |
| `state_prov` | text | State or province |
| `state_code` | text | ISO 3166-2 subdivision code |
| `district` | text | District or county |
| `city` | text | City |
| `zipcode` | text | Postal code |
| `latitude` | text | Latitude |
| `longitude` | text | Longitude |
| `is_eu` | boolean | True for EU member states |
| `geoname_id` | text | GeoNames identifier |
| `calling_code` | text | International dialling prefix |
| `tld` | text | Country code top level domain |
| `languages` | text | Official languages, comma separated |
| `currency_code` | text | ISO 4217 currency code |
| `currency_name` | text | Currency name |
| `currency_symbol` | text | Currency symbol |
| `connection_type` | text | Connection type |
| `route` | text | CIDR block containing the IP |
| `is_anycast` | boolean | Anycast prefix |
| `is_cdn` | boolean | CDN address |
| `cdn_provider_name` | text | CDN operator |
| `as_number` | text | AS number, for example `AS15169` |
| `asn_organization` | text | Organization holding the ASN |
| `asn_type` | text | ISP, HOSTING, EDUCATION, GOVERNMENT or BUSINESS |
| `asn_rir` | text | ARIN, RIPE, APNIC, LACNIC or AFRINIC |
| `company_name` | text | Company operating the IP |
| `company_domain` | text | Company domain |
| `timezone_name` | text | IANA timezone name |
| `timezone_offset` | double | UTC offset in seconds |
| `timezone_is_dst` | boolean | DST currently in effect |
| `hostname` | text | Reverse DNS name. Paid, opt-in |
| `locality` | text | Locality or neighbourhood. Paid, opt-in |
| `accuracy_radius` | text | Accuracy radius in kilometres. Paid, opt-in |
| `confidence` | text | `low`, `medium` or `high`. Paid, opt-in |
| `dma_code` | text | Designated Market Area code, US only. Paid, opt-in |

With `include_security: true` the table also carries every security column listed below, unprefixed. With `include_abuse: true` it carries every abuse column with an `abuse_` prefix.

Example:

```sql
SELECT ip, country_name, city, company_name, timezone_name, as_number
FROM ipgeolocation_ip_geolocation
WHERE country_code2 = 'US';
```

---

### `ipgeolocation_ip_security`

Threat intelligence for each IP in `ips`. Requires a paid plan and costs 2 credits per lookup.

| Column | Type | Description |
|---|---|---|
| `ip` | text | The IP that was looked up |
| `threat_score` | integer | Composite risk score from 0 to 100 |
| `is_anonymous` | boolean | Any anonymisation detected |
| `is_vpn` | boolean | Known VPN exit node |
| `vpn_provider_names` | text | VPN operators, comma separated |
| `vpn_confidence_score` | integer | VPN detection confidence |
| `vpn_last_seen` | text | Date last seen as a VPN exit |
| `is_proxy` | boolean | Known proxy |
| `proxy_provider_names` | text | Proxy operators, comma separated |
| `proxy_confidence_score` | integer | Proxy detection confidence |
| `proxy_last_seen` | text | Date last seen as a proxy |
| `is_residential_proxy` | boolean | Proxy on a residential connection |
| `is_tor` | boolean | Tor exit node |
| `is_relay` | boolean | Relay network such as iCloud Private Relay |
| `relay_provider_name` | text | Relay operator |
| `is_known_attacker` | boolean | Flagged for attack activity |
| `is_bot` | boolean | Automated traffic |
| `bot_confidence_score` | integer | Bot detection confidence |
| `bot_operator_name` | text | Who runs the bot |
| `bot_type` | text | Bot classification |
| `is_known_good_bot` | boolean | Declared, well behaved crawler |
| `bot_last_seen` | text | Date last seen as a bot |
| `is_spam` | boolean | On spam block lists |
| `is_cloud_provider` | boolean | Cloud or hosting infrastructure |
| `cloud_provider_name` | text | Cloud provider name |
| `is_corporate_gateway` | boolean | Shared corporate egress point |
| `corporate_gateway_type` | text | Gateway type |
| `corporate_gateway_provider_name` | text | Gateway operator |

Example:

```sql
SELECT ip, threat_score, is_vpn, is_proxy, is_tor, is_residential_proxy
FROM ipgeolocation_ip_security
WHERE threat_score > 50
ORDER BY threat_score DESC;
```

---

### `ipgeolocation_abuse_contact`

The contact registered with the Regional Internet Registry for reporting abuse from a network block. Requires a paid plan and costs 1 credit per lookup.

| Column | Type | Description |
|---|---|---|
| `ip` | text | The IP that was looked up |
| `route` | text | CIDR block the contact covers |
| `country` | text | Country of the registrant |
| `name` | text | Contact or incident response team name |
| `organization` | text | Organization responsible for the block |
| `kind` | text | Contact kind, such as group or individual |
| `address` | text | Postal address |
| `emails` | text | Abuse email addresses, comma separated |
| `phone_numbers` | text | Abuse phone numbers, comma separated |

Example:

```sql
SELECT ip, route, organization, emails, phone_numbers
FROM ipgeolocation_abuse_contact
WHERE emails != '';
```

---

### `ipgeolocation_asn`

Autonomous System details, looked up by IP address, by AS number, or both. Requires a paid plan and costs 1 credit per lookup.

| Column | Type | Description |
|---|---|---|
| `query_ip` | text | The IP this row was looked up by. Empty for AS number lookups |
| `query_asn` | text | The AS number this row was looked up by. Empty for IP lookups |
| `as_number` | text | AS number returned by the API |
| `asn_name` | text | Short registered name |
| `organization` | text | Organization holding the ASN |
| `country` | text | Country of registration |
| `type` | text | ISP, HOSTING, EDUCATION, GOVERNMENT or BUSINESS |
| `domain` | text | Primary domain |
| `date_allocated` | text | Allocation date |
| `allocation_status` | text | Registry status |
| `rir` | text | Regional Internet Registry |
| `num_of_ipv4_routes` | text | Count of announced IPv4 prefixes |
| `num_of_ipv6_routes` | text | Count of announced IPv6 prefixes |
| `peers` | text | Peer AS numbers. opt-in |
| `upstreams` | text | Upstream AS numbers. opt-in |
| `downstreams` | text | Downstream AS numbers. opt-in |
| `routes` | text | Announced CIDR prefixes. opt-in |
| `whois_response` | text | Raw WHOIS record. opt-in |

The last five columns stay NULL unless you name them in `asn_include`. That is deliberate: a tier 1 network can announce tens of thousands of prefixes and peer with thousands of other networks, and pulling all of it on every sync makes the table slow to write and awkward to query.

```yaml
asn_include: ["upstreams", "downstreams"]
```

Route counts come back as strings, so cast them when sorting:

```sql
SELECT as_number, organization, type, num_of_ipv4_routes
FROM ipgeolocation_asn
ORDER BY CAST(num_of_ipv4_routes AS INTEGER) DESC;
```

---

### `ipgeolocation_user_agent`

Parses each string in `user_agents`. The table is skipped when the list is empty. Requires a paid plan and costs 1 credit per string.

| Column | Type | Description |
|---|---|---|
| `user_agent_string` | text | The raw string that was parsed |
| `name` | text | Client name, such as Chrome or Googlebot |
| `type` | text | Client type: browser, bot, library |
| `version` | text | Full version |
| `version_major` | text | Major version |
| `device_name` | text | Device model |
| `device_type` | text | Desktop, Mobile, Tablet |
| `device_brand` | text | Manufacturer |
| `device_cpu` | text | CPU architecture |
| `engine_name` | text | Rendering engine |
| `engine_version` | text | Engine version |
| `os_name` | text | Operating system |
| `os_version` | text | OS version |
| `os_build` | text | Build identifier |

Example:

```sql
SELECT name, type, device_type, os_name, os_version
FROM ipgeolocation_user_agent
ORDER BY type;
```

---

## Query Examples

### Where are these IPs?

```sql
SELECT ip, country_name, state_prov, city, latitude, longitude, timezone_name
FROM ipgeolocation_ip_geolocation;
```

### Which IPs are hiding behind anonymisation?

```sql
SELECT ip, is_vpn, is_proxy, is_tor, is_relay, vpn_provider_names, threat_score
FROM ipgeolocation_ip_security
WHERE is_anonymous = true
ORDER BY threat_score DESC;
```

### Which IPs belong to cloud infrastructure rather than end users?

```sql
SELECT ip, cloud_provider_name, threat_score
FROM ipgeolocation_ip_security
WHERE is_cloud_provider = true;
```

### Which automation is undeclared?

Good crawlers identify themselves. This finds traffic that behaves like a bot without doing so.

```sql
SELECT ip, bot_type, bot_operator_name, bot_confidence_score, bot_last_seen
FROM ipgeolocation_ip_security
WHERE is_bot = true AND is_known_good_bot = false
ORDER BY bot_confidence_score DESC;
```

### Which IPs are corporate gateways?

A gateway can front an entire company, so blocking one is much wider than blocking a single user.

```sql
SELECT ip, corporate_gateway_provider_name, corporate_gateway_type
FROM ipgeolocation_ip_security
WHERE is_corporate_gateway = true;
```

### Country breakdown across everything you synced

```sql
SELECT country_name, COUNT(*) AS ip_count
FROM ipgeolocation_ip_geolocation
GROUP BY country_name
ORDER BY ip_count DESC;
```

### Who provides transit to each network?

Requires `asn_include: ["upstreams"]`.

```sql
SELECT as_number, organization, country, upstreams
FROM ipgeolocation_asn
WHERE upstreams IS NOT NULL AND upstreams != '';
```

---

## Joining Tables

Because everything lands in one database, joins are ordinary SQL and cost nothing extra at query time. The API calls happened during the sync.

### Location plus threat data

```sql
SELECT
  g.ip,
  g.country_name,
  g.city,
  g.company_name,
  s.threat_score,
  s.is_vpn,
  s.is_proxy,
  s.is_tor
FROM ipgeolocation_ip_geolocation g
JOIN ipgeolocation_ip_security s ON g.ip = s.ip
WHERE s.threat_score > 30
ORDER BY s.threat_score DESC;
```

### Everything needed to file an abuse report

```sql
SELECT
  g.ip,
  g.country_name,
  g.city,
  g.as_number,
  g.asn_organization,
  s.threat_score,
  s.is_known_attacker,
  a.route,
  a.emails
FROM ipgeolocation_ip_geolocation g
JOIN ipgeolocation_ip_security s ON g.ip = s.ip
JOIN ipgeolocation_abuse_contact a ON g.ip = a.ip
WHERE s.is_known_attacker = true;
```

### Joining against your own tables

This is the part that makes a warehouse worth the trouble. Once IP intelligence is a table, it joins to anything else you already have:

```sql
SELECT
  l.user_id,
  l.login_at,
  g.country_name,
  g.city,
  s.is_vpn,
  s.threat_score
FROM auth_logins l
JOIN ipgeolocation_ip_geolocation g ON l.source_ip = g.ip
JOIN ipgeolocation_ip_security s ON l.source_ip = s.ip
WHERE s.threat_score > 40
  AND l.login_at > NOW() - INTERVAL '7 days';
```

Boolean literals above follow PostgreSQL. SQLite stores booleans as 0 and 1, so write `is_vpn = 1` there.

---

## Real World Use Cases

### Building a Security Enrichment Table

Most SIEM platforms can join against a lookup table far more cheaply than they can call an external API per event. Syncing your known and suspicious IPs once a day gives the SIEM a local table to work from.

```yaml
kind: source
spec:
  name: ipgeolocation
  registry: grpc
  path: "localhost:7777"
  tables:
    - "ipgeolocation_ip_geolocation"
    - "ipgeolocation_ip_security"
  destinations: ["postgresql"]
  spec:
    api_key: "${IPGEOLOCATION_API_KEY}"
    ips:
      - "185.220.101.1"
      - "2.56.188.34"
      - "45.142.212.100"
    include_security: true
    rate_limit: 5
```

Run it on a schedule, point your detection rules at the resulting tables, and the enrichment happens at query time with no per event API cost.

### Compliance Reporting on Geographic Access

If your service is restricted by jurisdiction, the auditable question is not "which country is this IP in today" but "which country did we record it in when access was granted". A synced table answers that, because the rows are timestamped by the sync that wrote them.

```sql
SELECT ip, country_name, country_code2, city, asn_organization
FROM ipgeolocation_ip_geolocation
WHERE country_code2 IN ('RU', 'KP', 'IR', 'SY', 'CU');
```

Keeping historical snapshots also lets you show what was known at the time rather than what is true now, which is usually what an auditor is asking for.

### Vendor and Third Party Network Inventory

Platform teams often hold a list of vendor IP ranges in a spreadsheet nobody has checked in two years. Syncing them produces a table you can diff against the previous run.

```sql
SELECT query_ip, as_number, organization, type, country, rir
FROM ipgeolocation_asn
WHERE query_ip != ''
ORDER BY organization;
```

The `type` column separates hosting providers from ISPs, education networks and enterprise allocations, which is often the first thing to check when a vendor claims traffic came from their corporate office.

### Incident Response and Abuse Reporting

When malicious traffic comes from a network you do not control, you need the right contact and the right block. One sync gives you both.

```sql
SELECT ip, organization, route, emails, phone_numbers, country
FROM ipgeolocation_abuse_contact
WHERE ip = '45.142.212.100';
```

The `route` column is the CIDR block the contact is registered for, which tells you the scope a report or a block would cover.

### Traffic Analysis from Web Server Logs

Extract distinct user agent strings from your access logs, feed them through the user agent table, and you get a browser and bot breakdown without maintaining a parsing library.

```sql
SELECT type, name, COUNT(*) AS hits
FROM ipgeolocation_user_agent
GROUP BY type, name
ORDER BY hits DESC;
```

### Timezone Aware Scheduling

`timezone_name` returns an IANA identifier, which most date libraries accept directly.

```sql
SELECT ip, country_name, city, timezone_name, timezone_offset
FROM ipgeolocation_ip_geolocation;
```

---

## API Credits Reference

Each lookup spends credits from your plan. Multiply by the number of entries in the matching input list.

| Table | Endpoint | Credits per lookup |
|---|---|---|
| `ipgeolocation_ip_geolocation`, base fields | `/v3/ipgeo` | 1 |
| `ipgeolocation_ip_geolocation`, with security and abuse | `/v3/ipgeo` | Up to 4 |
| `ipgeolocation_ip_security` | `/v3/security` | 2 |
| `ipgeolocation_abuse_contact` | `/v3/abuse` | 1 |
| `ipgeolocation_asn` | `/v3/asn` | 1 |
| `ipgeolocation_user_agent` | `/v3/user-agent` | 1 |

The geo accuracy, DMA code and hostname modules add columns without adding credits.

Two ways to spend more than you meant to:

- Syncing `ipgeolocation_ip_security` while also setting `include_security: true`. That fetches the same data twice and bills for both. Pick the wide geolocation table or the dedicated security table, not both.
- Setting `tables: ["*"]` on a free plan. Every paid table returns 401 for every IP, and while those failures cost no credits, the sync looks successful and the tables come back empty.

The free Developer plan allows 1,000 requests per day. Paid plans start at 150,000 requests per month. Current figures are on the [pricing page](https://ipgeolocation.io/pricing.html).

---

## Scheduling Syncs

`cloudquery sync` is one command, so any scheduler works. A cron entry for a nightly run:

```cron
0 3 * * * cd /opt/ipgeo && IPGEOLOCATION_API_KEY=xxx /usr/local/bin/cloudquery sync ipgeo.yml >> sync.log 2>&1
```

The plugin binary has to be serving when the sync runs. Under systemd, a long-running unit for `cq-source-ipgeolocation serve` plus a timer for the sync keeps the two concerns separate.

Size the interval against your credits. Geolocation data for a given IP changes slowly, so daily is enough for most uses. Threat signals move faster, and hourly is reasonable for a small list of IPs you actively monitor.

---

## Troubleshooting

### The sync succeeds but a table is empty

Failed lookups are logged as warnings and skipped so one bad address cannot abort a run. An empty table usually means every lookup failed the same way. Re-run with `--log-level debug` and look for `failed to fetch` lines, which name the IP and the error.

The usual causes: a paid table on a free key, an `ips` list made up entirely of private ranges, or `user_agents` left empty, which skips the user agent table silently.

### Columns are NULL when you expected values

NULL means the module was never requested. Check the matching flag: `include_security`, `include_abuse`, `include_geo_accuracy`, `include_dma_code`, `include_hostname`, or `asn_include`.

An empty string means something different. It means the module ran and the API had nothing to return, which is normal for `dma_code` outside the US and for `upstreams` on a network that buys transit from nobody.

Because of this, write filters positively. `WHERE is_vpn = true` works whether or not the module ran, while `WHERE is_vpn != true` drops NULL rows in most SQL engines.

### HTTP status codes

| Status | Meaning |
|---|---|
| 400 | Malformed request, usually an invalid IP or AS number |
| 401 | Invalid key, or an endpoint your plan does not cover |
| 403 | Key disabled or blocked origin |
| 404 | No data for the queried resource |
| 423 | Bogon address: private, loopback, reserved or multicast |
| 429 | Rate limit or daily quota exhausted |
| 5xx | Server fault, retried automatically |

429 and 5xx responses are retried with backoff of 1 second, then 2, then 4. Other 4xx responses are returned immediately.

### Rate limits

`rate_limit` is a token bucket in requests per second. Lower it if your plan is tight or if other systems share the key. `concurrency` controls parallel table syncs and does not raise the request rate, since the limiter applies across the whole sync.

---

## Building from Source

```bash
git clone https://github.com/IPGeolocation/cq-source-ipgeolocation
cd cq-source-ipgeolocation
go build ./...
go build -o cq-source-ipgeolocation
```

Run the tests:

```bash
go test ./... -v
```

The suite runs against a stub HTTP server, so it needs no API key, works offline and spends no credits. It covers spec validation, retry and rate limiting behaviour, response flattening for all five tables, and a check that every documented API field reaches a column.

The repository also ships eight example configs under `example/`, covering the free plan, security focused syncs, ASN topology, user agent parsing, edge cases such as IPv6 and bogons, a PostgreSQL destination, and caller IP auto detection.

---

## Related Resources

- [IPGeolocation.io API Documentation](https://ipgeolocation.io/documentation.html)
- [IP Geolocation API Reference](https://ipgeolocation.io/documentation/ip-geolocation-api.html)
- [IP Security API Reference](https://ipgeolocation.io/documentation/ip-security-api.html)
- [IP Abuse Contact API Reference](https://ipgeolocation.io/documentation/ip-abuse-contact-api.html)
- [ASN API Reference](https://ipgeolocation.io/documentation/asn-api.html)
- [User Agent API Reference](https://ipgeolocation.io/documentation/user-agent-api.html)
- [Plugin repository on GitHub](https://github.com/IPGeolocation/cq-source-ipgeolocation)
- [CloudQuery documentation](https://docs.cloudquery.io)
- [IPGeolocation.io Steampipe plugin](https://github.com/IPGeolocation/steampipe-plugin-ipgeolocation), for SQL queries against the API without a sync step
- [What is IP Geolocation and How Does It Work?](https://ipgeolocation.io/guides/what-is-ip-geolocation-how-it-works)
- [What is an ASN?](https://ipgeolocation.io/guides/what-is-an-asn)

---

## Frequently Asked Questions

<details>
<summary><strong>Do I need a paid plan to use this plugin?</strong></summary>
No. The free Developer plan allows 1,000 requests per day and covers the `ipgeolocation_ip_geolocation` table with its base fields, which is enough for development and small syncs. The security, abuse, ASN and user agent tables need a paid plan, as do the optional modules such as `include_security` and `include_abuse`.
</details>

<details>
<summary><strong>How is this different from the Steampipe plugin?</strong></summary>
Steampipe queries the API live, so every `SELECT` makes API calls and the results are always current but never stored. CloudQuery syncs data into a database you own, so queries are free and fast afterwards but reflect the last sync. Use Steampipe for ad hoc investigation, CloudQuery for enrichment tables other systems join against.
</details>

<details>
<summary><strong>Which destinations does it support?</strong></summary>
Every CloudQuery destination, including PostgreSQL, SQLite, BigQuery, Snowflake, ClickHouse, DuckDB, S3 and GCS. Source plugins do not care where rows are written, so switching destinations is a change to one YAML block.
</details>

<details>
<summary><strong>Does it support IPv6?</strong></summary>
Yes. Put IPv6 addresses in `ips` exactly as you would IPv4. Coverage varies by prefix, and some IPv6 blocks return less location detail than comparable IPv4 space.
</details>

<details>
<summary><strong>Can I look up a domain name instead of an IP address?</strong></summary>
No. Resolve the name to an address first. The plugin sends whatever is in `ips` directly to the API.
</details>

<details>
<summary><strong>What happens if one IP in my list is invalid?</strong></summary>
It is logged as a warning and skipped, and the sync continues with the rest. Malformed addresses, private ranges and unsupported prefixes will not abort a run.
</details>

<details>
<summary><strong>How do I look up the machine's own public IP?</strong></summary>
Leave `ips` out of the spec. The geolocation, security and abuse tables then query whatever public address the syncing machine egresses from, which is a quick way to check what a network looks like from outside.
</details>

<details>
<summary><strong>Why are my security columns NULL rather than false?</strong></summary>
NULL means the module was never requested, while false means the API checked and found nothing. Keeping the two distinct stops an unfetched IP from looking like a clean one. Set `include_security: true` to populate them.
</details>

<details>
<summary><strong>Why is the ASN table missing peers and routes?</strong></summary>
Those modules are opt-in through `asn_include`. The default keeps rows compact, because large transit networks return peer and route lists in the thousands. Add only the modules you query, or `"*"` for all of them.
</details>

<details>
<summary><strong>How many credits will a sync cost?</strong></summary>
Count the lookups: 1 credit for geolocation, 2 for security, 1 for abuse, 1 for ASN, 1 for a user agent string, multiplied by the number of entries in each input list. Enabling `include_security` and `include_abuse` raises a geolocation lookup to as many as 4 credits.
</details>

<details>
<summary><strong>Can I run syncs on a schedule?</strong></summary>
Yes. `cloudquery sync` is a single command, so cron, systemd timers, GitHub Actions, Airflow and similar tools all work. Keep the plugin binary serving while the sync runs, and size the interval against your daily credit allowance.
</details>

<details>
<summary><strong>Does the plugin cache results between syncs?</strong></summary>
No. Every sync makes fresh API calls. To spend less, sync less often or shorten the `ips` list. Geolocation data changes slowly for most addresses, so daily is enough for many uses.
</details>

<details>
<summary><strong>Which Go version do I need?</strong></summary>
Go 1.22.7 or later, matching the `go` directive in `go.mod`.
</details>