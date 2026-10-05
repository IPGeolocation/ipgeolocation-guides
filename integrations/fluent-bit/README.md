# IPGeolocation.io with Fluent Bit

Fluent Bit's built-in [`geoip2` filter](https://docs.fluentbit.io/manual/data-pipeline/filters/geoip2-filter) can read IPGeolocation.io MMDB databases directly. It adds country, city, time zone, ASN and security data (VPN, proxy, Tor, threat score) to each log record, from local files, with no API calls and no plugin to install.

The filter takes a path into the record for each field, so it works with IPGeolocation.io's schema as it is. This guide shows a working configuration and the paths to use.

> [!TIP]
> Fluent Bit's documentation shows MaxMind paths such as `%{country.iso_code}`. IPGeolocation.io databases use their own paths, such as `%{location.country.code2}`. See the [field reference](../mmdb-field-reference/README.md) for the full list.

---

## Tested with

| Component | Version |
| --- | --- |
| Fluent Bit | 5.1.3 (official `fluent/fluent-bit` image) |
| Databases | IPGeolocation.io Location, Security and ASN MMDB files |

## Prerequisites

- Fluent Bit with the `geoip2` filter. The official images and packages include it.
- IPGeolocation.io MMDB files, for example `db-ip-location.mmdb`, `db-ip-security.mmdb` and `db-ip-asn.mmdb`. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or start with the free sample databases.
- Log records that carry the client IP address in a field. The examples use `remote_addr`.

## Configuration

Add one `geoip2` filter per database file. Each `record` line is `<new key> <field with the IP> %{<path in the database>}`.

```yaml
service:
  flush: 1
  log_level: warn

pipeline:
  inputs:
    - name: dummy
      tag: web
      dummy: '{"remote_addr": "203.0.113.10", "path": "/login"}'
      samples: 1

  filters:
    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-location.mmdb
      lookup_key: remote_addr
      record:
        - country_code remote_addr %{location.country.code2}
        - country_name remote_addr %{location.country.name.en}
        - city remote_addr %{location.city.name.en}
        - latitude remote_addr %{location.coordinates.latitude}
        - longitude remote_addr %{location.coordinates.longitude}
        - time_zone remote_addr %{time_zone}

    - name: geoip2
      match: web
      database: /usr/local/share/ipgeolocation/db-ip-security.mmdb
      lookup_key: remote_addr
      record:
        - is_tor remote_addr %{is_tor}
        - is_vpn remote_addr %{is_vpn}
        - is_proxy remote_addr %{is_proxy}
        - threat_score remote_addr %{threat_score}

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

The `dummy` input and `stdout` output are for testing. Replace them with your real input (for example `tail` on your web server's access log) and output.

If you use the classic configuration format, the same filter looks like this:

```ini
[FILTER]
    Name       geoip2
    Match      web
    Database   /usr/local/share/ipgeolocation/db-ip-location.mmdb
    Lookup_key remote_addr
    Record     country_code remote_addr %{location.country.code2}
    Record     city         remote_addr %{location.city.name.en}
```

## Try it with Docker

Put the configuration in `fluent-bit.yaml` and the database files in `./databases`, then run:

```sh
docker run --rm \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/fluent-bit.yaml":/fluent-bit/etc/fluent-bit.yaml:ro \
  fluent/fluent-bit:5.1.3 -c /fluent-bit/etc/fluent-bit.yaml
```

Change `203.0.113.10` in the `dummy` input to an address that is in your database. The output has the new keys:

```json
{"remote_addr":"203.0.113.10","path":"/login","country_code":"PK","country_name":"Pakistan","city":"Lahore","latitude":"31.54972","longitude":"74.34361","time_zone":"Asia/Karachi","is_tor":"true","is_vpn":"false","is_proxy":"false","threat_score":90,"asn":"64500","as_organization":"Example Telecom PK"}
```

The example values come from a test database. An address with no record in a file gets `null` for that file's keys.

## Notes

- **Booleans are strings.** Security flags arrive as `"true"` or `"false"`. Compare them as strings in later filters and in your backend's queries.
- **Coordinates are strings.** Convert `latitude` and `longitude` to numbers if your backend needs a geo point.
- **Other databases.** The ISP database is flat: use `%{country.code2}`, `%{isp}` and `%{asn}`. See the [field reference](../mmdb-field-reference/README.md).
- **Languages.** Replace `.en` in a name path with `de`, `fr`, `ja` or another supported language code.
- **Updates.** Fluent Bit opens the database when it starts. Replace the file atomically (download to a temporary file in the same directory, then rename it), then restart Fluent Bit.

## Troubleshooting

**Every key is `null`.** The address is not in that database, the `lookup_key` field is missing from the record, or the path is wrong. Check the record with `mmdbio read --db db-ip-location.mmdb --ip <address>`.

**Fluent Bit fails to start with a database error.** Check that the path in `database` exists inside the container and is readable.

## Related

- [Field reference](../mmdb-field-reference/README.md)
- [Vector](../vector/README.md) and [Grafana Alloy](../grafana-alloy/README.md) guides
- [IPGeolocation.io database documentation](https://ipgeolocation.io/documentation/databases.html)
