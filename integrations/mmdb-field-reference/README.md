# IPGeolocation.io MMDB Field Reference

## Overview

This page lists every field in the MMDB edition of IPGeolocation.io's IP databases: where it sits in the record, how it is stored and what a real value looks like. Use it when you write lookups by hand in a log pipeline, a web server or a SQL query.

Each tool spells paths its own way, so the first table shows how one path looks in every tool that has an IPGeolocation.io guide. The tables after it cover the databases one by one, from the IP Geolocation Database to the Residential Proxy Database, followed by combined databases, value types, languages and updates.

Unless a table says otherwise, examples come from the record for `37.120.202.92`, a VPN exit in a New Jersey data center.

---

## One path, five tools

Paths on this page use dots between keys, such as `location.country.code2`. This is how the same path is written in each integration:

| Tool | Syntax for `location.country.code2` | Guide |
| --- | --- | --- |
| Fluent Bit (`geoip2` filter) | `%{location.country.code2}` | [Fluent Bit log enrichment guide](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/fluent-bit/README.md) |
| Grafana Alloy (`stage.geoip`) | `"location.country.code2"` in `custom_lookups` | [Grafana Alloy and Loki guide](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/grafana-alloy/README.md) |
| Vector (VRL) | `rec.location.country.code2` on the looked-up record | [Vector enrichment table guide](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/vector/README.md) |
| Apache HTTP Server (`mod_maxminddb`) | `NAME/location/country/code2`, where `NAME` comes from `MaxMindDBFile` | [Apache HTTP Server guide](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/apache/README.md) |
| DuckDB | `mmdb_record(file, ip, 'location') ->> '$.location.country.code2'` | [DuckDB SQL guide](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/duckdb/README.md) |

A list entry adds an index: `vpn_provider_names.0` in Fluent Bit and Apache (`/0`), `vpn_provider_names[0]` in Alloy, Vector and DuckDB JSONPath.

### See a record for yourself

The [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio) prints the full record a file holds for an address, or only the fields you name:

```sh
mmdbio read --db db-ip-location.mmdb --ip 37.120.202.92 --fields location.country.code2,location.city.name.en,time_zone
```

```json
{
  "37.120.202.92": {
    "location.city.name.en": "Secaucus",
    "location.country.code2": "US",
    "time_zone": "America/New_York"
  }
}
```

---

## IP Geolocation Database and IP to City Database

Reference: [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html). Names marked `<lang>` exist once per language (see "Languages" below).

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `location.country.code2` | text | `US` | ISO 3166-1 alpha-2 country code |
| `location.country.code3` | text | `USA` | ISO 3166-1 alpha-3 country code |
| `location.country.code_ioc` | text | `USA` | International Olympic Committee country code |
| `location.country.name.<lang>` | text | `United States` | Country name |
| `location.country.name_official.<lang>` | text | `United States of America` | Official country name |
| `location.country.capital.<lang>` | text | `Washington, D.C.` | Capital city |
| `location.country.continent.code` | text | `NA` | Two-letter continent code |
| `location.country.continent.name.<lang>` | text | `North America` | Continent name |
| `location.country.currency.code` | text | `USD` | ISO 4217 currency code |
| `location.country.currency.name.en` | text | `US Dollar` | Currency name, English only |
| `location.country.currency.symbol` | text | `$` | Currency symbol |
| `location.country.metadata.calling_code` | text | `+1` | International dialing code |
| `location.country.metadata.tld` | text | `.us` | Country code top-level domain |
| `location.country.metadata.languages` | text | `en-US,es-US,haw,fr` | Languages spoken, comma-separated |
| `location.state.code` | text | `US-NJ` | ISO 3166-2 subdivision code |
| `location.state.name.<lang>` | text | `New Jersey` | State, province or region |
| `location.district.name.<lang>` | text | `Hudson` | District or county |
| `location.city.name.<lang>` | text | `Secaucus` | City |
| `location.zipcode` | text | `07094` | Postal code |
| `location.coordinates.latitude` | text | `40.78834` | Latitude in decimal degrees |
| `location.coordinates.longitude` | text | `-74.05502` | Longitude in decimal degrees |
| `location.geoname_id` | text | `5099438` | GeoNames identifier of the place |
| `time_zone` | text | `America/New_York` | IANA time zone name |

The advanced edition adds four fields:

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `location.accuracy_radius` | text | `10.251` | Accuracy radius in kilometers |
| `location.confidence` | text | `medium` | Confidence in the location: `high`, `medium` or `low` |
| `location.dma_code` | text | `501` | US Designated Market Area code, empty elsewhere |
| `connection_type` | text | `DSL` | Access technology, such as `Cable`, `DSL`, `Fiber`, `Mobile`, `Fixed Wireless`, `Satellite` or `Dial-Up` |

## IP to Country Database

Reference: [IP to Country Database documentation](https://ipgeolocation.io/documentation/ip-country-database.html).

The record holds only the `location.country` group, with the same paths and values as the country rows above, from `location.country.code2` to `location.country.metadata.languages`.

## IP Security Database

Reference: [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html). All fields sit at the top level of the record.

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `threat_score` | integer | `50` | Overall risk, 0 to 100; higher is riskier |
| `is_anonymous` | text flag | `true` | Traffic hides its origin, for example through a VPN, proxy or Tor |
| `is_vpn` | text flag | `true` | VPN exit |
| `vpn_provider_names` | list of text | `["Private Internet Access VPN"]` | VPN services seen on the range |
| `vpn_confidence_score` | integer | `99` | Confidence in the VPN result, 0 to 100 |
| `vpn_last_seen` | text | `2026-09-28` | Last date VPN use was observed |
| `is_proxy` | text flag | `true` | Proxy |
| `proxy_provider_names` | list of text | `[]` | Proxy services seen on the range |
| `proxy_confidence_score` | integer | `99` | Confidence in the proxy result, 0 to 100 |
| `proxy_last_seen` | text | `2026-09-08` | Last date proxy use was observed |
| `is_residential_proxy` | text flag | `false` | Proxy that routes through home connections |
| `is_tor` | text flag | `false` | Tor exit node |
| `is_relay` | text flag | `false` | Relay service, such as iCloud Private Relay |
| `relay_provider_name` | text | (empty) | Relay operator |
| `is_known_attacker` | text flag | `false` | Range linked to attacks |
| `is_spam` | text flag | `false` | Range linked to spam |
| `is_bot` | text flag | `false` | Automated traffic |
| `bot_type` | text | (empty) | Kind of automation, such as `scanner`, `brute_force`, `credential_stuffing`, `exploit`, `worm`, `seo_crawler` or `site_monitor` |
| `bot_operator_name` | text | (empty) | Operator of the bot, when known |
| `bot_confidence_score` | integer | `0` | Confidence in the bot result, 0 to 100 |
| `bot_last_seen` | text | (empty) | Last date bot traffic was observed |
| `is_known_good_bot` | text flag | `false` | Verified, well-behaved crawler, such as a search engine |
| `is_cloud_provider` | text flag | `true` | Range belongs to a cloud, hosting or infrastructure provider |
| `cloud_provider_name` | text | `M247` | Name of that provider |
| `is_corporate_gateway` | text flag | `false` | Corporate egress gateway |
| `corporate_gateway_type` | text | (empty) | Kind of gateway, such as `secure_web_gateway` |
| `corporate_gateway_provider_name` | text | (empty) | Gateway vendor, when known |

Plans that combine security data with other datasets still deliver it as a separate IP Security Database file, with this same top-level layout.

## IP to ASN Database

Reference: [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html).

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `asn.as_number` | text | `9009` | AS number, without the `AS` prefix |
| `asn.organization` | text | `M247 Europe SRL` | Organization the AS is registered to |
| `asn.country_code` | text | `RO` | Country the AS is registered in |
| `asn.domain` | text | `m247.com` | Domain of the organization |
| `asn.type` | text | `HOSTING` | Network type: mostly `ISP`, `HOSTING`, `BUSINESS`, `EDUCATION`, `GOVERNMENT` or `INDIVIDUAL` |

The extended edition, described in the [extended IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-extended.html), adds routing and registry data. The examples are real values from several records; any of these fields can also be empty:

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `asn.as_name` | text | `AS-ENECOM` | Registered AS name |
| `asn.allocation_status` | text | `ASSIGNED` | Registry allocation status |
| `asn.date_allocated` | text | `2002-03-21` | Allocation date |
| `asn.whois_host` | text | `JPNIC` | Registry that holds the WHOIS record |
| `asn.peers` | text | `AS7670` | Peer networks, comma-separated |
| `asn.upstreams` | text | `AS7670` | Upstream providers, comma-separated |
| `asn.downstreams` | text | `AS56120,AS131257,...` | Downstream customers, comma-separated |
| `asn.routes` | text | `116.81.0.0/16,223.223.0.0/17,...` | Announced prefixes in CIDR notation, comma-separated |

## IP to Company Database

Reference: [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html).

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `company.name.<lang>` | text | `M247 New Jersey Infrastructure` | Organization using the address |
| `company.domain` | text | `m247.com` | Its domain |
| `company.type` | text | `BUSINESS` | Organization type, such as `ISP`, `HOSTING`, `BUSINESS`, `EDUCATION` or `GOVERNMENT` |

## IP to ISP Database

The ISP fields sit at the top level of the record, and the country group is `country`, not `location.country`:

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `isp` | text | `M247 New Jersey Infrastructure` | Internet service provider |
| `asn` | text | `9009` | AS number of the network |
| `as_organization` | text | `M247 Europe SRL` | Organization the AS is registered to |
| `as_country` | text | `RO` | Country the AS is registered in |
| `connection_type` | text | `DSL` | Access technology |
| `country.code2` | text | `US` | Country code; the rest of the group matches `location.country` above |

ISP and AS data can name different organizations for the same address, for the reasons covered in [what an ISP is and how it differs from an ASN](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn).

## IP Abuse Contact Database

Reference: [IP Abuse Contact Database documentation](https://ipgeolocation.io/documentation/ip-abuse-contact-database.html).

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `abuse.route` | text | `37.120.202.0/24` | Network the contact is responsible for |
| `abuse.name.<lang>` | text | `Secure Data Systems` | Contact name |
| `abuse.kind` | text | `group` | Contact kind: `group`, `individual` or `org` |
| `abuse.emails` | text | `abuse@s-data.ro` | Email addresses, comma-separated |
| `abuse.phone_numbers` | text | | Phone numbers, comma-separated |
| `abuse.address` | text | | Postal address |
| `abuse.country_code` | text | `RO` | Country of the contact |

## IP WHOIS Database

Reference: [IP WHOIS Database documentation](https://ipgeolocation.io/documentation/ip-whois-database.html). Examples come from the record for `2.119.4.80`.

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `whois.name` | text | `INFRASTRUTTUREETELECOMUNICAZIONIPERLITALIASPA` | Network name in the registry |
| `whois.domain` | text | `business.telecomitalia.it` | Domain in the registry record |
| `whois.country` | text | `IT` | Country in the registry record |
| `whois.rir` | text | `ripencc` | Regional registry: `afrinic`, `apnic`, `arin`, `lacnic` or `ripencc` |
| `whois.date_created` | text | `2023-10-23T06:43:28` | When the record was created |
| `whois.date_updated` | text | `2023-10-23T06:43:28` | When the record last changed |
| `whois.raw_whois` | text | | Full WHOIS response as text |
| `whois.organization.handle` | text | | Registry handle of the organization |
| `whois.organization.name` | text | | Organization name |
| `whois.organization.type` | text | | Organization type, such as `lir` or `other` |
| `whois.organization.address` | text | | Postal address |
| `whois.organization.country` | text | | Country |
| `whois.organization.email` | text | | Email address |
| `whois.organization.phone` | text | | Phone number |
| `whois.organization.fax` | text | | Fax number |
| `whois.organization.date_updated` | text | | When the organization entry last changed |
| `whois.organization.source` | text | | Registry the entry came from |

Contacts come in four lists: `whois.admin_handles`, `whois.tech_handles`, `whois.abuse_handles` and `whois.irt_handles` (incident response team). Each entry has `handle`, `name`, `address`, `country`, `email`, `phone`, `fax`, `date_updated` and `source`. Read an entry by index, as in `whois.admin_handles.0.name`. A list can be empty, and the `organization` group can be missing or partly filled.

## IP to Hosting Database

Reference: [IP to Hosting Database documentation](https://ipgeolocation.io/documentation/ip-hosting-database.html). Example from `1.0.0.0`.

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `hosting_provider` | text | `Cloudflare, Inc.` | Hosting or infrastructure provider of the address |

## Residential Proxy Database

Reference: [Residential Proxy Database documentation](https://ipgeolocation.io/documentation/residential-proxy-database.html). Example from `1.0.141.172`.

| Path | Type | Example | Meaning |
| --- | --- | --- | --- |
| `proxy_provider` | text | `ProxyShare` | Residential proxy service using the address |
| `last_seen` | text | `2026-08-19` | Last date the address was seen in use |

---

## Combined databases

A combined database merges the records of the datasets it contains, and each dataset keeps its own paths:

| Combination | Paths in the one file |
| --- | --- |
| City and ISP | `location.*` and `time_zone`, plus the top-level ISP fields `isp`, `asn`, `as_organization`, `as_country` and `connection_type` |
| City, Company and ASN | `location.*`, `time_zone`, `connection_type`, `company.*` and `asn.*` |
| City, Company, ASN and Abuse Contact | As above, plus `abuse.*` |

In a City and ISP file, `asn` is the AS number itself, as in the IP to ISP Database. In the other combinations it is the `asn` group, so the number is `asn.as_number`.

---

## Value types

| Stored as | Fields | Notes |
| --- | --- | --- |
| Text flag | Every `is_*` field | The text `true` or `false`, not a boolean. Compare with the string `"true"`, or convert. |
| Integer | `threat_score` and the `*_confidence_score` fields | Real numbers in the file. |
| Text holding a number | Coordinates, `accuracy_radius`, `geoname_id`, `dma_code`, AS numbers | Convert before numeric work, for example for a geo point. |
| List of text | `vpn_provider_names`, `proxy_provider_names` | Often empty. Read one entry by index. |
| List of groups | The WHOIS `*_handles` fields | Read one entry by index, then a key. |
| Empty text | Any text field without data | The field exists, with nothing in it. |

An address that a database does not cover has no record at all, so every lookup against that file returns nothing. Each tool reports this its own way, as a `null`, a missing variable or an empty result; its guide gives the details.

## Languages

Fields marked `<lang>` hold one value per language. Replace `<lang>` with one of these keys:

| Key | Language | Key | Language |
| --- | --- | --- | --- |
| `cs` | Czech | `it` | Italian |
| `de` | German | `ja` | Japanese |
| `en` | English | `ko` | Korean |
| `es` | Spanish | `pt` | Portuguese |
| `fa` | Persian | `ru` | Russian |
| `fr` | French | `zh` | Chinese |

---

## Database updates in each tool

IPGeolocation.io publishes new releases daily or weekly, depending on the plan. In every tool, install a release with an atomic rename: write the new file next to the old one, then rename it over the old name. What happens next depends on the tool:

| Tool | How a new file takes effect |
| --- | --- |
| Fluent Bit | Send `SIGHUP` with `hot_reload: on`, or restart |
| Grafana Alloy | Restart, or switch files through a pointer file read by `local.file` |
| Vector | Send `SIGHUP` |
| Apache HTTP Server | `apachectl graceful` |
| DuckDB | Nothing to do; the next query reads the new file |

Each guide includes a tested update script for its tool.

---

## Related

- [Overview of IPGeolocation.io databases](https://ipgeolocation.io/documentation/databases.html)
- [IP database pricing and plans](https://ipgeolocation.io/db-pricing.html)
- [IPGeolocation.io Nginx module for MMDB databases](https://ipgeolocation.io/documentation/nginx-integration)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
