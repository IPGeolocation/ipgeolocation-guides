# IPGeolocation.io with DuckDB

The [`maxmind` community extension](https://duckdb.org/community_extensions/extensions/maxmind) for DuckDB reads any MMDB file, including IPGeolocation.io databases. Use it to enrich IP addresses in a table, a CSV or a Parquet file with country, city, ASN and security data (VPN, proxy, Tor, threat score) in plain SQL, or to scan a whole database, for example to list every Tor exit network. Everything runs locally, with no API calls.

> [!TIP]
> The extension's `geolite_*` and `geoip_*` functions expect MaxMind's databases. With IPGeolocation.io databases, use the generic `mmdb_record` and `read_mmdb` functions and IPGeolocation.io's own paths. See the [field reference](../mmdb-field-reference/README.md).

---

## Tested with

| Component | Version |
| --- | --- |
| DuckDB | 1.5.6 |
| `maxmind` extension | v0.10.0 |
| Databases | IPGeolocation.io Location and Security MMDB files |

## Prerequisites

- DuckDB (CLI, Python, or any client that can install community extensions).
- IPGeolocation.io MMDB files, for example `db-ip-location.mmdb` and `db-ip-security.mmdb`. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or start with the free sample databases.

## Install the extension

```sql
INSTALL maxmind FROM community;
LOAD maxmind;
```

## Look up addresses

`mmdb_record(file, ip, key)` returns the record's top-level `key` as JSON. Extract the value you need with DuckDB's JSON operators:

```sql
SELECT
    client_ip,
    mmdb_record('/usr/local/share/ipgeolocation/db-ip-location.mmdb', client_ip, 'location')::json
        -> 'location' -> 'country' ->> 'code2'                                       AS country_code,
    mmdb_record('/usr/local/share/ipgeolocation/db-ip-security.mmdb', client_ip, 'threat_score')::json
        ->> 'threat_score'                                                           AS threat_score,
    mmdb_record('/usr/local/share/ipgeolocation/db-ip-security.mmdb', client_ip, 'is_vpn')::json
        ->> 'is_vpn' = 'true'                                                        AS is_vpn
FROM access_log
ORDER BY client_ip;
```

On a test database this returned:

| client_ip | country_code | threat_score | is_vpn |
| --- | --- | --- | --- |
| 198.51.100.7 | DE | NULL | NULL |
| 203.0.113.10 | PK | 90 | false |
| 203.0.113.11 | PK | 60 | true |

`NULL` means the address has no record in that file (here, no security record).

The third argument is a top-level key. For nested fields, select the top-level key (`location`) and walk down with `->` and `->>`. Other examples:

```sql
-- City name
mmdb_record('db-ip-location.mmdb', ip, 'location')::json -> 'location' -> 'city' -> 'name' ->> 'en'

-- Time zone (top level)
mmdb_record('db-ip-location.mmdb', ip, 'time_zone')::json ->> 'time_zone'

-- AS number and organization
mmdb_record('db-ip-asn.mmdb', ip, 'asn')::json -> 'asn' ->> 'as_number'
mmdb_record('db-ip-asn.mmdb', ip, 'asn')::json -> 'asn' ->> 'organization'
```

## Scan a whole database

`read_mmdb(file)` returns one row per network, with the record as JSON in a `record` column. This answers questions that per-address lookups cannot.

Every Tor exit network:

```sql
SELECT network
FROM read_mmdb('/usr/local/share/ipgeolocation/db-ip-security.mmdb')
WHERE record::json ->> 'is_tor' = 'true';
```

Networks per country:

```sql
SELECT record::json -> 'location' -> 'country' ->> 'code2' AS country_code, count(*) AS networks
FROM read_mmdb('/usr/local/share/ipgeolocation/db-ip-location.mmdb')
GROUP BY country_code
ORDER BY networks DESC;
```

A full scan of a large database takes time. To scan only part of it, pass a subnet:

```sql
SELECT network, record
FROM read_mmdb('/usr/local/share/ipgeolocation/db-ip-security.mmdb', network := '203.0.113.0/24');
```

## Notes

- **Booleans are strings.** Security flags are `"true"` or `"false"`; compare with `= 'true'` to get a boolean.
- **Coordinates are strings.** Cast them when you need numbers, with parentheses around the extraction: `(mmdb_record(file, ip, 'location')::json -> 'location' -> 'coordinates' ->> 'latitude')::DOUBLE`.
- **Other databases.** The ISP database is flat (`country`, `isp`, `asn` at the top level); the Company database uses `company`. See the [field reference](../mmdb-field-reference/README.md).
- **Updates.** Replace each file atomically (download to a temporary file in the same directory, then rename it). Later queries read the new file.

## Troubleshooting

**`No function matches the given name and argument types 'mmdb_record(...)'`.** `mmdb_record` needs three arguments: the file, the IP address and a top-level key.

**`Referenced column "is_tor" not found`.** For IPGeolocation.io databases, `read_mmdb` returns only `network` and `record`. Read fields from `record::json`.

## Related

- [Field reference](../mmdb-field-reference/README.md)
- [IPGeolocation.io Snowflake integration](https://ipgeolocation.io/integrations.html), for warehouse-scale enrichment
- [IPGeolocation.io database documentation](https://ipgeolocation.io/documentation/databases.html)
