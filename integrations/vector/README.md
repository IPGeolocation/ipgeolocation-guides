# Vector IP Enrichment with IPGeolocation.io: Geolocation, VPN Detection and ASN Data

## Overview

Vector can look up any IP address in an IPGeolocation.io database while events are in flight. You declare each MMDB file as an enrichment table, call it from a `remap` transform, and get the complete record for that address back as a VRL object. VRL then decides what to keep, what to name it and which type it gets, before the event reaches Elasticsearch, ClickHouse, Datadog, Amazon S3 or any other sink.

The examples in this guide combine three databases: the [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html) for country, region, city, coordinates and time zone, the [IP Security Database](https://ipgeolocation.io/ip-security-database.html) for VPN, proxy and Tor flags and a threat score, and the [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html) for the [autonomous system number (ASN)](https://ipgeolocation.io/guides/what-is-an-asn) and its owner.

Every other IPGeolocation.io IP database works the same way: the IP to Country, IP to City and IP to ISP databases, the [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP WHOIS Database](https://ipgeolocation.io/ip-whois-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html), plus combined databases that pack several of them into one file.

The files sit next to Vector, so enrichment adds no network round trip and no per-event cost, and the IP addresses in your events stay inside your infrastructure. For help choosing, see [when to use an IP database instead of an API](https://ipgeolocation.io/guides/ip-geolocation-api-vs-database-guide).

> [!TIP]
> Declare IPGeolocation.io files as `type: mmdb` enrichment tables. The `geoip` table type rejects them. The [field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) lists every path inside the records.

---

## At a glance

| Topic | Details |
| --- | --- |
| Vector components | One `mmdb` enrichment table per database file, and a `remap` transform |
| What a lookup returns | The whole database record for the address, as a VRL object |
| Where lookups run | In memory on the Vector host, with no API calls |
| What leaves Vector | Clean, typed fields, such as `geo`, `security` and `network` objects |
| Tested with | Vector 0.58.0 |

## Why Vector suits IPGeolocation.io data

- **One lookup, the whole record.** Each call returns every field for the address. Copy one value, a group of values, or the entire record.
- **Real types before storage.** VRL turns the `"true"` and `"false"` flags into booleans and the coordinate strings into floats, so your sink stores typed values.
- **Lists stay lists.** VPN and proxy provider names arrive as arrays, ready for array fields in your backend.
- **Failed lookups cannot slip through.** VRL refuses to load a lookup whose error is not handled, so an unknown address never breaks the pipeline unnoticed.
- **Tests live with the config.** `vector test` checks your enrichment logic before you deploy it.
- **Updates without a restart.** A `SIGHUP` makes Vector reload the database files.

## Where it helps

- **Maps in Elasticsearch and OpenSearch.** Build a `geo.location` object with numeric `lat` and `lon`, ready for a `geo_point` field.
- **A separate stream for risky traffic.** Route events from VPNs, proxies and Tor to your SIEM, and send everything else to cheaper storage.
- **Typed columns in ClickHouse or BigQuery.** Store flags as booleans and AS numbers as integers, with no casting at query time.
- **Abuse handling.** Attach the network owner, or the abuse contact from the IP Abuse Contact Database, to events that hit your rate limits.

---

## What is Vector?

[Vector](https://vector.dev) is an open source tool for building observability pipelines, written in Rust. It collects logs and metrics from sources such as files, Kubernetes, syslog and Kafka, reshapes them with VRL (Vector Remap Language), and delivers them to dozens of sinks.

An enrichment table lets a VRL program look up reference data, such as an IPGeolocation.io database, for every event. That is how [IP enrichment for logs](https://ipgeolocation.io/guides/what-is-ip-enrichment-and-why-logs-need-it) happens inside Vector, before the data is stored.

---

## How the lookup works

1. At startup, Vector loads each `mmdb` enrichment table into memory.
2. In a `remap` transform, `get_enrichment_table_record` looks up the event's IP address.
3. The call returns the record for the address, or an error when the address is not in the file or is not a valid IP.
4. VRL copies and converts the fields you want onto the event, and the event moves on to its sinks.

### The record Vector returns

This is the complete IP Security Database record for `37.120.202.92`, a VPN exit, as VRL receives it:

```json
{
  "bot_confidence_score": 0,
  "bot_last_seen": "",
  "bot_operator_name": "",
  "bot_type": "",
  "cloud_provider_name": "M247",
  "corporate_gateway_provider_name": "",
  "corporate_gateway_type": "",
  "is_anonymous": "true",
  "is_bot": "false",
  "is_cloud_provider": "true",
  "is_corporate_gateway": "false",
  "is_known_attacker": "false",
  "is_known_good_bot": "false",
  "is_proxy": "true",
  "is_relay": "false",
  "is_residential_proxy": "false",
  "is_spam": "false",
  "is_tor": "false",
  "is_vpn": "true",
  "proxy_confidence_score": 99,
  "proxy_last_seen": "2026-09-08",
  "proxy_provider_names": [],
  "relay_provider_name": "",
  "threat_score": 50,
  "vpn_confidence_score": 99,
  "vpn_last_seen": "2026-09-28",
  "vpn_provider_names": [
    "Private Internet Access VPN"
  ]
}
```

Flags are strings, scores are integers, and provider names are lists. In VRL, you reach a value with a field path on the returned object. If the record is stored in `sec`, the VPN flag is `sec.is_vpn`. Location records nest deeper: the country code in the IP Geolocation Database is `loc.location.country.code2`.

---

## Requirements

| Component | Details |
| --- | --- |
| Vector | 0.58.0 (tested), with the `mmdb` enrichment table type. Check with `vector list`. |
| IPGeolocation.io databases | One or more IP databases in MMDB format, downloaded from your [IPGeolocation.io account](https://app.ipgeolocation.io). To pick a plan, see [IP database pricing](https://ipgeolocation.io/db-pricing.html). |
| Your events | A field that holds the client IP address. The quick start uses `remote_addr`. |

---

## Quick start

Pipe one JSON event through Vector in Docker and watch it come out enriched. No log files or sinks are needed.

### Step 1: Collect the database files

From your account, download the three MMDB files into a folder named `databases`:

```text
databases/
├── db-ip-asn.mmdb
├── db-ip-location.mmdb
└── db-ip-security.mmdb
```

### Step 2: Write the pipeline

Save this as `vector.yaml` next to the `databases` folder:

```yaml
enrichment_tables:
  ipgeo_location:
    type: mmdb
    path: /usr/local/share/ipgeolocation/db-ip-location.mmdb
  ipgeo_security:
    type: mmdb
    path: /usr/local/share/ipgeolocation/db-ip-security.mmdb
  ipgeo_asn:
    type: mmdb
    path: /usr/local/share/ipgeolocation/db-ip-asn.mmdb

sources:
  app:
    type: stdin
    decoding:
      codec: json

transforms:
  ipgeo:
    type: remap
    inputs: [app]
    source: |
      loc, err = get_enrichment_table_record("ipgeo_location", {"ip": .remote_addr})
      if err == null {
        .geo.country_code = loc.location.country.code2
        .geo.country_name = loc.location.country.name.en
        .geo.region = loc.location.state.name.en
        .geo.city = loc.location.city.name.en
        .geo.time_zone = loc.time_zone
        .geo.location.lat = to_float(loc.location.coordinates.latitude) ?? null
        .geo.location.lon = to_float(loc.location.coordinates.longitude) ?? null
      }

      sec, err = get_enrichment_table_record("ipgeo_security", {"ip": .remote_addr})
      if err == null {
        .security.threat_score = sec.threat_score
        .security.is_vpn = sec.is_vpn == "true"
        .security.is_proxy = sec.is_proxy == "true"
        .security.is_tor = sec.is_tor == "true"
        .security.is_anonymous = sec.is_anonymous == "true"
        .security.vpn_providers = sec.vpn_provider_names
      }

      net, err = get_enrichment_table_record("ipgeo_asn", {"ip": .remote_addr})
      if err == null {
        .network.asn = to_int(net.asn.as_number) ?? null
        .network.organization = net.asn.organization
      }

sinks:
  out:
    type: console
    inputs: [ipgeo]
    encoding:
      codec: json
```

### Step 3: Send an event through it

```sh
echo '{"remote_addr":"37.120.202.92","path":"/login"}' | docker run --rm -i \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/vector.yaml":/etc/vector/vector.yaml:ro \
  timberio/vector:0.58.0-alpine --config /etc/vector/vector.yaml
```

### Step 4: Read the result

Vector prints the event as one JSON line. Formatted, and without the `host`, `source_type` and `timestamp` fields that Vector adds, it looks like this:

```json
{
  "geo": {
    "city": "Secaucus",
    "country_code": "US",
    "country_name": "United States",
    "location": {
      "lat": 40.78834,
      "lon": -74.05502
    },
    "region": "New Jersey",
    "time_zone": "America/New_York"
  },
  "network": {
    "asn": 9009,
    "organization": "M247 Europe SRL"
  },
  "path": "/login",
  "remote_addr": "37.120.202.92",
  "security": {
    "is_anonymous": true,
    "is_proxy": true,
    "is_tor": false,
    "is_vpn": true,
    "threat_score": 50,
    "vpn_providers": [
      "Private Internet Access VPN"
    ]
  }
}
```

Compare it with the raw record above: the flags are now booleans, the coordinates are numbers in a `geo_point`-style object, and the AS number is an integer. Values depend on your database release. The address belongs to AS9009; its routes, peers and WHOIS data are in the [IPGeolocation.io ASN browser entry for AS9009](https://ipgeolocation.io/browse/asn/AS9009).

To go to production, replace the `stdin` source with your real source, such as `file`, `kubernetes_logs` or `kafka`, and the `console` sink with your real sink.

---

## Configuration reference

### Enrichment table settings

| Setting | Value |
| --- | --- |
| `type` | `mmdb`. The `geoip` type does not accept IPGeolocation.io files. |
| `path` | Path to the MMDB file. In Docker, the path inside the container. |

The table name, such as `ipgeo_security`, is the name you pass to `get_enrichment_table_record`.

### The lookup call

```text
record, err = get_enrichment_table_record("<table name>", {"ip": <field with the IP>})
```

| Result | When |
| --- | --- |
| `record` is the database record, `err` is `null` | The address is in the file. |
| `err` contains `No rows found` | The address is not in the file, for example a private address. |
| `err` contains `Invalid address: invalid IP address syntax` | The field is missing, empty, or not a single IP address. |

Always use the two-value form and check `err`, as in the quick start. Vector rejects a configuration that ignores the error.

### Field paths in each database

Every IPGeolocation.io IP database can be an enrichment table. With the record stored in `rec`, these are typical paths:

| Database | Example VRL path |
| --- | --- |
| IP Geolocation Database and IP to City Database | `rec.location.country.code2`, `rec.location.city.name.en`, `rec.time_zone` |
| IP to Country Database | `rec.location.country.code2` |
| IP Security Database | `rec.is_vpn`, `rec.threat_score`, `rec.vpn_provider_names` |
| IP to ASN Database | `rec.asn.as_number`, `rec.asn.organization` |
| IP to Company Database | `rec.company.name.en`, `rec.company.domain` |
| IP to ISP Database | `rec.isp`, `rec.asn`, `rec.as_organization`, `rec.country.code2` |
| IP Abuse Contact Database | `rec.abuse.emails`, `rec.abuse.name.en` |
| IP WHOIS Database | `rec.whois.organization.name`, `rec.whois.rir` |
| IP to Hosting Database | `rec.hosting_provider` |
| Residential Proxy Database | `rec.proxy_provider`, `rec.last_seen` |

In the IP to ISP Database, `rec.asn` is the AS number itself rather than a group, and the country sits at the top level instead of under `location`. The two datasets answer different questions; see [how an ISP differs from an ASN](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn).

Names come in several languages. Replace `en` with `cs`, `de`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` or `zh`. A name with no translation is an empty string.

For the meaning of each security field, see the [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html).

### Keeping the whole record

To keep every field instead of picking some, assign the record. This copies the full security record and turns every `"true"` and `"false"` into a boolean in one step, while scores, dates and lists stay as they are:

```vrl
sec, err = get_enrichment_table_record("ipgeo_security", {"ip": .remote_addr})
if err == null {
  .security = map_values(sec) -> |value| {
    if value == "true" { true } else if value == "false" { false } else { value }
  }
}
```

### Reading a combined database

A combined database holds the data of several databases, such as location, company and ASN, in a single file. Declare it once, and one lookup returns all of it:

```yaml
enrichment_tables:
  ipgeo_city_company_asn:
    type: mmdb
    path: /usr/local/share/ipgeolocation/db-ip-city-company-asn.mmdb
```

```vrl
rec, err = get_enrichment_table_record("ipgeo_city_company_asn", {"ip": .remote_addr})
if err == null {
  .geo.country_code = rec.location.country.code2
  .geo.city = rec.location.city.name.en
  .network.asn = to_int(rec.asn.as_number) ?? null
  .network.company = rec.company.name.en
}
```

---

## Recipes

### Send anonymous traffic to its own sink

Add a `route` transform after the enrichment. Events from VPNs, proxies and Tor go to one sink, and everything else to another:

```yaml
transforms:
  # ... the ipgeo transform from the quick start ...

  split:
    type: route
    inputs: [ipgeo]
    route:
      anonymous: .security.is_anonymous == true

sinks:
  siem:
    type: console            # replace with your SIEM sink
    inputs: [split.anonymous]
    encoding:
      codec: json
  everything_else:
    type: console            # replace with your normal sink
    inputs: [split._unmatched]
    encoding:
      codec: json
```

Because the quick start already turned the flags into booleans, the condition compares with `true`, not `"true"`. To route on a single signal, use `.security.is_tor == true` or `.security.is_vpn == true`.

### Enrich an Nginx access log

VRL parses the standard Nginx log format itself, so no regular expression is needed. `parse_nginx_log` puts the client address in `.client`. Keep the `enrichment_tables` section from the quick start, and change the source and transform:

```yaml
sources:
  nginx:
    type: file
    include: [/var/log/nginx/access.log]

transforms:
  ipgeo:
    type: remap
    inputs: [nginx]
    source: |
      . = parse_nginx_log!(.message, "combined")
      loc, err = get_enrichment_table_record("ipgeo_location", {"ip": .client})
      if err == null {
        .geo = {
          "country_code": loc.location.country.code2,
          "city": loc.location.city.name.en
        }
      }
```

After `parse_nginx_log`, VRL knows the exact shape of the event, so assign `.geo` as a whole object rather than one field at a time.

For the line `37.120.202.92 - - [06/Oct/2026:10:15:32 +0000] "POST /login HTTP/1.1" 401 512 "-" "Mozilla/5.0"`, the event gets `client`, `request`, `status` (`401`) and `size` (`512`) from the parser, plus `geo.country_code` (`"US"`) and `geo.city` (`"Secaucus"`).

### Use the visitor's address from `X-Forwarded-For`

Behind a load balancer or CDN, the connecting address belongs to the proxy. If your events carry an `X-Forwarded-For` value, take its first entry before the lookup:

```vrl
if exists(.x_forwarded_for) {
  first = split(string(.x_forwarded_for) ?? "", ",")[0]
  .remote_addr = strip_whitespace(string(first) ?? "")
}
```

For `37.120.202.92, 10.0.0.1`, this sets `.remote_addr` to `37.120.202.92`. Only trust this header when your own proxy sets it; see [how to get the real client IP address behind a proxy](https://ipgeolocation.io/guides/get-real-client-ip-address).

---

## Testing the enrichment with `vector test`

Vector can unit-test a transform against the real database files. Add a `tests` section to the configuration from the route recipe:

```yaml
tests:
  - name: VPN exit is flagged and routed
    inputs:
      - insert_at: ipgeo
        type: log
        log_fields:
          remote_addr: 37.120.202.92
    outputs:
      - extract_from: split.anonymous
        conditions:
          - type: vrl
            source: |
              assert_eq!(.security.is_vpn, true)
              assert!(to_int!(.security.threat_score) >= 50)
```

Then run:

```sh
vector test /etc/vector/vector.yaml
```

```text
Running tests
test VPN exit is flagged and routed ... passed
```

A failed assertion prints the expected and actual values and exits with a non-zero code, so the test can gate a deployment in CI. Test conditions do not know field types, so convert numbers with `to_int!` before comparing them.

---

## Keeping the databases up to date

IPGeolocation.io publishes new databases daily or weekly, depending on your plan. Vector keeps each table in memory, so a new file takes effect when Vector reloads:

1. Download the new file to a temporary name in the same directory, check it, and rename it over the old one. The rename is atomic.
2. Send Vector a `SIGHUP`: `kill -HUP <pid>`, or `docker kill --signal HUP <container>` in Docker. Vector reloads the configuration and the database files without stopping.

This script does both. It checks the download with the [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio), so a damaged file never replaces a good one:

```sh
#!/bin/sh
# Replace one IPGeolocation.io database and reload Vector.
set -eu

DB_DIR=/usr/local/share/ipgeolocation
DB_NAME=db-ip-security.mmdb
TMP="$DB_DIR/.$DB_NAME.new"

# 1. Download the new file next to the old one.
#    Replace this line with the download step for your plan.
curl -fsSL -o "$TMP" "$DOWNLOAD_URL"

# 2. Stop here if the file is damaged.
mmdbio verify --db "$TMP" || { rm -f "$TMP"; exit 1; }

# 3. Swap it in with an atomic rename.
mv "$TMP" "$DB_DIR/$DB_NAME"

# 4. Reload Vector.
kill -HUP "$(pidof vector)"
```

Schedule it with cron after each release. In Docker, mount the directory that holds the databases, as in the quick start, so the container sees the renamed file.

---

## Production notes

- **Memory.** Vector keeps each database file in memory. Budget RAM for at least the combined size of the files you load. In testing, three files totaling 245 MB added about 246 MB to Vector's memory use.
- **Only what you need.** Load only the databases your pipeline reads, and copy only the fields your dashboards and alerts use.
- **The right address.** Look up the visitor's address, not your proxy's. See the `X-Forwarded-For` recipe above.
- **Kubernetes.** Mount the databases from a volume at the same path in every Vector pod. After an update, send `SIGHUP` to each pod or restart the workload.
- **Tests in CI.** Run `vector test` on every configuration change.

---

## Troubleshooting

**Events have no `geo`, `security` or `network` fields.** The lookup returned an error and the `if err == null` block was skipped. Store the error on the event to see it, for example `.lookup_error = err` in an `else` branch:

- `No rows found`: the address is not in that database. Check it with mmdbio:

  ```sh
  mmdbio read --db /usr/local/share/ipgeolocation/db-ip-location.mmdb --ip 37.120.202.92
  ```

- `Invalid address: invalid IP address syntax`: the field is missing, empty, a list such as `203.0.113.7, 10.0.0.1`, or an address with a port.

**`Unsupported MMDB database type (ipgeolocation.io Database). Use mmdb enrichment table instead.`** The table is declared as `type: geoip`. Change it to `type: mmdb`.

**`error[E103]: unhandled fallible assignment`.** The lookup uses the one-value form, `rec = get_enrichment_table_record(...)`. Use `rec, err = ...` and check `err`.

**`i/o error: No such file or directory`.** The `path` of an enrichment table does not exist. In Docker, use the path inside the container.

**`error[E630]: fallible argument` in a test.** A test condition compares a value whose type is unknown. Convert it first, as in `to_int!(.security.threat_score) >= 50`.

**New data does not appear after an update.** Vector still holds the old file in memory. Send it a `SIGHUP`.

---

## FAQ

<details>
<summary><strong>Which enrichment table type should I use?</strong></summary>
<code>mmdb</code>. It returns the full record and works with every IPGeolocation.io IP database. The <code>geoip</code> type rejects these files.
</details>

<details>
<summary><strong>Can Vector use more than one IPGeolocation.io database at once?</strong></summary>
Yes. Declare one enrichment table per file and call each one from the same <code>remap</code> transform, as the quick start does with three databases.
</details>

<details>
<summary><strong>Are IP addresses sent to IPGeolocation.io?</strong></summary>
No. Vector reads the files on its own host. No API key or network access is needed at runtime.
</details>

<details>
<summary><strong>Does it handle IPv6?</strong></summary>
Yes. Each database covers both IPv4 and IPv6 addresses, and the lookup accepts either.
</details>

<details>
<summary><strong>Can I store the security flags as booleans?</strong></summary>
Yes. Compare each flag with <code>"true"</code>, as in the quick start, or convert all of them at once with <code>map_values</code>, as shown under "Keeping the whole record".
</details>

<details>
<summary><strong>How do I load a new database without downtime?</strong></summary>
Rename the new file over the old one and send Vector a <code>SIGHUP</code>. Vector reloads the tables without stopping.
</details>

---

## Related

- [IPGeolocation.io MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md)
- [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html)
- [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html)
- [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html)
- [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html)
- [Enrich logs with IPGeolocation.io in Fluent Bit](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/fluent-bit/README.md)
- [Enrich logs for Loki with IPGeolocation.io in Grafana Alloy](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/grafana-alloy/README.md)
- [IPGeolocation.io Nginx module for MMDB databases](https://ipgeolocation.io/documentation/nginx-integration)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
- [VRL function reference](https://vector.dev/docs/reference/vrl/functions/)
