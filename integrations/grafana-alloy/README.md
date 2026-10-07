# Grafana Alloy IP Geolocation and Threat Enrichment for Loki with IPGeolocation.io

## Overview

Grafana Alloy can attach IPGeolocation.io data to log lines before they reach Grafana Loki. A `stage.geoip` block inside `loki.process` matches the client address against a local IPGeolocation.io MMDB file, and later stages store the results as Loki labels or structured metadata. You then filter, chart and alert on country, VPN use or network owner with ordinary LogQL.

This guide pairs three databases. The [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html) supplies country, region, city, coordinates and time zone. The [IP Security Database](https://ipgeolocation.io/ip-security-database.html) supplies anonymity signals such as VPN, proxy and Tor flags, plus a threat score. The [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html) supplies the [AS number](https://ipgeolocation.io/guides/what-is-an-asn) and the organization that runs the network.

Alloy is not limited to those three. `stage.geoip` reads every IPGeolocation.io IP database: the IP to Country, IP to City and IP to ASN databases, the [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP WHOIS Database](https://ipgeolocation.io/ip-whois-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html), as well as combined databases that merge several of them into one file.

Because the databases are files on the Alloy host, enriching a line needs no network call and adds no usage charge, and client addresses stay in your environment. [Comparing an IP geolocation API with a database](https://ipgeolocation.io/guides/ip-geolocation-api-vs-database-guide) explains when each one fits.

> [!TIP]
> Leave `db_type` out of `stage.geoip`, and list the fields you want in `custom_lookups` using IPGeolocation.io paths such as `location.country.code2`. The [MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) has every available path.

---

## What you gain in Loki

- **Queries by country, VPN use or network.** Select streams with a country label, then narrow them with `| is_vpn="true"` or `| asn="9009"` on structured metadata.
- **A small index.** Only the country code becomes a label. Everything else travels as structured metadata, so the number of Loki streams barely grows.
- **Panels and alerts straight from LogQL.** Metric queries such as requests per country feed Grafana dashboards and alert rules with no extra processing.
- **Room to grow.** Location, security, ASN, company, ISP, abuse contact, WHOIS, hosting and residential proxy files all plug into the same stage.
- **Both IP versions, twelve languages.** Lookups accept IPv4 and IPv6 addresses. Place names are stored in Chinese, Czech, English, French, German, Italian, Japanese, Korean, Persian, Portuguese, Russian and Spanish.
- **Updates while Alloy keeps running.** A small pointer file lets Alloy move to a new database release without a restart.
- **Nothing leaves your network.** Addresses are matched on the Alloy host, with no API calls and no per-lookup fees.

## What teams do with it

- **Catch account takeover attempts.** Alert when failed logins from VPN, proxy or Tor exits rise above normal.
- **Investigate abuse.** Pull every line from one AS number or hosting network during an incident, and look up the network's abuse contact in the IP Abuse Contact Database.
- **Understand traffic.** Build a Grafana panel of requests per country without touching your application.
- **Answer data residency questions.** Show auditors which countries your users connect from, using data that stays in your own Loki.

---

## What is Grafana Alloy?

[Grafana Alloy](https://grafana.com/oss/alloy-opentelemetry-collector/) is Grafana Labs' open source telemetry collector. It is an OpenTelemetry Collector distribution with built-in Prometheus pipelines and native support for [Grafana Loki](https://grafana.com/oss/loki/), and it collects metrics, logs, traces and profiles. Grafana's documentation also describes Alloy as the migration path for Promtail and Grafana Agent users.

For logs, Alloy reads files, containers and other sources, processes each line in a `loki.process` component, and sends the result to Loki. [Adding IP context to logs](https://ipgeolocation.io/guides/what-is-ip-enrichment-and-why-logs-need-it) inside that component means every line arrives in Loki ready to filter, chart and alert on.

---

## How it works

1. A stage such as `stage.json` or `stage.regex` extracts the client IP address from the log line into a named value, for example `client_ip`.
2. Each `stage.geoip` block opens one MMDB file, looks up the address in `source`, and stores one value for each entry in `custom_lookups`.
3. `stage.labels` turns chosen values into Loki labels, and `stage.structured_metadata` attaches the rest to the log line.
4. The line continues to `loki.write`, which sends it to Loki.

Each `stage.geoip` block reads one file, so you add one block per database file.

### Reading a database record

Records in an MMDB file are nested objects. To reach a single value, list the keys that lead to it, joined by dots. Below is an excerpt of what the IP Geolocation Database holds for `37.120.202.92`:

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

Following `location`, then `country`, then `code2` gives `"US"`. Following `location`, `city`, `name` and `en` gives `"Secaucus"`. Alloy treats each path as a JMESPath expression, which also allows list indexes and functions.

---

## Requirements

| Component | Version |
| --- | --- |
| Grafana Alloy | v1.20.1 (tested). `stage.geoip` is built in. |
| Grafana Loki | 3.x (tested with 3.7.8). Structured metadata needs the TSDB index with schema v13, and is on by default. |
| IPGeolocation.io databases | The MMDB edition of each database you plan to read. Get the files from your [IPGeolocation.io account](https://app.ipgeolocation.io); plans and prices are on the [IP database pricing page](https://ipgeolocation.io/db-pricing.html). |
| Your logs | Log lines that carry the client IP address. The [quick start](#quick-start) uses JSON lines with a `remote_addr` field. |

---

## Quick start

This walkthrough runs Alloy in Docker against a one-line log file and prints the enriched entry, so you can see the enrichment working in a few minutes without Loki. The pipeline reads location, security and ASN data from three database files.

### Step 1: Prepare the files

Place the three MMDB files from your account in a `databases` folder, and create a `logs` folder with one test line:

```text
databases/
├── db-ip-asn.mmdb
├── db-ip-location.mmdb
└── db-ip-security.mmdb
logs/
└── app.log
```

```sh
mkdir -p logs
printf '{"remote_addr":"37.120.202.92","method":"POST","path":"/login"}\n' > logs/app.log
```

### Step 2: Create the configuration

Save this as `config.alloy`. It reads the log file, adds location, security and ASN data, and prints each entry with `loki.echo`:

```alloy
local.file_match "app" {
  path_targets = [{"__path__" = "/var/log/app/*.log", "job" = "app"}]
}

loki.source.file "app" {
  targets       = local.file_match.app.targets
  forward_to    = [loki.process.ipgeo.receiver]
  tail_from_end = false
}

loki.process "ipgeo" {
  forward_to = [loki.echo.out.receiver]

  stage.json {
    expressions = { client_ip = "remote_addr" }
  }

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-location.mmdb"
    custom_lookups = {
      "country_code" = "location.country.code2",
      "country_name" = "location.country.name.en",
      "region"       = "location.state.name.en",
      "city"         = "location.city.name.en",
      "latitude"     = "location.coordinates.latitude",
      "longitude"    = "location.coordinates.longitude",
      "time_zone"    = "time_zone",
    }
  }

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-security.mmdb"
    custom_lookups = {
      "threat_score" = "threat_score",
      "is_vpn"       = "is_vpn",
      "is_proxy"     = "is_proxy",
      "is_tor"       = "is_tor",
      "is_anonymous" = "is_anonymous",
    }
  }

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-asn.mmdb"
    custom_lookups = {
      "asn"             = "asn.as_number",
      "as_organization" = "asn.organization",
    }
  }

  stage.labels {
    values = { country_code = "" }
  }

  stage.structured_metadata {
    values = {
      country_name    = "",
      region          = "",
      city            = "",
      latitude        = "",
      longitude       = "",
      time_zone       = "",
      threat_score    = "",
      is_vpn          = "",
      is_proxy        = "",
      is_tor          = "",
      is_anonymous    = "",
      asn             = "",
      as_organization = "",
    }
  }
}

loki.echo "out" { }
```

### Step 3: Run Alloy

```sh
docker run --rm \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/logs":/var/log/app:ro \
  -v "$PWD/config.alloy":/etc/alloy/config.alloy:ro \
  grafana/alloy:v1.20.1 run /etc/alloy/config.alloy --storage.path=/tmp/alloy
```

### Step 4: Check the output

Alloy logs a `received log entry` line for each entry. These are its labels:

```text
{country_code="US", filename="/var/log/app/app.log", job="app"}
```

And this is its structured metadata, formatted for readability:

```json
{
  "as_organization": "M247 Europe SRL",
  "asn": "9009",
  "city": "Secaucus",
  "country_name": "United States",
  "is_anonymous": "true",
  "is_proxy": "true",
  "is_tor": "false",
  "is_vpn": "true",
  "latitude": "40.78834",
  "longitude": "-74.05502",
  "region": "New Jersey",
  "threat_score": "50",
  "time_zone": "America/New_York"
}
```

`37.120.202.92` is a commercial VPN endpoint in a New Jersey data center, which is why `is_vpn` and `is_anonymous` read `"true"`. Your values may differ as the data is updated. For the full picture of that network, see [AS9009's routes, peers and WHOIS records](https://ipgeolocation.io/browse/asn/AS9009).

Once this works, point `local.file_match` at your real logs and replace `loki.echo` with `loki.write`, as shown in [Sending the data to Loki](#sending-the-data-to-loki) below.

---

## Configuration reference

### `stage.geoip` settings

| Setting | What it does |
| --- | --- |
| `source` | The extracted value that holds the IP address, such as `client_ip`. |
| `db` | Location of the MMDB file. When Alloy runs in a container, give the container's path. |
| `custom_lookups` | One entry per new value: the name to store it under, and the path to read. |
| `db_type` | Leave it unset. It reads a different database layout and returns no values with IPGeolocation.io files. |

### Custom lookups

Each entry in `custom_lookups` has two parts:

```text
"<name of the new value>" = "<path in the database>",
```

For example, `"country_code" = "location.country.code2"` looks up `location.country.code2` and stores the result as `country_code`. A later stage decides where the value goes: `stage.labels` makes it a label, and `stage.structured_metadata` attaches it to the line.

### A lookup for each database

Any IPGeolocation.io IP database can sit behind a `stage.geoip` block. Here is one sample `custom_lookups` entry per database:

| Database | Sample entry |
| --- | --- |
| IP Geolocation Database or IP to City Database | `"city" = "location.city.name.en"` |
| IP to Country Database | `"country_code" = "location.country.code2"` |
| IP Security Database | `"is_vpn" = "is_vpn"` |
| IP to ASN Database | `"asn" = "asn.as_number"` |
| IP to Company Database | `"company" = "company.name.en"` |
| IP to ISP Database | `"isp" = "isp"` and `"asn" = "asn"` |
| IP Abuse Contact Database | `"abuse_email" = "abuse.emails"` |
| IP WHOIS Database | `"rir" = "whois.rir"` |
| IP to Hosting Database | `"hosting_provider" = "hosting_provider"` |
| Residential Proxy Database | `"proxy_provider" = "proxy_provider"` |

ISP data differs from ASN data in two ways. In the IP to ISP Database, `asn` holds the number directly rather than a group of fields, and the country code is `country.code2` at the top level. For background, read [the difference between an ISP and an ASN](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn).

### Extra security signals

Beyond the five flags in the [quick start](#quick-start), the IP Security Database can feed a second set of signals. Add this block after the first security stage, and list the new names in `stage.structured_metadata`:

```alloy
  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-security.mmdb"
    custom_lookups = {
      "is_residential_proxy"   = "is_residential_proxy",
      "is_relay"               = "is_relay",               // such as iCloud Private Relay
      "is_known_attacker"      = "is_known_attacker",
      "is_bot"                 = "is_bot",
      "is_spam"                = "is_spam",
      "is_cloud_provider"      = "is_cloud_provider",
      "cloud_provider_name"    = "cloud_provider_name",
      "vpn_confidence_score"   = "vpn_confidence_score",   // 0 to 100
      "proxy_confidence_score" = "proxy_confidence_score", // 0 to 100
    }
  }
```

The [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html) defines each signal, including [residential proxy detection](https://ipgeolocation.io/residential-proxy-database.html), and the field reference lists the remaining paths.

### Reading a combined database

If your plan ships a combined database, such as location, company and ASN data in one file, a single stage covers all of it:

```alloy
  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-city-company-asn.mmdb"
    custom_lookups = {
      "country_code" = "location.country.code2",
      "city"         = "location.city.name.en",
      "asn"          = "asn.as_number",
      "company"      = "company.name.en",
    }
  }
```

### Labels or structured metadata

Loki indexes labels, and every new label value creates a new stream. Keep labels for values with few possible values, and put everything else in structured metadata, which you can still filter on in LogQL.

| Value | Where to put it | Why |
| --- | --- | --- |
| `country_code` | Label | About 250 possible values. Fast stream selection by country. |
| City, region, coordinates, time zone | Structured metadata | Thousands of possible values. |
| Security flags and scores | Structured metadata | Filter them with `\| is_vpn="true"` at query time. |
| AS number and organization | Structured metadata | More than 100,000 possible values. |

---

## Sending the data to Loki

Replace `loki.echo` with `loki.write`, and point `forward_to` at it:

```alloy
loki.process "ipgeo" {
  forward_to = [loki.write.default.receiver]

  // ... the same stages as in the quick start ...
}

loki.write "default" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}
```

Change the URL to your Loki address. For Grafana Cloud, use the push URL and credentials from your stack.

---

## Querying enriched logs in Loki

Once the data is in Loki, filter on the label and the structured metadata in LogQL. These queries work in Grafana Explore, in dashboards and in alert rules.

Requests from one country (label):

```logql
{job="app", country_code="US"}
```

Requests from VPN exits (structured metadata):

```logql
{job="app"} | is_vpn="true"
```

Requests from any anonymizing network, such as a VPN, proxy or Tor:

```logql
{job="app"} | is_anonymous="true"
```

Requests with a threat score of 50 or more:

```logql
{job="app"} | threat_score >= 50
```

Requests per country over the last hour, for a dashboard panel:

```logql
sum by (country_code) (count_over_time({job="app"}[1h]))
```

Anonymous requests per country over five minutes, for an alert rule:

```logql
sum by (country_code) (count_over_time({job="app"} | is_anonymous="true" [5m]))
```

---

## Enriching a web server access log

Nginx and Apache write access logs as plain text, not JSON. Use `stage.regex` to pull the client IP out of each line. This configuration reads the Nginx access log, enriches each request and keeps the HTTP status:

```alloy
local.file_match "nginx" {
  path_targets = [{"__path__" = "/var/log/nginx/access.log", "job" = "nginx"}]
}

loki.source.file "nginx" {
  targets    = local.file_match.nginx.targets
  forward_to = [loki.process.ipgeo.receiver]
}

loki.process "ipgeo" {
  forward_to = [loki.echo.out.receiver]

  stage.regex {
    expression = `^(?P<client_ip>\S+) \S+ \S+ \[[^\]]+\] "(?P<method>\S+) (?P<path>\S+) [^"]*" (?P<status>\d{3})`
  }

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-location.mmdb"
    custom_lookups = {
      "country_code" = "location.country.code2",
      "city"         = "location.city.name.en",
    }
  }

  stage.labels {
    values = { country_code = "" }
  }

  stage.structured_metadata {
    values = { city = "", status = "" }
  }
}

loki.echo "out" { }
```

The expression uses a raw string (between backticks), so the backslashes need no escaping. For the line `37.120.202.92 - - [06/Oct/2026:10:15:32 +0000] "POST /login HTTP/1.1" 401 512 "-" "Mozilla/5.0"`, the entry gets the label `country_code="US"` and the structured metadata `city="Secaucus"` and `status="401"`.

Behind a load balancer or CDN, Nginx records the proxy as the client unless you tell it otherwise. Enable the `real_ip` module, or your proxy's equivalent, so the first field holds the visitor's address. [Finding the real client IP behind a proxy](https://ipgeolocation.io/guides/get-real-client-ip-address) covers the headers involved.

---

## Working with the values

Alloy hands every value to Loki as a string. LogQL can still compare numbers, as the threat score query above shows.

| Value | Arrives as | Example | Notes |
| --- | --- | --- | --- |
| Security flags (`is_vpn`, `is_tor`, ...) | String | `"true"` | Filter with `\| is_vpn="true"`. |
| Scores (`threat_score`, confidence scores) | String | `"50"` | 0 to 100. LogQL compares them as numbers: `\| threat_score >= 50`. |
| Coordinates | String | `"40.78834"` | Decimal degrees. |
| AS number | String | `"9009"` | Digits only. |
| Field with nothing recorded | Empty string | `""` | The stage still adds the value. |
| Address not in the database | No value | | The stage adds nothing for that line. |

**Lists.** Provider names, such as `vpn_provider_names`, are lists. A path that points at a whole list or a group returns no value, without an error. Use a JMESPath expression to turn the list into a string:

```alloy
    custom_lookups = {
      "vpn_provider"  = "vpn_provider_names[0]",
      "vpn_providers" = "join(', ', vpn_provider_names || `[]`)",
    }
```

`vpn_provider_names[0]` returns the first name. The `join` expression returns every name, separated by commas, and the `` || `[]` `` part keeps it from logging an error for addresses that are not in the database.

**Other languages.** Every `name` group holds translations keyed by language code. Swap `en` for `cs`, `de`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` or `zh`; missing translations come back as empty strings.

---

## Running in production

- **Keep labels few.** Make only low-cardinality values, such as `country_code`, into labels. Put everything else in structured metadata.
- **Look up the visitor, not the proxy.** Make sure the field you extract holds the address of the person or system that made the request.
- **Keep a persistent storage path.** Alloy records how far it has read each file under `--storage.path`. Keep that directory across restarts, and Alloy resumes where it stopped instead of reading files again.
- **Kubernetes.** Put the databases in a volume and mount it read-only at the same path in every Alloy pod.
- **Plan for new releases.** Pick one of the two update methods below before you go live.

---

## Keeping the databases up to date

New database releases arrive daily or weekly, depending on your plan. Alloy opens each file when the stage starts and keeps reading that copy. A configuration reload (`POST /-/reload` or `SIGHUP`) does not reopen it while the configuration is unchanged. Use one of these two methods.

### Option 1: Restart Alloy

Unpack and check the release as the script under Option 2 does, move each MMDB file from the archive over the old one, then restart Alloy: `systemctl restart alloy`, `docker restart <container>`, or `kubectl rollout restart daemonset/<name>`. With a persistent `--storage.path`, Alloy resumes reading where it stopped.

### Option 2: Switch files without a restart

Keep each release under a dated file name, and store the name of the current one in a small pointer file. Alloy's `local.file` component watches the pointer file. When its content changes, the `db` path changes, and Alloy rebuilds the pipeline with the new database. It notices the change within seconds, and checks the file at least once a minute.

Read the database name from the pointer file:

```alloy
local.file "security_db" {
  filename = "/usr/local/share/ipgeolocation/db-ip-security.current"
}

loki.process "ipgeo" {
  // ...

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/" + string.trim_space(local.file.security_db.content)
    custom_lookups = {
      "is_vpn"       = "is_vpn",
      "is_anonymous" = "is_anonymous",
    }
  }

  // ...
}
```

The pointer file holds only the name of the current file, not its full path, so the same pointer file works on the host and inside a container.

Releases are delivered as ZIP archives holding the MMDB file of each database in your plan, a `README.md` and a `checksum.txt` of SHA-256 hashes. The file names inside never change, so this script renames each database to a dated name as it installs it. It confirms the hashes and validates every database with the [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio) first, then updates one pointer file per database with an atomic rename, and finally deletes old copies. Set `DOWNLOAD_URL` to the MMDB download link from your IPGeolocation.io account; the script also needs `curl`, `unzip` and `sha256sum`. Run it once before you start Alloy, so the dated files and pointer files exist, then schedule it with cron to match your release cycle:

```sh
#!/bin/sh
# Install a new IPGeolocation.io database release for Grafana Alloy without a restart.
set -eu

DB_DIR=/usr/local/share/ipgeolocation
DOWNLOAD_URL="<MMDB download link from your IPGeolocation.io account>"
STAMP=$(date +%Y%m%d%H%M%S)

# 1. Unpack the release in a temporary folder beside the live databases.
WORK=$(mktemp -d "$DB_DIR/.release.XXXXXX")
trap 'rm -rf "$WORK"' EXIT
curl -fsSL -o "$WORK/release.zip" "$DOWNLOAD_URL"
# -DD stamps the unpacked files with the current time, not the archive's.
unzip -q -DD "$WORK/release.zip" -d "$WORK"
rm "$WORK/release.zip"

# 2. Stop if any file fails its checksum or any database is damaged.
(cd "$WORK" && sha256sum --quiet -c checksum.txt)
for db in "$WORK"/*.mmdb; do
    mmdbio verify --db "$db"
done

# 3. Give each database a dated name and point Alloy at it.
for db in "$WORK"/*.mmdb; do
    name=$(basename "$db" .mmdb)
    mv "$db" "$DB_DIR/$name-$STAMP.mmdb"
    printf '%s\n' "$name-$STAMP.mmdb" > "$DB_DIR/.$name.current.new"
    mv "$DB_DIR/.$name.current.new" "$DB_DIR/$name.current"

    # 4. Delete older copies of this database, keeping the new one and the one before it.
    ls -1t "$DB_DIR/$name"-[0-9]*.mmdb | tail -n +3 | xargs -r rm -f
done
```

A combined plan gets one dated file and one pointer file per database, such as one for the security data and one for the city data; give each its own `local.file` block in the configuration. If a check fails, the script stops before touching any pointer file, and Alloy keeps using the files it has.

With Docker, mount the whole databases folder, as the [quick start](#quick-start) does, so new files and pointer changes are visible inside the container.

---

## Troubleshooting

**No values at all, and no errors.** Check these in order:

- `db_type` is set. Remove it and use `custom_lookups`.
- A path is misspelled, or points at a group or a list instead of a single value. For example, `location.country.name` needs a language at the end: `location.country.name.en`. Alloy skips such lookups without an error.
- The `source` value is empty because the IP was not extracted. Check the `stage.json` or `stage.regex` expression. Alloy only reports this at debug level, as `failed to convert source value to string`.

**Values for some lines but not others.** The database has no entry for the missing addresses. The [mmdbio tool](https://ipgeolocation.io/cli/mmdbio) prints the record a database holds for any address:

```sh
mmdbio read --db /usr/local/share/ipgeolocation/db-ip-location.mmdb --ip 37.120.202.92
```

**`source is not an ip`.** The extracted value is more than one address, such as the `X-Forwarded-For` value `203.0.113.7, 10.0.0.1`, or an address followed by a port. Adjust the extraction so it captures the client address alone.

**`failed to search JMES expression`.** A JMESPath function, such as `join`, received no value because the address is not in the database. Add a default, as in ``join(', ', vpn_provider_names || `[]`)``.

**Alloy fails to start with `invalid stage config open ...: no such file or directory`.** Alloy cannot open the file named in `db`. Check the spelling and permissions, and when Alloy runs in a container, use the path as the container sees it.

**New data does not show up after an update.** Alloy still has the old file open, and a configuration reload does not reopen it. Restart Alloy, or switch files with a pointer file as described above.

---

## FAQ

<details>
<summary><strong>Is an API key needed at runtime?</strong></summary>
No. Alloy reads the database files from disk. An IPGeolocation.io database subscription is what gives you access to download them.
</details>

<details>
<summary><strong>Does Alloy send IP addresses to IPGeolocation.io?</strong></summary>
No. Every lookup happens on your own machine, and nothing is sent to IPGeolocation.io.
</details>

<details>
<summary><strong>Which IPGeolocation.io databases can Alloy read?</strong></summary>
Every IP database in its MMDB edition, combined databases included. [A lookup for each database](#a-lookup-for-each-database) above gives a sample entry for each one.
</details>

<details>
<summary><strong>Are IPv6 addresses supported?</strong></summary>
Yes. The databases cover IPv6 as well as IPv4, and `stage.geoip` accepts addresses of either version.
</details>

<details>
<summary><strong>Should the enriched values be labels or structured metadata?</strong></summary>
Make only low-cardinality values, such as the country code, into labels. Put city, coordinates, security flags, scores and ASN data in structured metadata. You can still filter on them in LogQL.
</details>

<details>
<summary><strong>Does Alloy pick up a new database file automatically?</strong></summary>
Not by itself, and not with a configuration reload. Restart Alloy, or use a pointer file so the `db` path changes, as described in [Keeping the databases up to date](#keeping-the-databases-up-to-date).
</details>

<details>
<summary><strong>How often do new database releases come out?</strong></summary>
Every day or every week, depending on the plan you subscribe to.
</details>

---

## Related

- [IPGeolocation.io MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md)
- [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html)
- [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html)
- [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html)
- [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html)
- [Enrich logs with IPGeolocation.io in Fluent Bit](https://ipgeolocation.io/documentation/fluent-bit-integration)
- [Enrich logs with IPGeolocation.io in Vector](https://ipgeolocation.io/documentation/vector-integration)
- [IPGeolocation.io Nginx module for MMDB databases](https://ipgeolocation.io/documentation/nginx-integration)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
