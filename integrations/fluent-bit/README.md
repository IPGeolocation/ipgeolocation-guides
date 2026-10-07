# Fluent Bit IP Geolocation and Threat Enrichment with IPGeolocation.io

## Overview

Enrich your Fluent Bit logs with IP geolocation, VPN and proxy detection, and network data. Fluent Bit's built-in `geoip2` filter reads [IPGeolocation.io's downloadable IP databases](https://ipgeolocation.io/documentation/databases.html) in MMDB format, so every record that carries an IP address can leave Fluent Bit with:

- Country, region, city, coordinates and time zone from the [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html)
- VPN, proxy and Tor flags and a threat score from the [IP Security Database](https://ipgeolocation.io/ip-security-database.html)
- The [autonomous system number (ASN)](https://ipgeolocation.io/guides/what-is-an-asn) and its organization from the [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html)

All IPGeolocation.io IP databases are supported, not only these three. That includes the IP to Country, IP to City and IP to ISP databases, the [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP WHOIS Database](https://ipgeolocation.io/ip-whois-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html), and combined databases that hold several of these in one file.

The lookups run against files on your own machine. There are no API calls and no plugin to install. To decide whether a database or an API suits your workload, see [IP geolocation API vs database](https://ipgeolocation.io/guides/ip-geolocation-api-vs-database-guide).

> [!TIP]
> IPGeolocation.io databases use their own paths, such as `%{location.country.code2}`. This guide and the [field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) list all paths.

---

## Why use IPGeolocation.io databases with Fluent Bit

- **Location and security in one pass.** Country and city sit next to VPN, proxy, residential proxy, Tor, relay and bot flags and a threat score in the same record.
- **Every IP database works.** Geolocation, security, ASN, company, ISP, abuse contact, WHOIS, hosting and residential proxy data all come in MMDB format and use the same filter.
- **IPv4 and IPv6.** Each file covers both address families.
- **Names in 12 languages.** City, region and country names are stored in English, Czech, German, Spanish, Persian, French, Italian, Japanese, Korean, Portuguese, Russian and Chinese.
- **Fresh data without downtime.** Databases are updated daily or weekly, depending on your plan, and Fluent Bit can load a new file without a restart.
- **Private and predictable.** Lookups happen locally. IP addresses in your logs never leave your network, and there is no per-request cost.

## Common use cases

- **Security monitoring.** Flag logins and API calls from VPNs, proxies and Tor, and send them to your SIEM.
- **Fraud and abuse prevention.** Spot sign-ups and checkouts from anonymizing networks or hosting providers.
- **Traffic analytics.** Chart requests by country, region and city in Grafana, Kibana or OpenSearch Dashboards.
- **Incident response.** See which network an attacking address belongs to, and who to contact about it.
- **Compliance reporting.** Show where your traffic comes from, using data that never leaves your infrastructure.

---

## What is Fluent Bit?

[Fluent Bit](https://fluentbit.io) is a lightweight, open source log and metrics processor and forwarder, and a graduated project of the Cloud Native Computing Foundation. It collects data from files, containers, systemd and network inputs, transforms it with filters, and sends it to destinations such as Elasticsearch, OpenSearch, Grafana Loki, Splunk, Kafka and Amazon S3. It often runs as a DaemonSet in Kubernetes or as an agent on each server.

[Enriching logs with IP data](https://ipgeolocation.io/guides/what-is-ip-enrichment-and-why-logs-need-it) inside Fluent Bit means the data reaches your backend already enriched. Dashboards, alerts and queries can use the country or the VPN flag directly, with no lookup at query time.

---

## How it works

1. When Fluent Bit starts, each `geoip2` filter opens one MMDB file.
2. For each record that matches the filter, it reads the IP address from the field named in `lookup_key`.
3. It looks the address up and, for each `record` rule, copies one value from the database into a new key.
4. The record continues to the next filter or output.

Each filter reads one file, so you add one `geoip2` filter per database file.

### What is a path?

An MMDB file stores a nested record for each IP range. A path names one value inside that record, with a dot between each level. Here is part of the record for `37.120.202.92` in the IP Geolocation Database:

```json
{
  "location": {
    "city": {
      "name": {
        "en": "Secaucus",
        "de": "Secaucus"
      }
    },
    "coordinates": {
      "latitude": "40.78834",
      "longitude": "-74.05502"
    },
    "country": {
      "code2": "US",
      "name": {
        "en": "United States"
      }
    },
    "state": {
      "name": {
        "en": "New Jersey"
      }
    }
  },
  "time_zone": "America/New_York"
}
```

The path `location.country.code2` leads to `"US"`, and `location.city.name.en` leads to `"Secaucus"`. In a Fluent Bit rule, you wrap the path in `%{...}`.

---

## Requirements

| Component | Version |
| --- | --- |
| Fluent Bit | 5.1.3 (tested). The official container images and packages include the `geoip2` filter. |
| IPGeolocation.io databases | Any IP database in MMDB format. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or visit [IP database pricing](https://ipgeolocation.io/db-pricing.html) to choose a plan. |
| Your logs | Records that carry the client IP address in a field. The examples use a field named `remote_addr`. |

---

## Quick start

This walkthrough runs Fluent Bit in Docker with a test record, so you can see the enrichment working in a few minutes. It uses the IP Geolocation, IP Security and IP to ASN databases.

### Step 1: Put the databases in a folder

Download the MMDB files from your account and place them in a folder named `databases`:

```text
databases/
├── db-ip-asn.mmdb
├── db-ip-location.mmdb
└── db-ip-security.mmdb
```

### Step 2: Create the configuration

Save this as `fluent-bit.yaml` next to the `databases` folder. It creates one test record and adds location, security and ASN data to it:

```yaml
service:
  flush: 1
  log_level: warn

pipeline:
  inputs:
    - name: dummy
      tag: web
      dummy: '{"remote_addr": "37.120.202.92", "method": "POST", "path": "/login"}'
      samples: 1

  filters:
    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-location.mmdb
      lookup_key: remote_addr
      record:
        - country_code remote_addr %{location.country.code2}
        - country_name remote_addr %{location.country.name.en}
        - region remote_addr %{location.state.name.en}
        - city remote_addr %{location.city.name.en}
        - latitude remote_addr %{location.coordinates.latitude}
        - longitude remote_addr %{location.coordinates.longitude}
        - time_zone remote_addr %{time_zone}

    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-security.mmdb
      lookup_key: remote_addr
      record:
        - threat_score remote_addr %{threat_score}
        - is_vpn remote_addr %{is_vpn}
        - is_proxy remote_addr %{is_proxy}
        - is_tor remote_addr %{is_tor}
        - is_anonymous remote_addr %{is_anonymous}

    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-asn.mmdb
      lookup_key: remote_addr
      record:
        - asn remote_addr %{asn.as_number}
        - as_organization remote_addr %{asn.organization}

  outputs:
    - name: stdout
      match: '*'
      format: json_lines
```

### Step 3: Run Fluent Bit

```sh
docker run --rm \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/fluent-bit.yaml":/fluent-bit/etc/fluent-bit.yaml:ro \
  fluent/fluent-bit:5.1.3 -c /fluent-bit/etc/fluent-bit.yaml
```

### Step 4: Check the output

Fluent Bit prints one JSON line per record. Here it is formatted for readability:

```json
{
  "date": 1791266086.766181,
  "remote_addr": "37.120.202.92",
  "method": "POST",
  "path": "/login",
  "country_code": "US",
  "country_name": "United States",
  "region": "New Jersey",
  "city": "Secaucus",
  "latitude": "40.78834",
  "longitude": "-74.05502",
  "time_zone": "America/New_York",
  "threat_score": 50,
  "is_vpn": "true",
  "is_proxy": "true",
  "is_tor": "false",
  "is_anonymous": "true",
  "asn": "9009",
  "as_organization": "M247 Europe SRL"
}
```

The test address is a VPN exit, so `is_vpn` is `"true"`. Values depend on your database release, and the `date` key comes from the `stdout` output. To see the network behind the address, with its routes, peers and WHOIS data, open [AS9009 (M247 Europe SRL) in the IPGeolocation.io ASN browser](https://ipgeolocation.io/browse/asn/AS9009).

Once this works, replace the `dummy` input and `stdout` output with your real input and output. The [access log](#enriching-a-web-server-access-log) example further down shows a typical setup.

---

## Configuration reference

### Filter settings

| Setting | What it does |
| --- | --- |
| `name` | Always `geoip2`. |
| `match` | The tag of the records to enrich, such as `web` or `nginx.*`. Records with other tags pass through unchanged. |
| `database` | Path to the MMDB file. In Docker, this is the path inside the container. |
| `lookup_key` | The record field that holds the IP address. |
| `record` | One rule per new key. A filter can have as many rules as you need. |

### Record rules

Each `record` rule has three parts:

```text
<new key> <field that holds the IP> %{<path in the database>}
```

For example, `country_code remote_addr %{location.country.code2}` reads the IP from `remote_addr`, looks up `location.country.code2`, and stores the result in a new key named `country_code`. The field in the rule is normally the same as `lookup_key`.

### Paths in each database

Every IPGeolocation.io IP database works with the filter. Each one stores its values under its own paths, and location data uses the same paths in every database that includes it.

| Database | Paths |
| --- | --- |
| IP Geolocation Database and IP to City Database | Under `location.`, such as `location.country.code2`, plus `time_zone` |
| IP to Country Database | Under `location.country.`, such as `location.country.code2` |
| IP Security Database | At the top level, such as `is_vpn` and `threat_score` |
| IP to ASN Database | Under `asn.`, such as `asn.as_number` |
| IP to Company Database | Under `company.`, such as `company.name.en` |
| IP to ISP Database | At the top level: `isp`, `asn`, `as_organization` |
| IP Abuse Contact Database | Under `abuse.`, such as `abuse.emails` |
| IP WHOIS Database | Under `whois.`, such as `whois.organization.name` |
| IP to Hosting Database | At the top level: `hosting_provider` |
| Residential Proxy Database | At the top level: `proxy_provider` and `last_seen` |

Two details to watch with ISP data:

- `asn` is the AS number itself (`%{asn}`), not a group (`%{asn.as_number}`).
- In the IP to ISP Database, the country sits at the top level (`%{country.code2}`), not under `location.`.

ISP and ASN fields describe different things. See [how an ISP differs from an ASN](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn).

### More security fields

The IP Security Database has more fields than the quick start uses. Add any of these to the security filter in the same way:

| Path | Meaning |
| --- | --- |
| `is_residential_proxy` | [Residential proxy](https://ipgeolocation.io/residential-proxy-database.html) |
| `is_relay` | Relay service, such as iCloud Private Relay |
| `is_known_attacker` | Known source of attacks |
| `is_bot` | Automated traffic |
| `is_spam` | Known spam source |
| `is_cloud_provider` | Address belongs to a cloud provider |
| `vpn_confidence_score` | Confidence in the VPN result, 0 to 100 |
| `proxy_confidence_score` | Confidence in the proxy result, 0 to 100 |
| `cloud_provider_name` | Name of the cloud provider |

See the [field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) for every path, and the [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html) for what each field means.

### Reading a combined database

Some plans deliver several databases in one file, such as location, company and ASN data together. The file keeps each database's paths, so one filter can read all of them:

```yaml
    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-city-company-asn.mmdb
      lookup_key: remote_addr
      record:
        - country_code remote_addr %{location.country.code2}
        - city remote_addr %{location.city.name.en}
        - asn remote_addr %{asn.as_number}
        - company remote_addr %{company.name.en}
```

### Classic configuration format

If you use the classic `.conf` format, the same filter looks like this:

```ini
[FILTER]
    Name       geoip2
    Match      web
    Database   /usr/local/share/ipgeolocation/db-ip-location.mmdb
    Lookup_key remote_addr
    Record     country_code remote_addr %{location.country.code2}
    Record     city         remote_addr %{location.city.name.en}
```

---

## Enriching a web server access log

In production, Fluent Bit usually reads a log file. This configuration tails an Nginx access log, parses it with Fluent Bit's built-in `nginx` parser, and enriches each request. The parser puts the client address in a field named `remote`, so that is the `lookup_key`.

```yaml
service:
  flush: 1
  log_level: warn
  parsers_file: /fluent-bit/etc/parsers.conf

pipeline:
  inputs:
    - name: tail
      path: /var/log/nginx/access.log
      tag: nginx.access
      parser: nginx

  filters:
    - name: geoip2
      match: nginx.*
      database: /usr/local/share/ipgeolocation/db-ip-location.mmdb
      lookup_key: remote
      record:
        - country_code remote %{location.country.code2}
        - city remote %{location.city.name.en}

  outputs:
    - name: stdout
      match: '*'
      format: json_lines
```

`/fluent-bit/etc/parsers.conf` is the path in the official container image. Package installs keep it at `/etc/fluent-bit/parsers.conf`. In Docker, also mount the log directory into the container.

For this log line:

```bash
37.120.202.92 - - [06/Oct/2026:10:15:32 +0000] "POST /login HTTP/1.1" 401 512 "-" "Mozilla/5.0"
```

Fluent Bit prints (formatted for readability):

```json
{
  "date": 1791281732.0,
  "remote": "37.120.202.92",
  "host": "-",
  "user": "-",
  "method": "POST",
  "path": "/login",
  "code": "401",
  "size": "512",
  "referer": "-",
  "agent": "Mozilla/5.0",
  "country_code": "US",
  "city": "Secaucus"
}
```

If your server sits behind a load balancer or CDN, the first field in the log is the proxy's address, not the visitor's. Configure the server to log the client address, for example with Nginx's `real_ip` module. See [how to get the real client IP address behind a proxy](https://ipgeolocation.io/guides/get-real-client-ip-address).

---

## Routing VPN, proxy and Tor traffic

Enrichment becomes more useful when you act on it. This example sends records from anonymizing networks to their own destination, such as your SIEM, and leaves the rest of the traffic on its normal path. Add the `rewrite_tag` filter after the security filter:

```bash
  filters:
    # ... the geoip2 filters from the quick start ...

    - name: rewrite_tag
      match: web
      rule: $is_anonymous ^true$ security.anonymous false

  outputs:
    - name: stdout            # normal traffic
      match: web
    - name: stdout            # VPN, proxy and Tor traffic; replace with your SIEM output
      match: security.*
```

The rule reads: when `is_anonymous` is `true`, give the record the tag `security.anonymous`. The final `false` means the record is not also kept under the original tag. To route on one signal only, use another flag, such as `$is_tor ^true$` or `$is_vpn ^true$`.

The flags are strings, so match them with the `^true$` pattern shown above.

---

## Working with the values

| Value | Type | Example | Notes |
| --- | --- | --- | --- |
| Security flags (`is_vpn`, `is_tor`, ...) | String | `"true"` | Compare as strings, not booleans. |
| Scores (`threat_score`, confidence scores) | Integer | `50` | 0 to 100. |
| Coordinates | String | `"40.78834"` | Convert to numbers for geo points (below). |
| AS number | String | `"9009"` | No `AS` prefix. |
| Provider names | List | `["Private Internet Access VPN"]` | Read the first entry with `.0` (below). |
| Missing value | Empty string | `""` | The database has no value for that address. |
| Address not in the database | `null` | `null` | Every key from that filter is `null`. |

**Coordinates as numbers.** Fluent Bit's `type_converter` filter converts the coordinate strings to numbers, which most backends need for a geo point. Add it after the location filter:

```yaml
    - name: type_converter
      match: web
      str_key:
        - latitude lat float
        - longitude lon float
```

**Lists.** The filter cannot copy a whole list: it writes `null` and logs `Not supported MAP and ARRAY`. To get the first provider name, use `%{vpn_provider_names.0}`. Most addresses have an empty list, and the filter logs a warning for each of those records, so add list fields only when you need them.

**Languages.** Replace `.en` in a name path with `cs`, `de`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` or `zh`. A name without a translation is an empty string.

---

## Running in production

- **Enrich only what you need.** Set `match` to the tags that carry client IP addresses, and add only the fields your dashboards and alerts use.
- **Use the real client IP.** Behind a proxy, load balancer or CDN, look up the visitor's address, not the proxy's.
- **Keep the logs quiet.** A wrong path logs a warning for every record. Test new rules with the quick start before you deploy them.
- **Kubernetes.** Put the databases in a volume and mount it read-only at the same path in every Fluent Bit pod. After you update the files, reload Fluent Bit or restart the DaemonSet with `kubectl rollout restart daemonset/<name>`.
- **Keep the databases current.** Automate updates as shown in the next section.

---

## Keeping the databases up to date

IPGeolocation.io updates the databases daily or weekly, depending on your plan. Fluent Bit opens each file when it starts and keeps reading that copy, so a new file on disk takes effect only after a reload or restart.

To reload without stopping Fluent Bit, turn on hot reload:

```yaml
service:
  hot_reload: on
```

Fluent Bit then reopens its files when it receives `SIGHUP` (`kill -HUP <pid>`, or `docker kill --signal HUP <container>` in Docker).

This script installs a release safely. A release arrives as a ZIP archive with the MMDB file of each database in your plan, a `README.md`, and a `checksum.txt` that lists a SHA-256 hash for every file. The script unpacks the archive beside the live files, refuses it if a hash or a database check fails, moves the new MMDB files into place and sends Fluent Bit a `SIGHUP`. Set `DOWNLOAD_URL` to the MMDB download link in your IPGeolocation.io account. Besides `curl`, the script needs `unzip`, `sha256sum` and the [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio):

```sh
#!/bin/sh
# Install a new IPGeolocation.io database release and reload Fluent Bit.
set -eu

DB_DIR=/usr/local/share/ipgeolocation
DOWNLOAD_URL="<MMDB download link from your IPGeolocation.io account>"

# 1. Unpack the release in a temporary folder beside the live databases.
WORK=$(mktemp -d "$DB_DIR/.release.XXXXXX")
trap 'rm -rf "$WORK"' EXIT
curl -fsSL -o "$WORK/release.zip" "$DOWNLOAD_URL"
# -DD stamps the unpacked files with the current time, not the archive's.
unzip -q -DD "$WORK/release.zip" -d "$WORK"
rm "$WORK/release.zip"

# 2. Refuse the release if any file fails its checksum or any database is damaged.
(cd "$WORK" && sha256sum --quiet -c checksum.txt)
for db in "$WORK"/*.mmdb; do
    mmdbio verify --db "$db"
done

# 3. Move the new databases over the old ones. Each rename is atomic.
for db in "$WORK"/*.mmdb; do
    mv "$db" "$DB_DIR/"
done

# 4. Make Fluent Bit reopen its files (requires hot_reload: on).
kill -HUP "$(pidof fluent-bit)"
```

The file names inside the archive stay the same from release to release, so the `database` paths in your configuration never change. After a failed check, the script exits with an error and removes its temporary folder, and Fluent Bit carries on with the files it already has. Run it from cron on your plan's release schedule.

In Docker, mount the directory that holds the databases, as in the quick start, not the individual files. A file mounted on its own keeps pointing at the old copy after the rename.

---

## Troubleshooting

**No new keys appear.** The record's tag does not match the filter's `match` setting, so the filter skips it. Check the tag your input sets.

**Every key is `null`.** The address is not in that database, or the record has no `lookup_key` field. To see what a database holds for an address, use [mmdbio](https://ipgeolocation.io/cli/mmdbio):

```sh
mmdbio read --db /usr/local/share/ipgeolocation/db-ip-location.mmdb --ip 37.120.202.92
```

**`cannot get value: The lookup path does not match the data`.** The path does not exist in that database. Check the spelling, and check that the filter points at the right database. For example, `%{asn.as_number}` works with the IP to ASN Database but not with the IP to ISP Database, where the path is `%{asn}`.

**`Not supported MAP and ARRAY`.** The path stops at a group or a list instead of a single value. For example, `%{location.country.name}` needs a language at the end: `%{location.country.name.en}`.

**`getaddrinfo failed: Name or service not known`.** The lookup field does not hold a single IP address. This happens with `X-Forwarded-For` values such as `203.0.113.7, 10.0.0.1`, or with an address that includes a port. Extract the client address into its own field first.

**Fluent Bit stops with `Cannot open geoip2 database`.** The file in `database` does not exist or is not readable. In Docker, the path must be the path inside the container.

**New data does not show up after an update.** Fluent Bit still has the old file open. Reload it with `SIGHUP` (with `hot_reload: on`) or restart it.

---

## FAQ

<details>
<summary><strong>Do I need an API key?</strong></summary>
No. Fluent Bit reads the database files directly. You only need an IPGeolocation.io database plan to download the files.
</details>

<details>
<summary><strong>Does Fluent Bit send IP addresses to IPGeolocation.io?</strong></summary>
No. Every lookup happens on your own machine, and nothing is sent over the network.
</details>

<details>
<summary><strong>Which IPGeolocation.io databases work with Fluent Bit?</strong></summary>
All IP databases in MMDB format, including combined databases. The [Paths in each database](#paths-in-each-database) table above lists the paths to use with each one.
</details>

<details>
<summary><strong>Does it work with IPv6 addresses?</strong></summary>
Yes. Each database covers IPv4 and IPv6, and the filter looks up both.
</details>

<details>
<summary><strong>How often are the databases updated?</strong></summary>
Daily or weekly, depending on your plan. Fluent Bit picks up a new file after a reload or restart.
</details>

<details>
<summary><strong>Can I use more than one database in the same pipeline?</strong></summary>
Yes. Add one `geoip2` filter per database file, as in the quick start, or use a combined database and read every value with one filter.
</details>

---

## Related

- [IPGeolocation.io MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md)
- [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html)
- [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html)
- [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html)
- [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html)
- [Enrich logs with IPGeolocation.io in Vector](https://ipgeolocation.io/documentation/vector-integration)
- [Enrich logs with IPGeolocation.io in Grafana Alloy](https://ipgeolocation.io/documentation/grafana-alloy-integration)
- [IPGeolocation.io Nginx module for MMDB databases](https://ipgeolocation.io/documentation/nginx-integration)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
