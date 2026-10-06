# DuckDB IP Geolocation in SQL with IPGeolocation.io Databases

## Overview

DuckDB can join IP intelligence onto your data with nothing but SQL. A community extension adds two functions that read IPGeolocation.io MMDB files: one looks up a single address per row, the other turns a whole database into a table. Point them at a CSV export, a folder of Parquet logs or a table you already have, and you get country, city, VPN and Tor flags, threat scores and network owners alongside your own columns.

The examples combine three IPGeolocation.io databases. The [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html) places each address in a country, region and city, with coordinates and a time zone. The [IP Security Database](https://ipgeolocation.io/ip-security-database.html) flags VPNs, proxies and Tor and rates the risk of each address. The [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html) identifies the network, by its [Autonomous System Number](https://ipgeolocation.io/guides/what-is-an-asn) and owner.

Nothing in the extension is specific to those three. You can query the IP to Country, IP to City and IP to ASN databases, the [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP WHOIS Database](https://ipgeolocation.io/ip-whois-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html) in exactly the same way, along with combined databases.

DuckDB runs inside your process, and the databases are local files, so analysis needs no API key, no rate limit and no network access once the extension is installed. For when a hosted API is the better fit, see [choosing between an IP geolocation API and a database](https://ipgeolocation.io/guides/ip-geolocation-api-vs-database-guide).

> [!TIP]
> `mmdb_record` takes a top-level key and returns that part of the record as JSON. Pull nested values out with JSONPath, as in `->> '$.location.country.code2'`. The [MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) lists every key.

---

## Two ways to query a database

| Function | Kind | Use it to |
| --- | --- | --- |
| `mmdb_record(file, ip, key)` | Scalar | Add data for the address in each row of your own table or file. |
| `read_mmdb(file)` | Table | Read the database itself, one row per network, to filter, count or export it. |

## Questions you can answer

- **Where do our sign-in attempts come from?** Group a login log by country and network owner.
- **How much of our traffic is anonymized?** Count requests from VPN, proxy and Tor addresses, per day or per endpoint.
- **Which hosting networks hit our API?** Join firewall or API logs with network owners to find automated sources.
- **Which Tor exit networks fall inside a given range?** Scan the IP Security Database for one subnet or for all of them.
- **Can this dataset be shared enriched?** Write the enriched result to Parquet for a data warehouse or a notebook.

---

## About DuckDB and the extension

[DuckDB](https://duckdb.org/) is an in-process analytical database. It reads CSV, Parquet and JSON files directly, runs in a command-line shell or inside Python, R, Java and other languages, and needs no server.

The IPGeolocation.io lookups come from the open source community extension named `maxmind`. Of its functions, `mmdb_record` and `read_mmdb` are generic MMDB readers, and they are the ones to use with IPGeolocation.io files. DuckDB downloads the extension from its community repository the first time you install it, and every session loads it with `LOAD`.

---

## How a lookup comes back

`mmdb_record` returns one top-level key of the record, wrapped in a JSON object. These are real results for `37.120.202.92`:

| Call | Result |
| --- | --- |
| `mmdb_record('databases/db-ip-security.mmdb', '37.120.202.92', 'is_vpn')` | `{"is_vpn":"true"}` |
| `mmdb_record('databases/db-ip-security.mmdb', '37.120.202.92', 'threat_score')` | `{"threat_score":50}` |
| `mmdb_record('databases/db-ip-asn.mmdb', '37.120.202.92', 'asn')` | `{"asn":{"as_number":"9009","country_code":"RO","domain":"m247.com","organization":"M247 Europe SRL","type":"HOSTING"}}` |

DuckDB's `->>` operator reads a value out of that JSON as text. `mmdb_record(..., 'asn') ->> '$.asn.organization'` returns `M247 Europe SRL`. Security flags arrive as the text `true` or `false`, scores as JSON numbers, and provider names as JSON arrays.

---

## Requirements

| Component | Details |
| --- | --- |
| DuckDB | 1.5.6 (tested), as the CLI or any client that can install community extensions. Python was tested too. |
| `maxmind` extension | v0.10.0 (tested), installed with `INSTALL maxmind FROM community`. |
| IPGeolocation.io databases | The MMDB edition of each database you want to query, downloaded from your [IPGeolocation.io account](https://app.ipgeolocation.io). Plans are listed on the [IP database pricing page](https://ipgeolocation.io/db-pricing.html). |

---

## Quick start

Enrich a six-line access log in the DuckDB shell.

### Step 1: Lay out the files

Create a working folder with the three MMDB files in a `databases` subfolder:

```text
databases/
├── db-ip-asn.mmdb
├── db-ip-location.mmdb
└── db-ip-security.mmdb
access_log.csv
```

### Step 2: Create a sample log

Save this as `access_log.csv`:

```text
ts,client_ip,method,path,status
2026-10-06 10:15:32,37.120.202.92,POST,/login,401
2026-10-06 10:15:40,5.45.96.188,POST,/login,401
2026-10-06 10:16:02,131.229.141.0,GET,/,200
2026-10-06 10:16:09,5.9.12.205,GET,/pricing,200
2026-10-06 10:16:15,2001:1540::,GET,/,200
2026-10-06 10:16:21,203.0.113.10,GET,/,200
```

### Step 3: Run the query

Start `duckdb` in the working folder and run:

```sql
INSTALL maxmind FROM community;
LOAD maxmind;

SELECT
    client_ip,
    path,
    mmdb_record('databases/db-ip-location.mmdb', client_ip, 'location')
        ->> '$.location.country.code2'                                   AS country,
    mmdb_record('databases/db-ip-location.mmdb', client_ip, 'location')
        ->> '$.location.city.name.en'                                    AS city,
    (mmdb_record('databases/db-ip-security.mmdb', client_ip, 'is_anonymous')
        ->> '$.is_anonymous') = 'true'                                   AS anonymous,
    (mmdb_record('databases/db-ip-security.mmdb', client_ip, 'threat_score')
        ->> '$.threat_score')::INTEGER                                   AS threat_score,
    mmdb_record('databases/db-ip-asn.mmdb', client_ip, 'asn')
        ->> '$.asn.organization'                                         AS network
FROM read_csv('access_log.csv')
ORDER BY threat_score DESC NULLS LAST;
```

### Step 4: Read the result

```text
┌───────────────┬──────────┬─────────┬─────────────┬───────────┬──────────────┬─────────────────────┐
│   client_ip   │   path   │ country │    city     │ anonymous │ threat_score │       network       │
│    varchar    │ varchar  │ varchar │   varchar   │  boolean  │    int32     │       varchar       │
├───────────────┼──────────┼─────────┼─────────────┼───────────┼──────────────┼─────────────────────┤
│ 5.45.96.188   │ /login   │ DE      │ Nuremberg   │ true      │           80 │ netcup GmbH         │
│ 37.120.202.92 │ /login   │ US      │ Secaucus    │ true      │           50 │ M247 Europe SRL     │
│ 5.9.12.205    │ /pricing │ DE      │ Falkenstein │ false     │            5 │ Hetzner Online GmbH │
│ 2001:1540::   │ /        │ NL      │ Amsterdam   │ false     │            5 │ Equinix, Inc.       │
│ 131.229.141.0 │ /        │ US      │ Ashburn     │ false     │            0 │ Skyhigh Security    │
│ 203.0.113.10  │ /        │ NULL    │ NULL        │ NULL      │         NULL │ NULL                │
└───────────────┴──────────┴─────────┴─────────────┴───────────┴──────────────┴─────────────────────┘
```

Both failed sign-ins came from anonymizing networks: a Tor exit in a Nuremberg data center and a VPN exit hosted by M247. The IPv6 address works like any other, and `203.0.113.10`, a documentation address, has no record, so its columns are `NULL`. Values change as the databases are updated. The [ASN browser entry for AS9009](https://ipgeolocation.io/browse/asn/AS9009) shows more about the VPN's network.

This direct form is fine for small files. For large ones, use the pattern in "Enrich a large log file" below.

---

## Reference

### Functions

| Function | Arguments | Returns |
| --- | --- | --- |
| `mmdb_record` | Database file, IP address, top-level key | The key and its value as JSON text, or `NULL` |
| `read_mmdb` | Database file; optional `network := 'CIDR'` | A table with `network` and `record` (the full record as JSON text) |

### What `mmdb_record` returns

| Situation | Result |
| --- | --- |
| The address has a record and the key exists | JSON such as `{"is_vpn":"true"}` |
| The key does not exist at the top level, for example `'location.country.code2'` | `{}` |
| The address has no record | `NULL` |
| The IP argument is `NULL`, empty, a list such as `203.0.113.7, 10.0.0.1`, or has a port | `NULL` |
| The file cannot be found | `Invalid Input Error: FileNotFound` |

### Reading values out of the JSON

| Need | Expression |
| --- | --- |
| Text | `mmdb_record(f, ip, 'location') ->> '$.location.city.name.en'` |
| Boolean | `(mmdb_record(f, ip, 'is_vpn') ->> '$.is_vpn') = 'true'` |
| Integer | `(mmdb_record(f, ip, 'threat_score') ->> '$.threat_score')::INTEGER` |
| Coordinate | `(mmdb_record(f, ip, 'location') ->> '$.location.coordinates.latitude')::DOUBLE` |
| First list entry | `mmdb_record(f, ip, 'vpn_provider_names') ->> '$.vpn_provider_names[0]'` |
| Whole list as `VARCHAR[]` | `from_json(mmdb_record(f, ip, 'vpn_provider_names') -> '$.vpn_provider_names', '["VARCHAR"]')` |

Names are stored in several languages. Change `en` in a path to `cs`, `de`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` or `zh`; a name without a translation is an empty string.

### An expression for each database

Here `f` is the database file and `ip` the address column:

| Database | Sample expression |
| --- | --- |
| IP Geolocation Database or IP to City Database | `mmdb_record(f, ip, 'location') ->> '$.location.city.name.en'` |
| IP to Country Database | `mmdb_record(f, ip, 'location') ->> '$.location.country.code2'` |
| IP Security Database | `mmdb_record(f, ip, 'is_vpn') ->> '$.is_vpn'` |
| IP to ASN Database | `mmdb_record(f, ip, 'asn') ->> '$.asn.as_number'` |
| IP to Company Database | `mmdb_record(f, ip, 'company') ->> '$.company.name.en'` |
| IP to ISP Database | `mmdb_record(f, ip, 'isp') ->> '$.isp'` |
| IP Abuse Contact Database | `mmdb_record(f, ip, 'abuse') ->> '$.abuse.emails'` |
| IP WHOIS Database | `mmdb_record(f, ip, 'whois') ->> '$.whois.rir'` |
| IP to Hosting Database | `mmdb_record(f, ip, 'hosting_provider') ->> '$.hosting_provider'` |
| Residential Proxy Database | `mmdb_record(f, ip, 'proxy_provider') ->> '$.proxy_provider'` |

In the IP to ISP Database, the AS number is a top-level key of its own (`mmdb_record(f, ip, 'asn') ->> '$.asn'`), and so is the country (`'country'`). An ISP and the AS that routes its traffic are not always the same organization; [how an ISP differs from an ASN](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn) explains the difference.

The [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html) describes every security key, including the [residential proxy](https://ipgeolocation.io/residential-proxy-database.html) and relay flags and the confidence scores.

### Reusable macros

Macros keep long queries readable. Define them once per session, or put them in a file and load it with `.read`:

```sql
CREATE OR REPLACE MACRO ipgeo_location(ip) AS
    mmdb_record('databases/db-ip-location.mmdb', ip, 'location')::JSON -> 'location';

CREATE OR REPLACE MACRO ipgeo_security(ip, field) AS
    mmdb_record('databases/db-ip-security.mmdb', ip, field) ->> ('$.' || field);

CREATE OR REPLACE MACRO ipgeo_asn(ip) AS
    mmdb_record('databases/db-ip-asn.mmdb', ip, 'asn')::JSON -> 'asn';
```

With them, the quick start columns become one-liners:

```sql
SELECT
    client_ip,
    ipgeo_location(client_ip) ->> '$.country.code2'      AS country,
    ipgeo_security(client_ip, 'is_vpn') = 'true'         AS is_vpn,
    ipgeo_security(client_ip, 'threat_score')::INTEGER   AS threat_score,
    ipgeo_asn(client_ip) ->> '$.as_number'               AS asn
FROM read_csv('access_log.csv');
```

Relative file paths resolve from the folder DuckDB runs in. In scripts and scheduled jobs, use absolute paths.

---

## Recipes

### Enrich a large log file

Logs repeat the same addresses many times. Look each distinct address up once, store the results in a table, and join them back. This uses the macros above:

```sql
CREATE OR REPLACE TABLE ip_geo AS
SELECT
    client_ip,
    ipgeo_location(client_ip) ->> '$.country.code2'      AS country,
    ipgeo_location(client_ip) ->> '$.city.name.en'       AS city,
    ipgeo_security(client_ip, 'is_anonymous') = 'true'   AS anonymous,
    ipgeo_security(client_ip, 'threat_score')::INTEGER   AS threat_score
FROM (SELECT DISTINCT client_ip FROM read_parquet('logs/*.parquet'));

COPY (
    SELECT logs.*, g.country, g.city, g.anonymous, g.threat_score
    FROM read_parquet('logs/*.parquet') AS logs
    LEFT JOIN ip_geo AS g USING (client_ip)
) TO 'enriched.parquet' (FORMAT parquet);
```

The `LEFT JOIN` keeps rows whose address has no record. Swap `read_parquet` for `read_csv`, or for a table name, to match your source.

> [!IMPORTANT]
> In testing with extension v0.10.0, calling `mmdb_record` on every row of a large file read directly with `read_csv` or `read_parquet` sometimes ended the DuckDB process with a segmentation fault. The distinct-addresses pattern above never crashed in repeated runs, with up to 747,071 distinct addresses. Use it for anything bigger than a quick check.

### Summarize traffic by country

Once the data is enriched, ordinary SQL answers the questions:

```sql
SELECT
    country,
    count(*)                                                       AS requests,
    round(100.0 * count(*) FILTER (WHERE anonymous) / count(*), 1) AS anonymous_pct
FROM 'enriched.parquet'
GROUP BY country
ORDER BY requests DESC
LIMIT 10;
```

### Scan a whole database

`read_mmdb` returns one row per network, with the full record as JSON in the `record` column.

Every Tor exit network in the IP Security Database:

```sql
SELECT network
FROM read_mmdb('databases/db-ip-security.mmdb')
WHERE record ->> '$.is_tor' = 'true';
```

Networks per country in the IP Geolocation Database:

```sql
SELECT record ->> '$.location.country.code2' AS country, count(*) AS networks
FROM read_mmdb('databases/db-ip-location.mmdb')
GROUP BY country
ORDER BY networks DESC;
```

A full database holds millions of networks, so a complete scan takes a while. To read one range only, pass it as `network`:

```sql
SELECT network, record ->> '$.is_vpn' AS is_vpn
FROM read_mmdb('databases/db-ip-security.mmdb', network := '37.120.202.0/24');
```

For that range, the result starts with `37.120.202.0/30`, `37.120.202.7/32` and `37.120.202.8/29`, all with `is_vpn` set to `true`.

### From Python

The same SQL runs through DuckDB's Python package:

```python
import duckdb

con = duckdb.connect()
con.sql("INSTALL maxmind FROM community")
con.sql("LOAD maxmind")

rows = con.sql("""
    SELECT client_ip,
           mmdb_record('databases/db-ip-location.mmdb', client_ip, 'location')
               ->> '$.location.country.code2' AS country
    FROM read_csv('access_log.csv')
""").fetchall()

for client_ip, country in rows:
    print(client_ip, country)
```

With the quick start files, it prints `37.120.202.92 US`, `5.45.96.188 DE` and so on, and `203.0.113.10 None` for the address without a record.

---

## Keeping the databases up to date

Depending on your plan, IPGeolocation.io publishes new releases every day or every week. DuckDB needs no reload: in testing, the next query after a file was replaced already returned the new data, even in the same session.

Replace files with an atomic rename, so a query never reads a half-written file. This script also refuses a damaged download, using the [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio):

```sh
#!/bin/sh
# Install a new IPGeolocation.io database release for DuckDB queries.
set -eu

DB_DIR=/data/ipgeolocation
DB_NAME=db-ip-security.mmdb
TMP="$DB_DIR/.$DB_NAME.new"

# 1. Fetch the release next to the current file.
#    Replace this line with the download step for your plan.
curl -fsSL -o "$TMP" "$DOWNLOAD_URL"

# 2. Keep the old file if the new one is damaged.
mmdbio verify --db "$TMP" || { rm -f "$TMP"; exit 1; }

# 3. Swap it in. The next query reads the new release.
mv "$TMP" "$DB_DIR/$DB_NAME"
```

Enriched tables and Parquet files keep the values from the release they were built with. Rebuild them when you need current data.

---

## Troubleshooting

**`Catalog Error: Scalar Function with name mmdb_record does not exist!`** The extension is not loaded in this session. Run `LOAD maxmind;`, after `INSTALL maxmind FROM community;` if it was never installed.

**`Binder Error: No function matches the given name and argument types 'mmdb_record(STRING_LITERAL, STRING_LITERAL)'`.** `mmdb_record` needs three arguments: the file, the address and a top-level key.

**`mmdb_record` returns `{}`.** The key is not a top-level key. Pass the first part only, such as `'location'`, and read the rest with `->> '$.location.country.code2'`.

**`mmdb_record` returns `NULL`.** The address has no record, or the value is not a single valid IP address. Check an address with mmdbio:

```sh
mmdbio read --db databases/db-ip-location.mmdb --ip 37.120.202.92
```

**`Invalid Input Error: FileNotFound`.** The path to the database is wrong. Relative paths start from the folder DuckDB runs in.

**DuckDB exits with `Segmentation fault`.** This happened in testing when `mmdb_record` ran on every row of a large file read directly with `read_csv` or `read_parquet`. Look up distinct addresses into a table first, as in "Enrich a large log file".

**Enriched results show old values.** They were built from an earlier release. Rerun the enrichment after you install a new file.

---

## FAQ

<details>
<summary><strong>Does DuckDB need a server to do this?</strong></summary>
No. DuckDB runs inside the shell or your program, and the databases are ordinary files next to it.
</details>

<details>
<summary><strong>Do the addresses in my data leave my machine?</strong></summary>
No. Lookups read the local MMDB files. The only network access is the one-time extension download from DuckDB's community repository, which does not include your data.
</details>

<details>
<summary><strong>Which IPGeolocation.io databases can I query?</strong></summary>
All IP databases in their MMDB edition, and combined databases too. "An expression for each database" above has a starting point for each.
</details>

<details>
<summary><strong>Can my data contain IPv6 addresses?</strong></summary>
Yes. The quick start includes one, and the databases cover IPv6 alongside IPv4.
</details>

<details>
<summary><strong>Can I list every network with a given property?</strong></summary>
Yes. <code>read_mmdb</code> turns a database into a table, so a <code>WHERE</code> clause can find, for example, every Tor exit network.
</details>

<details>
<summary><strong>Do I need to restart anything after a database update?</strong></summary>
No. The next query reads the new file. Rebuild any tables or files you enriched earlier.
</details>

---

## Related

- [IPGeolocation.io MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md)
- [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html)
- [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html)
- [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html)
- [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html)
- [IPGeolocation.io Snowflake integration, for enrichment in a cloud data warehouse](https://ipgeolocation.io/integrations/snowflake)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
