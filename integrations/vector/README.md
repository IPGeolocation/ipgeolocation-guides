# IPGeolocation.io with Vector

Vector's [`mmdb` enrichment table](https://vector.dev/docs/reference/configuration/enrichment_tables/) reads any MMDB file, including IPGeolocation.io databases. A `remap` transform looks up each event's IP address and adds country, city, time zone, ASN and security data (VPN, proxy, Tor, threat score), from local files, with no API calls and no plugin to install.

> [!TIP]
> Use the `mmdb` table type, not `geoip`. The `geoip` type only accepts MaxMind's GeoIP2 and GeoLite2 databases. The `mmdb` type returns the whole record, so you use IPGeolocation.io's own paths, such as `location.country.code2`. See the [field reference](../mmdb-field-reference/README.md).

---

## Tested with

| Component | Version |
| --- | --- |
| Vector | 0.58.0 (official `timberio/vector` image) |
| Databases | IPGeolocation.io Location, Security and ASN MMDB files |

## Prerequisites

- A Vector release with the `mmdb` enrichment table, which was added in March 2024 ([#20054](https://github.com/vectordotdev/vector/pull/20054)). This guide was tested on 0.58.0.
- IPGeolocation.io MMDB files, for example `db-ip-location.mmdb`, `db-ip-security.mmdb` and `db-ip-asn.mmdb`. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or start with the free sample databases.
- Events that carry the client IP address in a field. The examples use `remote_addr`.

## Configuration

Declare one enrichment table per database file, then look up each table in a `remap` transform.

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
  app_logs:
    type: stdin
    decoding:
      codec: json

transforms:
  ipgeo:
    type: remap
    inputs: [app_logs]
    source: |
      loc, err = get_enrichment_table_record("ipgeo_location", {"ip": .remote_addr})
      if err == null {
        .geo.country_code = loc.location.country.code2
        .geo.country_name = loc.location.country.name.en
        .geo.city = loc.location.city.name.en
        .geo.latitude = to_float(loc.location.coordinates.latitude) ?? null
        .geo.longitude = to_float(loc.location.coordinates.longitude) ?? null
        .geo.time_zone = loc.time_zone
      }
      sec, err = get_enrichment_table_record("ipgeo_security", {"ip": .remote_addr})
      if err == null {
        .security.is_tor = sec.is_tor == "true"
        .security.is_vpn = sec.is_vpn == "true"
        .security.is_proxy = sec.is_proxy == "true"
        .security.threat_score = sec.threat_score
      }
      asn, err = get_enrichment_table_record("ipgeo_asn", {"ip": .remote_addr})
      if err == null {
        .network.asn = asn.asn.as_number
        .network.organization = asn.asn.organization
      }

sinks:
  out:
    type: console
    inputs: [ipgeo]
    encoding:
      codec: json
```

What the transform does:

- `get_enrichment_table_record` returns an error when the address has no record, for example a private address. The `if err == null` blocks skip those events instead of failing.
- Security flags are stored as the strings `"true"` and `"false"`. Comparing with `== "true"` turns them into real booleans.
- Coordinates are stored as strings. `to_float` turns them into numbers for geo point fields.

The `stdin` source and `console` sink are for testing. Replace them with your real source (for example `file` or `kafka`) and sink.

## Try it with Docker

Put the configuration in `vector.yaml` and the database files in `./databases`, then run:

```sh
echo '{"remote_addr":"203.0.113.10","path":"/login"}' | docker run --rm -i \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/vector.yaml":/etc/vector/vector.yaml:ro \
  timberio/vector:0.58.0-alpine --config /etc/vector/vector.yaml
```

Use an address that is in your database. The event comes out enriched:

```json
{"geo":{"city":"Lahore","country_code":"PK","country_name":"Pakistan","latitude":31.54972,"longitude":74.34361,"time_zone":"Asia/Karachi"},"network":{"asn":"64500","organization":"Example Telecom PK"},"path":"/login","remote_addr":"203.0.113.10","security":{"is_proxy":false,"is_tor":true,"is_vpn":false,"threat_score":90}}
```

The example values come from a test database (Vector also adds `host`, `source_type` and `timestamp`).

## Notes

- **Other databases.** The ISP database is flat: use `rec.country.code2`, `rec.isp` and `rec.asn`. The Company database uses `rec.company.name.en`. See the [field reference](../mmdb-field-reference/README.md).
- **Languages.** Replace `.en` in a name path with `de`, `fr`, `ja` or another supported language code.
- **Updates.** Replace each file atomically (download to a temporary file in the same directory, then rename it), then reload or restart Vector so it reopens the tables.

## Troubleshooting

**No `geo` fields on any event.** The field you look up (`.remote_addr`) is missing, or the addresses are not in the database. Check one with `mmdbio read --db db-ip-location.mmdb --ip <address>`.

**`Unsupported MMDB database type (ipgeolocation.io Database)`.** The table is declared as `type: geoip`. Change it to `type: mmdb`.

**Vector refuses the configuration.** VRL requires errors to be handled. Keep the `value, err = ...` form shown above.

## Related

- [Field reference](../mmdb-field-reference/README.md)
- [Fluent Bit](../fluent-bit/README.md) and [Grafana Alloy](../grafana-alloy/README.md) guides
- [IPGeolocation.io database documentation](https://ipgeolocation.io/documentation/databases.html)
