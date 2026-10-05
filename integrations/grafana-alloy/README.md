# IPGeolocation.io with Grafana Alloy and Loki

Grafana Alloy's [`stage.geoip`](https://grafana.com/docs/alloy/latest/reference/components/loki/loki.process/) block in `loki.process` can read IPGeolocation.io MMDB databases. With `custom_lookups`, it adds country, city, time zone and security data (VPN, proxy, Tor, threat score) to log lines before they reach Loki, from local files, with no API calls and no plugin to install.

> [!TIP]
> The built-in lookups of `stage.geoip` (`db_type = "city"` or `"asn"`) expect MaxMind's GeoIP2 field names. With IPGeolocation.io databases, use `custom_lookups` and IPGeolocation.io's own paths, such as `location.country.code2`. See the [field reference](../mmdb-field-reference/README.md).

---

## Tested with

| Component | Version |
| --- | --- |
| Grafana Alloy | v1.20.1 (official `grafana/alloy` image) |
| Databases | IPGeolocation.io Location and Security MMDB files |

## Prerequisites

- Grafana Alloy, sending logs to Loki or another `loki.write` target.
- IPGeolocation.io MMDB files, for example `db-ip-location.mmdb` and `db-ip-security.mmdb`. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or start with the free sample databases.
- Log lines that carry the client IP address. The examples parse JSON lines with a `remote_addr` field.

## Configuration

Extract the IP address, then add one `stage.geoip` per database file. Each key in `custom_lookups` becomes a value in the extracted map; later stages turn it into a label or structured metadata.

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
      "city"         = "location.city.name.en",
      "time_zone"    = "time_zone",
    }
  }

  stage.geoip {
    source = "client_ip"
    db     = "/usr/local/share/ipgeolocation/db-ip-security.mmdb"
    custom_lookups = {
      "is_tor"       = "is_tor",
      "threat_score" = "threat_score",
    }
  }

  stage.labels {
    values = { country_code = "" }
  }

  stage.structured_metadata {
    values = { city = "", time_zone = "", is_tor = "", threat_score = "" }
  }
}

loki.echo "out" { }
```

`loki.echo` prints entries for testing. Replace it with your `loki.write` component.

> [!IMPORTANT]
> Keep labels low-cardinality. A country code makes a good label; city, threat score and similar values belong in structured metadata, as above, or Loki's index grows with every new value.

## Try it with Docker

Put the configuration in `config.alloy`, the database files in `./databases`, and a sample log in `./logs/app.log`:

```sh
printf '{"remote_addr":"203.0.113.10","msg":"login"}\n' > logs/app.log

docker run --rm \
  -v "$PWD/databases":/usr/local/share/ipgeolocation:ro \
  -v "$PWD/logs":/var/log/app:ro \
  -v "$PWD/config.alloy":/etc/alloy/config.alloy:ro \
  grafana/alloy:v1.20.1 run /etc/alloy/config.alloy --storage.path=/tmp/alloy
```

Use an address that is in your database. Alloy logs each entry it received:

```text
msg="received log entry" component_id=loki.echo.out labels="{country_code=\"PK\", filename=\"/var/log/app/app.log\", job=\"app\"}" structured_metadata="{\"city\":\"Lahore\",\"is_tor\":\"true\",\"threat_score\":\"90\",\"time_zone\":\"Asia/Karachi\"}"
```

The example values come from a test database. An address with no record in a file gets no values from that file.

## Querying in Loki

With the configuration above you can filter by label and by structured metadata, for example in LogQL:

```logql
{job="app", country_code="PK"} | is_tor="true"
```

## Notes

- **Booleans are strings.** Security flags arrive as `"true"` or `"false"`.
- **Other databases.** The ISP database is flat (`country.code2`, `isp`, `asn`); the ASN database uses `asn.as_number` and `asn.organization`. See the [field reference](../mmdb-field-reference/README.md).
- **Languages.** Replace `.en` in a name path with `de`, `fr`, `ja` or another supported language code.
- **Updates.** Replace each file atomically (download to a temporary file in the same directory, then rename it), then reload or restart Alloy so it reopens the databases.

## Troubleshooting

**`unable to get City record for the ip ... cannot unmarshal`.** The stage uses `db_type = "city"` (or `"asn"`), which decodes MaxMind's GeoIP2 layout. Remove `db_type` and use `custom_lookups` as shown above.

**No new labels or metadata.** The `client_ip` value is empty (check the `stage.json` expression), or the addresses are not in the database. Check one with `mmdbio read --db db-ip-location.mmdb --ip <address>`.

## Related

- [Field reference](../mmdb-field-reference/README.md)
- [Fluent Bit](../fluent-bit/README.md) and [Vector](../vector/README.md) guides
- [IPGeolocation.io database documentation](https://ipgeolocation.io/documentation/databases.html)
