# IPGeolocation.io Steampipe Plugin

## Overview

Query IP geolocation, threat intelligence, ASN details, and abuse contact data using plain SQL. The IPGeolocation.io Steampipe plugin connects your SQL workflows directly to the [IPGeolocation.io API](https://ipgeolocation.io/) so you can enrich, filter, and join IP intelligence data without writing a single line of application code.

---

## What is Steampipe?

[Steampipe](https://steampipe.io/) is an open-source tool that lets you query cloud, SaaS, and API data using SQL. It acts as a Postgres-compatible query engine and maps external APIs to tables that you can filter, join, and aggregate with standard SQL syntax.

Instead of writing scripts that call APIs, handle pagination, and parse JSON, you write a `SELECT` statement. Steampipe handles authentication, rate limiting, and response parsing under the hood.

This makes it practical for:

- Security engineers who want to enrich IP addresses in logs without building a data pipeline
- DevOps teams who want to check ASN or threat data across a list of server IPs
- Compliance teams who need a repeatable, auditable query against IP intelligence data
- Data analysts who want to JOIN IP geolocation with their own internal tables

The IPGeolocation.io plugin brings the full [IPGeolocation.io API](https://ipgeolocation.io/documentation.html) into Steampipe as queryable tables.

---

## What Does This Plugin Do?

The plugin exposes four tables that map to IPGeolocation.io's core API endpoints:

| Table | API Endpoint | What It Returns |
|---|---|---|
| `ipgeolocation_ip` | `/v3/ipgeo` | Country, city, ISP, timezone, company, security flags, hostname, abuse contact |
| `ipgeolocation_security` | `/v3/security` | VPN, proxy, Tor, relay, bot, spam detection, and threat score |
| `ipgeolocation_abuse` | `/v3/abuse` | Abuse contact emails, phone, organization, CIDR route |
| `ipgeolocation_asn` | `/v3/asn` | AS number, routes, peers, upstreams, downstreams, WHOIS |

Each table requires an `ip` filter in the `WHERE` clause. You pass an IPv4 or IPv6 address, and the plugin fetches the data from the API and returns it as a row you can work with in SQL.

---

## Prerequisites

Before you start, you need:

- [Steampipe](https://steampipe.io/downloads) version 0.20 or higher
- [Go](https://golang.org/dl/) 1.21 or higher (only if building from source)
- An IPGeolocation.io API key. Get a free key at [app.ipgeolocation.io/dashboard](https://app.ipgeolocation.io/dashboard). The free plan supports up to 1,000 requests per day.

---

## Installation

Install the plugin with the Steampipe plugin manager:

```bash
steampipe plugin install ipgeolocation/ipgeolocation
```

Steampipe downloads and installs the plugin binary automatically. No manual build step is required for standard use.

Verify the installation:

```bash
steampipe plugin list
```

You should see `ipgeolocation/ipgeolocation` in the output.

---

## Configuration

The plugin reads your API key from a config file at:

```
~/.steampipe/config/ipgeolocation.spc
```

Open or create that file and add the following:

```hcl
connection "ipgeolocation" {
  plugin = "ipgeolocation/ipgeolocation"

  # Your API key from https://app.ipgeolocation.io/dashboard
  # You can also set the IPGEOLOCATION_API_KEY environment variable instead.
  api_key = "your_api_key_here"
}
```

Alternatively, set the environment variable before running Steampipe:

```bash
export IPGEOLOCATION_API_KEY="your_api_key_here"
steampipe query
```

If both the config file and the environment variable are set, the environment variable takes precedence.

To verify your connection is working, run a test query:

```sql
select ip, country_name, city, company_name
from ipgeolocation_ip
where ip = '8.8.8.8';
```

If you see a row with Google's ASN details and geolocation for `8.8.8.8`, the plugin is configured correctly.

---

## Available Tables

### `ipgeolocation_ip`

This is the main table. It returns geolocation, network, timezone, company, security, abuse, and hostname data for any IP address.

**Required filter:** `WHERE ip = '<address>'`

**Key columns:**

| Column | Type | Description |
|---|---|---|
| ip | text | Queried IP address |
| hostname | text | Resolved hostname (paid) |
| continent_code | text | Two-letter continent code |
| continent_name | text | Continent name |
| country_code2 | text | ISO alpha-2 country code |
| country_code3 | text | ISO alpha-3 country code |
| country_name | text | Country name |
| country_name_official | text | Official country name |
| country_capital | text | Capital city |
| state_prov | text | State / province |
| state_code | text | ISO 3166-2 state code |
| district | text | District / county |
| city | text | City |
| locality | text | Locality (Paid) |
| zipcode | text | ZIP / postal code |
| latitude | text | Latitude |
| longitude | text | Longitude |
| accuracy_radius | text | Accuracy radius (km) (Paid) |
| confidence | text | Confidence level (low, medium, high) (Paid) |
| is_eu | bool | True if EU member state |
| country_flag | text | Country flag image URL |
| country_emoji | text | Country flag emoji |
| geoname_id | text | GeoNames ID |
| dma_code | text | DMA code (US only) (Paid) |
| asn | text | Autonomous System Number |
| organization | text | ASN organization |
| connection_type | text | Connection type |
| timezone_name | text | IANA timezone name |
| timezone_offset | bigint | UTC offset (seconds) |
| timezone_offset_with_dst | bigint | UTC offset incl. DST |
| timezone_current_time | text | Current local time |
| timezone_is_dst | bool | DST active? |
| threat_score | bigint | Threat score 0-100 (paid) |
| is_vpn | bool | VPN exit node? (paid) |
| is_proxy | bool | Proxy? (paid) |
| is_tor | bool | Tor exit node? (paid) |
| is_bot | bool | Bot IP? (paid) |
| is_spam | bool | On spam lists? (paid) |
| is_cloud_provider | bool | Cloud/hosting IP? (paid) |
| is_known_attacker | bool | Known attacker? (paid) |
| is_anonymous | bool | Anonymous IP? (paid) |
| is_residential_proxy | bool | Residential proxy? (paid) |
| company_name | text | Company name (paid) |
| company_domain | text | Company domain (paid) |
| company_type | text | Company type (paid) |
| abuse_name | text | Abuse contact name (paid) |
| abuse_email | text | Abuse email (paid) |
| abuse_address | text | Abuse postal address (paid) |
| abuse_phone | text | Abuse phone (paid) |
| abuse_network | text | Abuse CIDR block (paid) |

**Example:**

```sql
select
  ip,
  country_name,
  city,
  company_name,
  timezone_name,
  is_vpn,
  threat_score
from
  ipgeolocation_ip
where
  ip = '1.1.1.1';
```

---

### `ipgeolocation_security`

Returns threat intelligence signals for an IP address. This includes VPN, proxy, Tor, relay, bot, and spam detection along with a composite threat score.

**Required filter:** `WHERE ip = '<address>'`

**Key columns:**

| Column | Type | Description |
|---|---|---|
| ip | text | Queried IP address |
| threat_score | bigint | 0–100 overall risk score |
| is_anonymous | bool | Any anonymization detected |
| is_vpn | bool | Known VPN exit node |
| vpn_provider_names | jsonb | Array of VPN provider names |
| vpn_confidence_score | bigint | VPN detection confidence 0–100 |
| vpn_last_seen | text | Last seen as VPN (YYYY-MM-DD) |
| is_proxy | bool | Known proxy |
| proxy_provider_names | jsonb | Array of proxy provider names |
| proxy_confidence_score | bigint | Proxy detection confidence 0–100 |
| proxy_last_seen | text | Last seen as proxy (YYYY-MM-DD) |
| is_residential_proxy | bool | Residential proxy |
| is_tor | bool | Tor exit node |
| is_relay | bool | Relay network (e.g. iCloud Private Relay) |
| relay_provider_name | text | Relay provider name |
| is_cloud_provider | bool | Cloud or hosting IP |
| cloud_provider_name | text | Cloud provider name |
| is_known_attacker | bool | Flagged for attack activity |
| is_bot | bool | Bot activity detected |
| is_spam | bool | On spam block lists |

**Example:**

```sql
select
  ip,
  is_vpn,
  is_proxy,
  is_tor,
  is_bot,
  threat_score,
  is_residential_proxy
from
  ipgeolocation_security
where
  ip = '185.220.101.1';
```

---

### `ipgeolocation_abuse`

Returns the abuse contact information for an IP address. This is the contact details registered with the relevant Regional Internet Registry (RIR) for reporting abuse.

**Required filter:** `WHERE ip = '<address>'`

**Key columns:**

| Column | Type | Description |
|---|---|---|
| ip | text | Queried IP address |
| route | text | CIDR block covering the IP |
| country | text | ISO alpha-2 country of registrant |
| name | text | Abuse contact or IRT name |
| organization | text | Responsible organisation |
| kind | text | Contact kind (e.g. "group") |
| address | text | Postal address |
| emails | jsonb | Array of abuse email addresses |
| phone_numbers | jsonb | Array of abuse phone numbers |

**Example:**

```sql
select
  ip,
  abuse_email,
  abuse_phone,
  organization,
  network
from
  ipgeolocation_abuse
where
  ip = '192.0.2.1';
```

---

### `ipgeolocation_asn`

Returns Autonomous System Number (ASN) details for an IP address, including routing information, peers, upstreams, downstreams, and WHOIS data.

**Required filter:** `WHERE ip = '<address>'`

**Key columns:**

| Column | Type | Description |
|---|---|---|
| ip | text | IP used for lookup (when queried by IP) |
| asn | text | ASN used for lookup (when queried by ASN) |
| as_number | text | AS number e.g. "AS15169" |
| asn_name | text | Short registered ASN name |
| organization | text | Organisation owning the ASN |
| country | text | ISO alpha-2 country of registration |
| type | text | ASN type: ISP, HOSTING, EDUCATION, GOVERNMENT, BUSINESS |
| domain | text | Organisation's primary domain |
| date_allocated | text | Allocation date (YYYY-MM-DD) |
| allocation_status | text | e.g. "assigned" |
| rir | text | Regional Internet Registry (ARIN, RIPE, APNIC, LACNIC, AFRINIC) |
| num_of_ipv4_routes | bigint | Number of IPv4 prefixes announced |
| num_of_ipv6_routes | bigint | Number of IPv6 prefixes announced |
| routes | jsonb | Array of CIDR prefixes announced |
| peers | jsonb | Array of peer ASNs |
| upstreams | jsonb | Array of upstream/transit ASNs |
| downstreams | jsonb | Array of downstream customer ASNs |
| whois_response | text | Raw WHOIS text |

**Example:**

```sql
select
  ip,
  as_number,
  organization,
  country,
  type,
  domain
from
  ipgeolocation_asn
where
  ip = '8.8.8.8';
```

---

## Query Examples

### Look up basic geolocation for a single IP

```sql
select
  ip,
  country_name,
  country_code2,
  city,
  latitude,
  longitude,
  timezone_name
from
  ipgeolocation_ip
where
  ip = '8.8.8.8';
```

---

### Check whether an IP is behind a VPN or proxy

```sql
select
  ip,
  is_vpn,
  is_proxy,
  is_tor,
  is_relay,
  vpn_provider_names,
  threat_score
from
  ipgeolocation_security
where
  ip = '104.28.0.1';
```

---

### Get the ISP and network details for an IP

```sql
select
  ip,
  company_name,
  organization,
  asn,
  country_name,
  city
from
  ipgeolocation_ip
where
  ip = '203.0.113.10';
```

---

### Find the abuse contact for a suspicious IP

```sql
select
  ip,
  abuse_email,
  abuse_phone,
  organization,
  network
from
  ipgeolocation_abuse
where
  ip = '198.51.100.5';
```

---

### Look up ASN routing details

```sql
select
  ip,
  as_number,
  asn_name,
  organization,
  type,
  domain,
  country
from
  ipgeolocation_asn
where
  ip = '1.1.1.1';
```

---

### Filter by country code

```sql
select
  ip,
  country_name,
  city
from
  ipgeolocation_ip
where
  ip = '103.21.244.0'
  and country_code2 = 'AU';
```

---

### Check high-risk signals across multiple IPs using a subquery

Steampipe supports querying a list of IPs by using a `UNION ALL` subquery:

```sql
select
  ip,
  is_vpn,
  is_tor,
  threat_score
from
  ipgeolocation_security
where
  ip in (
    select ip from (values
      ('185.220.101.1'),
      ('45.142.212.100'),
      ('195.154.122.36')
    ) as t(ip)
  );
```

---

## Real-World Use Cases

### 1. Log Enrichment for Security Teams

Security engineers frequently need to enrich raw server logs with geolocation and threat context before investigating alerts. Instead of building a Python script that loops over IPs and calls the API, you can use Steampipe to pull location and threat signals alongside your internal log data.

**Scenario:** You have a list of login attempt source IPs from a failed authentication spike and want to quickly understand their geography and threat posture.

```sql
-- Enrich a known suspicious IP from your logs
select
  g.ip,
  g.country_name,
  g.city,
  g.company_name,
  g.organization,
  s.is_vpn,
  s.is_proxy,
  s.is_tor,
  s.threat_score,
  a.emails
from
  ipgeolocation_ip g
  join ipgeolocation_security s on g.ip = s.ip
  join ipgeolocation_abuse a on g.ip = a.ip
where
  g.ip = '185.220.101.1';
```

In a single query you get the country, the ISP, VPN and Tor flags, a threat score, and the abuse email you would use to file a report, all from one row. No API wrapper code needed.

---

### 2. Threat Triage for SOC Analysts

When working a security incident, analysts often need to triage a list of IPs quickly. This use case shows how to enrich a set of IPs flagged by your SIEM or IDS.

**Scenario:** Your intrusion detection system flagged five IPs in the last 24 hours. You need to know if any of them are Tor nodes or known VPN exit points before escalating.

```sql
select
  ip,
  is_vpn,
  is_tor,
  is_bot,
  is_spam,
  threat_score,
  vpn_provider_names
from
  ipgeolocation_security
where
  ip = '91.108.4.1'

union all

select
  ip,
  is_vpn,
  is_tor,
  is_bot,
  is_spam,
  threat_score,
  vpn_provider_names 
from
  ipgeolocation_security
where
  ip = '149.154.160.1';
```

Sort by `threat_score` descending to prioritize the most suspicious IPs first.

---

### 3. Compliance Audits for Restricted Jurisdictions

If your service has geographic access restrictions (sanctions compliance, licensing rules, or regional regulations), you can use Steampipe to audit specific IPs and confirm whether they fall inside or outside permitted regions.

**Scenario:** Your compliance team wants to verify that a set of IPs that accessed your platform do not originate from sanctioned countries.

```sql
select
  ip,
  country_name,
  country_code2,
  city,
  organization
from
  ipgeolocation_ip
where
  ip = '78.40.124.1'
  and country_code2 not in ('RU', 'KP', 'IR', 'SY', 'CU');
```

If the query returns no rows, the IP is from a restricted country. You can adapt this into a larger compliance report by joining against a table of access logs.

---

### 4. Infrastructure Inventory and Network Mapping

DevOps and network teams can use the ASN table to map out the ownership and routing of IPs across their cloud infrastructure or third-party vendor networks.

**Scenario:** You want to confirm that your CDN IPs are all originating from the expected ASN before pushing a firewall rule.

```sql
select
  ip,
  as_number,
  asn_name,
  organization,
  type,
  domain,
  country
from
  ipgeolocation_asn
where
  ip = '104.21.0.1';
```

Use the `type` field to distinguish between hosting provider IPs (`HOSTING`), ISP IPs (`ISP`), education networks (`EDUCATION`), and enterprise allocations (`ENTERPRISE`).

---

### 5. Abuse Reporting Workflow

When you detect malicious traffic from an IP, you need to report it to the right abuse contact. This query gives you everything needed to file a report: the IP's network block, the abuse email, and the responsible organization.

```sql
select
  ip,
  organization,
  network,
  emails,
  phone_numbers,
  country
from
  ipgeolocation_abuse
where
  ip = '45.142.212.100';
```

The `network` column returns the CIDR block that the IP belongs to, which is useful when you want to report or block the entire range rather than just the single address.

---

### 6. Timezone Routing for Scheduled Jobs

If you are building a scheduling system that needs to assign jobs based on a user's timezone, you can query the timezone data for their IP.

```sql
select
  ip,
  country_name,
  city,
  timezone_name,
  latitude,
  longitude
from
  ipgeolocation_ip
where
  ip = '202.12.29.1';
```

The `timezone_name` column returns a valid IANA timezone string that you can pass directly to your application's time handling library.

---

## Joining Tables

The plugin's tables all use `ip` as their key column, which means you can join them to pull data from multiple endpoints in a single query.

### Join geolocation and security data

```sql
select
  g.ip,
  g.country_name,
  g.city,
  s.is_vpn,
  s.is_proxy,
  s.is_tor,
  s.threat_score
from
  ipgeolocation_ip g
  join ipgeolocation_security s on g.ip = s.ip
where
  g.ip = '185.220.101.1';
```

### Join geolocation, security, and abuse data

```sql
select
  g.ip,
  g.country_name,
  g.city,
  g.organization,
  s.threat_score,
  s.is_vpn,
  a.emails,
  a.route
from
  ipgeolocation_ip g
  join ipgeolocation_security s on g.ip = s.ip
  join ipgeolocation_abuse a on g.ip = a.ip
where
  g.ip = '91.108.4.1';
```

Note that each `JOIN` against a different table triggers an additional API call. See the [API Credits Reference](#api-credits-reference) below for cost details.

---

## API Credits Reference

Each query to the IPGeolocation.io API consumes credits from your plan. The number of credits per query depends on which table you query and which data modules you request.

| Table | Endpoint | Credits per Query |
|---|---|---|
| `ipgeolocation_ip` (base geolocation only) | `/v3/ipgeo` | 1 |
| `ipgeolocation_ip` (with security module and/or abuse module) | `/v3/ipgeo` | Up to 4 |
| `ipgeolocation_security` | `/v3/security` | 2 |
| `ipgeolocation_abuse` | `/v3/abuse` | 1 |
| `ipgeolocation_asn` | `/v3/asn` | 1 |

When you join multiple tables in a single SQL query, Steampipe issues a separate API call for each table involved. A three-table join for a single IP costs the sum of the credits for each endpoint.

The free plan includes 1,000 requests per day. For production use cases or batch processing, see the [IPGeolocation.io pricing page](https://ipgeolocation.io/pricing.html) for plan options.

---

## Building from Source

If you want to develop the plugin or run the latest unreleased code, you can build from source.

**Prerequisites:**

- [Steampipe](https://steampipe.io/downloads) version 0.20 or higher
- [Go](https://golang.org/dl/) 1.21 or higher

**Clone, build, and install:**

```bash
git clone https://github.com/IPGeolocation/steampipe-plugin-ipgeolocation
cd steampipe-plugin-ipgeolocation
make setup
```

`make setup` runs `go mod tidy`, compiles the plugin binary, and installs both the binary and a sample config file into the correct Steampipe directories.

After making changes to the source, rebuild and reinstall:

```bash
make install
```

Run the test suite:

```bash
make test
```

The test suite runs Go unit tests and validates that the tables and columns are correctly defined.

---

## Related Resources

- [IPGeolocation.io API Documentation](https://ipgeolocation.io/documentation.html)
- [IP Location API Reference](https://ipgeolocation.io/documentation/ip-location-api.html)
- [IP Security API Reference](https://ipgeolocation.io/ip-security-api.html)
- [ASN API Reference](https://ipgeolocation.io/asn-api.html)
- [Steampipe Plugin Repository](https://github.com/IPGeolocation/steampipe-plugin-ipgeolocation)
- [Steampipe Hub Listing](https://hub.steampipe.io/plugins/ipgeolocation/ipgeolocation)
- [What is IP Geolocation? How It Works](https://ipgeolocation.io/guides/what-is-ip-geolocation-how-it-works)
- [What is an ASN?](https://ipgeolocation.io/guides/what-is-an-asn)

## Frequently Asked Questions

<details>
<summary><strong>What IP address formats does the plugin support?</strong></summary>
The plugin accepts any valid IPv4 address (for example, `8.8.8.8`) or IPv6 address (for example, `2001:4860:4860::8888`). Domain names are not supported directly, so you must resolve the domain to an IP address before querying it.
</details>

<details>
<summary><strong>Do I need a paid plan to use this plugin?</strong></summary>
No. The free plan provides up to 1,000 requests per day, which is sufficient for development and small-scale use cases. For production workloads or large batch enrichment, a paid plan is recommended.
</details>

<details>
<summary><strong>Why does my query return no rows?</strong></summary>
The most common cause is a missing `WHERE ip = '...'` filter or an incorrectly configured API key. Every table query requires an `ip` filter. Also verify that your API key is configured in `~/.steampipe/config/ipgeolocation.spc` or through the `IPGEOLOCATION_API_KEY` environment variable.
</details>

<details>
<summary><strong>Can I query multiple IP addresses in one statement?</strong></summary>
Steampipe does not support batch IP lookups directly. To query multiple IPs, combine individual queries with `UNION ALL` or use a script to iterate through a list of IP addresses. Keep your daily API request limits in mind when processing large datasets.
</details>

<details>
<summary><strong>Can I join IPGeolocation data with my own database tables?</strong></summary>
Yes. Steampipe provides a PostgreSQL-compatible query engine, allowing you to join IP geolocation data with your own tables or with data from other Steampipe plugins, such as AWS, GCP, or GitHub, using standard SQL.
</details>

<details>
<summary><strong>Does this plugin support IPv6?</strong></summary>
Yes. You can query any valid IPv6 address by specifying it in the `WHERE ip = '...'` clause. Coverage depends on the availability of data for the requested IPv6 prefix in the IPGeolocation database.
</details>

<details>
<summary><strong>Where can I find the full table schema?</strong></summary>
Complete documentation for all available tables and columns is available in the plugin repository under `docs/tables/` and on Steampipe Hub.
</details>

<details>
<summary><strong>How do I get an API key?</strong></summary>
Create an account through the IPGeolocation dashboard and retrieve your API key from your account dashboard. A free plan is available and does not require a credit card.
</details>


