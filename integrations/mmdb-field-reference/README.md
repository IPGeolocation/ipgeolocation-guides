# IPGeolocation.io MMDB field paths

Tools that read MMDB files with a generic reader, such as Fluent Bit, Vector, Grafana Alloy, Apache `mod_maxminddb` and DuckDB, need the path of each value inside a record. IPGeolocation.io databases use their own schema, so the paths differ from MaxMind's GeoIP2 paths (`location.country.code2`, not `country.iso_code`).

This page lists the paths for the most used fields. The guides for each tool link here:

- [Fluent Bit](../fluent-bit/README.md)
- [Vector](../vector/README.md)
- [Grafana Alloy and Loki](../grafana-alloy/README.md)
- [Apache HTTP Server](../apache/README.md)
- [DuckDB](../duckdb/README.md)

> [!TIP]
> To see the full record for an address in your own file, use [mmdbio](https://github.com/IPGeolocation/mmdbio): `mmdbio read --db db-ip-location.mmdb --ip 8.8.8.8`. Records differ between products and releases, so check a few addresses before you rely on a path.

---

## Location and City databases

Files: `db-ip-location.mmdb`, `db-ip-city.mmdb`, and the City bundles (`db-ip-city-isp.mmdb`, `db-ip-city-company-asn.mmdb`, ...).

| Field | Path | Example |
| --- | --- | --- |
| Country code (ISO 3166-1 alpha-2) | `location.country.code2` | `US` |
| Country code (alpha-3) | `location.country.code3` | `USA` |
| Country name | `location.country.name.en` | `United States` |
| Continent code | `location.country.continent.code` | `NA` |
| State or region code | `location.state.code` | `US-NV` |
| State or region name | `location.state.name.en` | `Nevada` |
| District | `location.district.name.en` | `Clark County` |
| City | `location.city.name.en` | `Las Vegas` |
| Postal code | `location.zipcode` | `89101` |
| Latitude | `location.coordinates.latitude` | `36.17157` |
| Longitude | `location.coordinates.longitude` | `-115.13912` |
| GeoNames ID | `location.geoname_id` | `5514560` |
| Time zone | `time_zone` | `America/Los_Angeles` |
| Accuracy radius (Advance) | `location.accuracy_radius` | `4.608` |
| Confidence (Advance) | `location.confidence` | `high` |
| Connection type (Advance) | `connection_type` | `Leased Line` |

Names are stored per language: replace `en` with `cs`, `de`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` or `zh`. A language without a translation holds an empty string.

## Security database

File: `db-ip-security.mmdb`. Fields sit at the top level of the record. In bundles that combine security data with other data in one file, they sit under `security.` (for example `security.is_vpn`).

| Field | Path | Example |
| --- | --- | --- |
| Threat score (0 to 100) | `threat_score` | `85` |
| Tor exit node | `is_tor` | `"true"` |
| VPN | `is_vpn` | `"false"` |
| Proxy | `is_proxy` | `"false"` |
| Residential proxy | `is_residential_proxy` | `"false"` |
| Relay (such as iCloud Private Relay) | `is_relay` | `"false"` |
| Anonymous | `is_anonymous` | `"true"` |
| Known attacker | `is_known_attacker` | `"true"` |
| Bot | `is_bot` | `"false"` |
| Known good bot (search engines) | `is_known_good_bot` | `"false"` |
| Spam source | `is_spam` | `"false"` |
| Cloud provider | `is_cloud_provider` | `"false"` |
| VPN provider names | `vpn_provider_names` | `["Nord VPN"]` |
| Proxy provider names | `proxy_provider_names` | `["Oxy Labs"]` |

Older Security Database releases have fewer fields; some have no `is_vpn` or `is_relay`.

## ASN database

File: `db-ip-asn.mmdb`.

| Field | Path | Example |
| --- | --- | --- |
| AS number (without the `AS` prefix) | `asn.as_number` | `"13335"` |
| AS organization | `asn.organization` | `Cloudflare, Inc.` |
| AS country | `asn.country_code` | `US` |
| AS domain | `asn.domain` | `cloudflare.com` |
| AS type | `asn.type` | `HOSTING` |

## ISP database

File: `db-ip-isp.mmdb`. The record is flat: the country sits at the top level, not under `location`.

| Field | Path | Example |
| --- | --- | --- |
| Country code | `country.code2` | `AU` |
| Country name | `country.name.en` | `Australia` |
| ISP | `isp` | `Example ISP` |
| AS number | `asn` | `"13335"` |
| AS organization | `as_organization` | `Cloudflare, Inc.` |
| AS country | `as_country` | `US` |
| Connection type | `connection_type` | `Cable` |

## Company database

File: `db-ip-company.mmdb`.

| Field | Path | Example |
| --- | --- | --- |
| Company name | `company.name.en` | `OVH Hosting, Inc.` |
| Company domain | `company.domain` | `ovh.com` |
| Company type | `company.type` | `hosting` |

## Value types

- Security flags are the strings `"true"` and `"false"`, not booleans. Compare them as strings (`is_tor == "true"`), or convert them in your pipeline.
- Scores (`threat_score`, confidence scores) are integers.
- Coordinates are strings such as `"36.17157"`. Convert them before using them as numbers.
- AS numbers are strings without the `AS` prefix.
- An unknown value is an empty string. An address with no record returns nothing.

## Keeping files up to date

IPGeolocation.io updates the databases regularly. Replace a file atomically: download it to a temporary file in the same directory, check it, then rename it over the old file. Never rewrite a file in place while a program has it open. Most tools read the file when they start, so restart or reload them after an update. Each guide notes what its tool needs.
