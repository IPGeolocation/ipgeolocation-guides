# Apache HTTP Server IP Geolocation and VPN Blocking with IPGeolocation.io

## Overview

With the open source `mod_maxminddb` module, Apache HTTP Server looks up the client address of every request in IPGeolocation.io databases and exposes the answers as environment variables. Because this happens inside the web server, before your application runs, you can log, forward, block or redirect requests by location and risk without changing application code.

This guide works with three databases. The [IP Geolocation Database](https://ipgeolocation.io/ip-geolocation-database.html) answers where a visitor is: country, region, city, coordinates and time zone. The [IP Security Database](https://ipgeolocation.io/ip-security-database.html) answers whether the connection hides behind a VPN, a proxy or Tor, and how risky it is overall. The [IP to ASN Database](https://ipgeolocation.io/ip-asn-database.html) names the [autonomous system](https://ipgeolocation.io/guides/what-is-an-asn) that announces the address and the organization behind it.

Apache can read any IPGeolocation.io IP database in MMDB format the same way. Besides the three above, that covers the IP to Country, IP to City and IP to ASN databases, the [IP to Company Database](https://ipgeolocation.io/ip-company-database.html), [IP Abuse Contact Database](https://ipgeolocation.io/ip-abuse-contact-database.html), [IP WHOIS Database](https://ipgeolocation.io/ip-whois-database.html), [IP to Hosting Database](https://ipgeolocation.io/ip-hosting-database.html) and [Residential Proxy Database](https://ipgeolocation.io/residential-proxy-database.html), and combined databases that hold several datasets in a single file.

Every lookup reads a file on the web server itself, so it costs no API call and no network time, and visitor addresses are never sent anywhere. If you are still weighing the options, read [IP geolocation API vs database](https://ipgeolocation.io/guides/ip-geolocation-api-vs-database-guide).

> [!TIP]
> A path in `MaxMindDBEnv` starts with the name you gave the file in `MaxMindDBFile` and uses slashes between keys, for example `LOCATION/location/country/code2`. The [MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md) writes paths with dots; replace each dot with a slash.

---

## What Apache can do with the variables

| Goal | Apache feature | Example in this guide |
| --- | --- | --- |
| Record | `mod_log_config` | Country, city, VPN flag and AS number in the access log |
| Hand over | `mod_headers`, PHP and CGI | An `X-IPGeo-Country` header for a backend, `$_SERVER` values in PHP |
| Restrict | `Require expr` | Refuse VPN, proxy and Tor traffic on `/login`, and high-risk addresses everywhere |
| Route | `mod_rewrite` | Send visitors from Germany, Austria and Switzerland to `/de/` |

## Where it pays off

- **Credential stuffing and fake sign-ups.** Turn away anonymized connections at the sign-in page before they reach your login code.
- **Localized landing pages.** Redirect by country in one rewrite rule, with no JavaScript and no extra request.
- **Older applications.** Give a legacy PHP or CGI application country and risk data through variables it can already read.
- **Log analysis.** Write location and network data into the access log, where existing log tools pick it up.

---

## About `mod_maxminddb`

`mod_maxminddb` is an open source module for Apache 2.4, released under the Apache License 2.0, that reads any file in the MMDB format. Ubuntu and Debian do not package it, so you build it from source; Fedora and EPEL ship it as the `mod_maxminddb` package.

The module adds five directives. `MaxMindDBFile` opens a database, `MaxMindDBEnv` maps a value to an environment variable, and `MaxMindDBEnable` switches lookups on. `MaxMindDBNetworkEnv` and `MaxMindDBSetNotes` are optional extras covered in the reference below.

---

## How a request gets its variables

1. If [`mod_remoteip`](https://httpd.apache.org/docs/2.4/mod/mod_remoteip.html) is set up, it replaces a trusted proxy's address with the visitor's address.
2. Early in request processing, `mod_maxminddb` looks that address up in every file declared with `MaxMindDBFile`.
3. For each `MaxMindDBEnv` whose path exists in the record, it sets an environment variable. It also sets `MMDB_ADDR` and `MMDB_INFO`.
4. Later modules use the variables: `mod_rewrite`, access rules, `mod_headers`, the access log, and PHP or CGI programs.

`mod_maxminddb` runs before `mod_rewrite` and `mod_setenvif`, so rewrite conditions and `SetEnvIf` rules can already see the values.

### From record to variable

A database record is a set of nested keys. This is the IP to ASN Database record for `37.120.202.92`:

```json
{
  "asn": {
    "as_number": "9009",
    "country_code": "RO",
    "domain": "m247.com",
    "organization": "M247 Europe SRL",
    "type": "HOSTING"
  }
}
```

If you open that file as `ASN`, the line `MaxMindDBEnv IPGEO_ASN ASN/asn/as_number` sets `IPGEO_ASN` to `9009`, and `ASN/asn/organization` gives `M247 Europe SRL`.

---

## Requirements

| Component | Details |
| --- | --- |
| Apache HTTP Server | 2.4 (tested with 2.4.52 on Ubuntu 22.04). `mod_remoteip`, `mod_headers` and `mod_rewrite` ship with Apache. |
| `mod_maxminddb` | 1.3.0 (tested), with `libmaxminddb`. Installation is below. |
| IPGeolocation.io databases | MMDB files for the data you need, from your [IPGeolocation.io account](https://app.ipgeolocation.io). Compare plans on the [IP database pricing page](https://ipgeolocation.io/db-pricing.html). |

---

## Installing `mod_maxminddb`

### Debian and Ubuntu

Build the module against your Apache, then enable it together with `mod_remoteip` and `mod_headers`:

```sh
sudo apt-get install -y apache2 apache2-dev libmaxminddb-dev build-essential curl
curl -fsSL https://github.com/maxmind/mod_maxminddb/releases/download/1.3.0/mod_maxminddb-1.3.0.tar.gz | tar xz
cd mod_maxminddb-1.3.0 && ./configure && sudo make install
echo 'LoadModule maxminddb_module /usr/lib/apache2/modules/mod_maxminddb.so' | sudo tee /etc/apache2/mods-available/maxminddb.load
sudo a2enmod remoteip headers maxminddb
```

### Fedora, RHEL, Rocky Linux and AlmaLinux

Install the packaged module. On RHEL and its rebuilds, enable [EPEL](https://docs.fedoraproject.org/en-US/epel/) first:

```sh
sudo dnf install -y mod_maxminddb
```

Put the configuration from the next section in a file under `/etc/httpd/conf.d/`.

### Check that the module is loaded

```sh
apachectl -M | grep maxminddb
```

```text
 maxminddb_module (shared)
```

---

## Quick start

These steps add location, security and network variables to every request on a Debian or Ubuntu server, then show them with a test request.

### Step 1: Store the databases

Copy the three MMDB files from your account to `/usr/local/share/ipgeolocation`:

```text
/usr/local/share/ipgeolocation/
├── db-ip-asn.mmdb
├── db-ip-location.mmdb
└── db-ip-security.mmdb
```

### Step 2: Configure the lookups

Save this as `/etc/apache2/conf-available/ipgeolocation.conf`:

```apache
# Use the visitor's address when a proxy or load balancer sits in front of
# Apache. List only proxies you control.
RemoteIPHeader X-Forwarded-For
RemoteIPInternalProxy 127.0.0.1

MaxMindDBEnable On
MaxMindDBFile LOCATION /usr/local/share/ipgeolocation/db-ip-location.mmdb
MaxMindDBFile SECURITY /usr/local/share/ipgeolocation/db-ip-security.mmdb
MaxMindDBFile ASN      /usr/local/share/ipgeolocation/db-ip-asn.mmdb

MaxMindDBEnv IPGEO_COUNTRY_CODE LOCATION/location/country/code2
MaxMindDBEnv IPGEO_COUNTRY_NAME LOCATION/location/country/name/en
MaxMindDBEnv IPGEO_REGION       LOCATION/location/state/name/en
MaxMindDBEnv IPGEO_CITY         LOCATION/location/city/name/en
MaxMindDBEnv IPGEO_LATITUDE     LOCATION/location/coordinates/latitude
MaxMindDBEnv IPGEO_LONGITUDE    LOCATION/location/coordinates/longitude
MaxMindDBEnv IPGEO_TIME_ZONE    LOCATION/time_zone

MaxMindDBEnv IPGEO_THREAT_SCORE SECURITY/threat_score
MaxMindDBEnv IPGEO_IS_VPN       SECURITY/is_vpn
MaxMindDBEnv IPGEO_IS_PROXY     SECURITY/is_proxy
MaxMindDBEnv IPGEO_IS_TOR       SECURITY/is_tor
MaxMindDBEnv IPGEO_IS_ANONYMOUS SECURITY/is_anonymous

MaxMindDBEnv IPGEO_ASN          ASN/asn/as_number
MaxMindDBEnv IPGEO_AS_ORG       ASN/asn/organization
```

The directives are not wrapped in `<IfModule mod_maxminddb.c>` on purpose. If the module is missing, `apachectl configtest` fails loudly instead of Apache running without the variables and silently skipping any rule that depends on them.

`RemoteIPInternalProxy 127.0.0.1` trusts `X-Forwarded-For` only from the server itself, which also lets you test with `curl` in Step 3. In production, list the addresses of your own load balancers or reverse proxies instead. If clients connect to Apache directly, the two `RemoteIP` lines do no harm.

Enable the configuration and reload Apache:

```sh
sudo a2enconf ipgeolocation
sudo apachectl configtest && sudo systemctl reload apache2
```

### Step 3: Make a test request

Temporarily echo a few values back as response headers by adding these lines to the same file, and reload Apache again:

```apache
# Testing only: echo some of the values back as response headers.
Header set X-IPGeo-Country "%{IPGEO_COUNTRY_CODE}e" env=IPGEO_COUNTRY_CODE
Header set X-IPGeo-City    "%{IPGEO_CITY}e"         env=IPGEO_CITY
Header set X-IPGeo-VPN     "%{IPGEO_IS_VPN}e"       env=IPGEO_IS_VPN
Header set X-IPGeo-Threat  "%{IPGEO_THREAT_SCORE}e" env=IPGEO_THREAT_SCORE
Header set X-IPGeo-ASN     "%{IPGEO_ASN}e"          env=IPGEO_ASN
```

From the server, pretend to be a visitor at `37.120.202.92`:

```sh
curl -s -o /dev/null -D - -H 'X-Forwarded-For: 37.120.202.92' http://localhost/ | grep -i '^x-ipgeo'
```

### Step 4: Read the response

```text
X-IPGeo-Country: US
X-IPGeo-City: Secaucus
X-IPGeo-VPN: true
X-IPGeo-Threat: 50
X-IPGeo-ASN: 9009
```

`37.120.202.92` is a commercial VPN exit hosted by M247, so the VPN flag is `true`. Your values may change with newer database releases. The [IPGeolocation.io page for AS9009](https://ipgeolocation.io/browse/asn/AS9009) lists the routes, peers and WHOIS details of that network.

Delete the `Header set X-IPGeo-*` lines when you are done, so visitors do not see the values.

---

## Configuration reference

### Directives

| Directive | Syntax | Purpose |
| --- | --- | --- |
| `MaxMindDBEnable` | `MaxMindDBEnable On` | Turns lookups on for the server or virtual host. |
| `MaxMindDBFile` | `MaxMindDBFile NAME /path/to/file.mmdb` | Opens a database and gives it a name for use in paths. |
| `MaxMindDBEnv` | `MaxMindDBEnv VARIABLE NAME/key/key` | Sets `VARIABLE` to the value at that path. |
| `MaxMindDBNetworkEnv` | `MaxMindDBNetworkEnv NAME VARIABLE` | Sets `VARIABLE` to the network block that matched, such as `37.120.202.92/32`. |
| `MaxMindDBSetNotes` | `MaxMindDBSetNotes On` | Also stores every value as a request note, readable in `LogFormat` as `%{VARIABLE}n`. |

### Variables the module always sets

| Variable | Value |
| --- | --- |
| `MMDB_ADDR` | The address that was looked up. |
| `MMDB_INFO` | `result found` when a database had a record for the address, `lookup success` when it did not. |

### A variable for each database

Every IPGeolocation.io IP database can be opened with `MaxMindDBFile`. This table shows one sample path per database, using the name in the first part of the path as the `MaxMindDBFile` name:

| Database | Sample `MaxMindDBEnv` path |
| --- | --- |
| IP Geolocation Database or IP to City Database | `LOCATION/location/city/name/en` |
| IP to Country Database | `COUNTRY/location/country/code2` |
| IP Security Database | `SECURITY/is_vpn` |
| IP to ASN Database | `ASN/asn/as_number` |
| IP to Company Database | `COMPANY/company/name/en` |
| IP to ISP Database | `ISP/isp` and `ISP/asn` |
| IP Abuse Contact Database | `ABUSE/abuse/emails` |
| IP WHOIS Database | `WHOIS/whois/rir` |
| IP to Hosting Database | `HOSTING/hosting_provider` |
| Residential Proxy Database | `RESIDENTIAL_PROXY/proxy_provider` |

The IP to ISP Database is laid out flat: `ISP/asn` is the AS number itself, and the country code sits at `ISP/country/code2`. ISP names and AS owners can differ for the same address; [ISP versus ASN explained](https://ipgeolocation.io/guides/what-is-an-isp-and-how-is-it-different-from-an-asn) covers why.

### More security variables

The quick start reads five security values. These lines add the remaining flags, the confidence scores and the first VPN provider name:

```apache
MaxMindDBEnv IPGEO_IS_RESIDENTIAL_PROXY SECURITY/is_residential_proxy
MaxMindDBEnv IPGEO_IS_RELAY             SECURITY/is_relay
MaxMindDBEnv IPGEO_IS_KNOWN_ATTACKER    SECURITY/is_known_attacker
MaxMindDBEnv IPGEO_IS_BOT               SECURITY/is_bot
MaxMindDBEnv IPGEO_IS_SPAM              SECURITY/is_spam
MaxMindDBEnv IPGEO_IS_CLOUD_PROVIDER    SECURITY/is_cloud_provider
MaxMindDBEnv IPGEO_CLOUD_PROVIDER       SECURITY/cloud_provider_name
MaxMindDBEnv IPGEO_VPN_CONFIDENCE       SECURITY/vpn_confidence_score
MaxMindDBEnv IPGEO_PROXY_CONFIDENCE     SECURITY/proxy_confidence_score
MaxMindDBEnv IPGEO_VPN_PROVIDER         SECURITY/vpn_provider_names/0
```

What each signal means, from relay services to [residential proxies](https://ipgeolocation.io/residential-proxy-database.html), is described in the [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html).

### A combined database

When one file holds several datasets, open it once and read every part of it:

```apache
MaxMindDBEnable On
MaxMindDBFile CITY_COMPANY_ASN /usr/local/share/ipgeolocation/db-ip-city-company-asn.mmdb

MaxMindDBEnv IPGEO_COUNTRY_CODE CITY_COMPANY_ASN/location/country/code2
MaxMindDBEnv IPGEO_CITY         CITY_COMPANY_ASN/location/city/name/en
MaxMindDBEnv IPGEO_ASN          CITY_COMPANY_ASN/asn/as_number
MaxMindDBEnv IPGEO_COMPANY      CITY_COMPANY_ASN/company/name/en
```

---

## Recipes

Each recipe builds on the quick start configuration.

### Add location and VPN data to the access log

```apache
LogFormat "%a %t \"%r\" %>s country=%{IPGEO_COUNTRY_CODE}e city=\"%{IPGEO_CITY}e\" vpn=%{IPGEO_IS_VPN}e threat=%{IPGEO_THREAT_SCORE}e asn=%{IPGEO_ASN}e" ipgeo
CustomLog ${APACHE_LOG_DIR}/access-ipgeo.log ipgeo
```

A request from the VPN address above is logged as:

```bash
37.120.202.92 [06/Oct/2026:13:11:05 +0500] "GET / HTTP/1.1" 200 country=US city="Secaucus" vpn=true threat=50 asn=9009
```

An address with no record logs `-` in each field. For what this extra context does for investigations, see [why logs need IP enrichment](https://ipgeolocation.io/guides/what-is-ip-enrichment-and-why-logs-need-it).

### Pass the data to your application

With `mod_php`, every variable is already in `$_SERVER`:

```php
<?php
$country = $_SERVER['IPGEO_COUNTRY_CODE'] ?? null;
$isVpn   = ($_SERVER['IPGEO_IS_VPN'] ?? '') === 'true';
```

For an application behind `ProxyPass`, send the values as request headers. Remove any copy the client sent first, or a visitor could supply their own:

```apache
# Drop any copy the client sent, then add the value Apache looked up.
RequestHeader unset X-IPGeo-Country
RequestHeader set X-IPGeo-Country "%{IPGEO_COUNTRY_CODE}e" env=IPGEO_COUNTRY_CODE
```

In testing, a request that arrived with a forged `X-IPGeo-Country: ZZ` header reached the application with `US`, the value from the database.

### Block anonymous and high-risk traffic

This refuses any request with a threat score of 80 or more, and on `/login` also refuses VPN, proxy and Tor connections:

```apache
# Refuse every request with a threat score of 80 or more.
<Location "/">
    <RequireAll>
        Require all granted
        Require not expr "-n %{ENV:IPGEO_THREAT_SCORE} && %{ENV:IPGEO_THREAT_SCORE} -ge 80"
    </RequireAll>
</Location>

# On the sign-in page, also refuse VPNs, proxies and Tor.
<Location "/login">
    AuthMerging And
    <RequireAll>
        Require all granted
        Require not expr "%{ENV:IPGEO_IS_ANONYMOUS} == 'true'"
    </RequireAll>
</Location>
```

Two details matter here. Apache applies `<Location>` sections in the order they appear, and without `AuthMerging And` the `/login` rules would replace the site-wide rule instead of adding to it. And a `Require not` line only works inside `<RequireAll>` next to a positive rule such as `Require all granted`.

Results from a test run:

| Visitor | `/` | `/login/` |
| --- | --- | --- |
| VPN exit, threat score 50 | `200` | `403` |
| Tor exit, threat score 45 | `200` | `403` |
| Known attacker, threat score 85 | `403` | `403` |
| Clean address, threat score 0 | `200` | `200` |
| Address with no security record | `200` | `200` |

### Redirect visitors by country

```apache
RewriteEngine On
RewriteCond %{ENV:IPGEO_COUNTRY_CODE} ^(DE|AT|CH)$
RewriteRule ^/$ /de/ [R=302,L]
```

Place it in the server or virtual host configuration. A visitor from Frankfurt who opens `/` receives `302 Found` pointing to `/de/`; visitors from other countries stay on `/`.

---

## Working with the values

| Value | Example | Notes |
| --- | --- | --- |
| Security flags | `true`, `false` | Plain text. Compare with `== 'true'`. |
| Scores | `50` | Whole numbers 0 to 100. Use `-ge` and `-gt` in expressions for numeric comparisons. |
| Coordinates | `40.78834` | Decimal degrees. |
| AS number | `9009` | No `AS` prefix. |
| Empty field | (empty) | The variable exists but holds an empty string. |
| Path not in the record | | The variable is not set. |
| Address not in the database | | Only `MMDB_ADDR` and `MMDB_INFO` are set. |

**Lists.** Provider names are lists. A path that ends at the list, such as `SECURITY/vpn_provider_names`, sets nothing and writes `Database error: unknown data type` to the error log on every request. Add an index, `SECURITY/vpn_provider_names/0`, to read the first name.

**Languages.** Place names exist in `cs`, `de`, `en`, `es`, `fa`, `fr`, `it`, `ja`, `ko`, `pt`, `ru` and `zh`. Change the last part of the path, as in `LOCATION/location/city/name/de`. A missing translation gives an empty value.

---

## Keeping the databases up to date

IPGeolocation.io issues new database releases daily or weekly, depending on your plan. Apache opens each file when it reads its configuration, so a replaced file is used only after a graceful restart. `apachectl graceful` lets current requests finish while new workers pick up the new files.

Each release arrives as a ZIP archive. It holds the MMDB file of the database, or several files for a combined plan, along with a `README.md` and a `checksum.txt` with the SHA-256 hash of every file. Installing a release safely takes four steps:

1. Download and unpack the archive in a temporary folder on the same disk as the live databases.
2. Check every file against `checksum.txt`, and check each database with the [mmdbio command-line tool](https://ipgeolocation.io/cli/mmdbio). A failed check stops the update and leaves the live files untouched.
3. Move each new MMDB file over the old one. A rename within one disk is atomic, so Apache never reads a half-written file.
4. Run `apachectl configtest`, then `apachectl graceful`.

This script performs all four. Set `DOWNLOAD_URL` to the MMDB download link from your IPGeolocation.io account; it needs `curl`, `unzip`, `sha256sum` and `mmdbio`:

```sh
#!/bin/sh
# Install a new IPGeolocation.io database release and restart Apache gracefully.
set -eu

DB_DIR=/usr/local/share/ipgeolocation
DOWNLOAD_URL="<MMDB download link from your IPGeolocation.io account>"

# 1. Download and unpack the release next to the live databases.
WORK=$(mktemp -d "$DB_DIR/.release.XXXXXX")
trap 'rm -rf "$WORK"' EXIT
curl -fsSL -o "$WORK/release.zip" "$DOWNLOAD_URL"
# -DD stamps the unpacked files with the current time, not the archive's.
unzip -q -DD "$WORK/release.zip" -d "$WORK"
rm "$WORK/release.zip"

# 2. Check the files against the release checksums, then check each database.
(cd "$WORK" && sha256sum --quiet -c checksum.txt)
for db in "$WORK"/*.mmdb; do
    mmdbio verify --db "$db"
done

# 3. Swap the databases in. Each rename is atomic.
for db in "$WORK"/*.mmdb; do
    mv "$db" "$DB_DIR/"
done

# 4. Restart gracefully, but only if the configuration still loads.
apachectl configtest && apachectl graceful
```

The archive keeps the same file names from release to release, so the paths in `MaxMindDBFile` never change. For a combined plan, the script installs every MMDB file in the archive in one run.

Run it as root from cron, on the same schedule as your plan's releases. If any check fails, the script exits with an error, deletes the temporary folder and leaves Apache serving the previous release.

---

## Production checklist

- **Trust only your own proxies.** `RemoteIPInternalProxy` and `RemoteIPTrustedProxy` decide whose `X-Forwarded-For` header Apache believes. A proxy missing from the list makes every visitor look like the proxy; an address that should not be on it lets clients choose their own location.
- **Never forward client-supplied copies.** Always `RequestHeader unset` an `X-IPGeo-*` header before setting it.
- **Load the module explicitly.** Keep the directives outside `<IfModule>`, so a missing module stops `apachectl configtest` instead of quietly disabling your access rules.
- **Mind the order of `<Location>` sections.** A later section replaces earlier access rules unless it uses `AuthMerging And`.
- **Remove test headers.** The `X-IPGeo-*` response headers from the quick start are for testing only.

---

## Troubleshooting

**No variables are set, and nothing appears in the error log.** The module is not loaded, but its directives sit inside `<IfModule mod_maxminddb.c>`, so Apache skips them. Run `apachectl -M | grep maxminddb`, and enable the module with `a2enmod maxminddb`.

**`Invalid command 'MaxMindDBEnable', perhaps misspelled or defined by a module not included in the server configuration`.** The module is not loaded. Enable it as above.

**`MaxMindDBFile: Failed to open ...`.** Apache cannot open the file at that path. Check the spelling and the file permissions.

**`Database error: unknown data type`.** A `MaxMindDBEnv` path ends at a list or a group, such as `SECURITY/vpn_provider_names` or `LOCATION/location/country/name`. Add an index (`/0`) or a language (`/en`).

**Every visitor gets the location of your load balancer.** `mod_remoteip` is not enabled, `RemoteIPHeader` names the wrong header, or the load balancer is missing from `RemoteIPInternalProxy`.

**`negative Require directive has no effect in <RequireAny> directive`.** A `Require not` line stands on its own. Wrap it in `<RequireAll>` together with `Require all granted`.

**The `/login` rule never blocks anything.** A later `<Location "/">` section replaced it. Put the broader section first and add `AuthMerging And` to the narrower one.

**`MMDB_INFO` says `lookup success` but other variables are missing.** The address is not in that database. Check it with mmdbio:

```sh
mmdbio read --db /usr/local/share/ipgeolocation/db-ip-location.mmdb --ip 37.120.202.92
```

**A new database release has no effect.** Apache still holds the previous file. Run `apachectl graceful`.

---

## FAQ

<details>
<summary><strong>Does this work behind a CDN or load balancer?</strong></summary>
Yes. Configure `mod_remoteip` with the header your proxy sets and list the proxy addresses as trusted. `mod_maxminddb` then looks up the visitor's address instead of the proxy's.
</details>

<details>
<summary><strong>Can PHP applications read the values?</strong></summary>
Yes. With `mod_php`, each variable appears in `$_SERVER`, for example `$_SERVER['IPGEO_COUNTRY_CODE']`. For applications behind `ProxyPass`, forward the values as request headers.
</details>

<details>
<summary><strong>Is an IPGeolocation.io API key involved?</strong></summary>
No. Apache reads the downloaded files directly and makes no outgoing requests. Your database plan only matters when you download new releases.
</details>

<details>
<summary><strong>Are visitors on IPv6 covered?</strong></summary>
Yes. IPv6 addresses are looked up the same way as IPv4 addresses, and the databases hold data for both.
</details>

<details>
<summary><strong>Which IPGeolocation.io databases can Apache use?</strong></summary>
All of the IP databases, in their MMDB edition, including combined files. [A variable for each database](#a-variable-for-each-database) above has a sample path for every one.
</details>

<details>
<summary><strong>Does installing a new database release need a full restart?</strong></summary>
No. A graceful restart with `apachectl graceful` loads the new file while current requests finish.
</details>

---

## Related

- [IPGeolocation.io MMDB field reference](https://github.com/IPGeolocation/ipgeolocation-guides/blob/main/integrations/mmdb-field-reference/README.md)
- [IP Geolocation Database documentation](https://ipgeolocation.io/documentation/ip-geolocation-advance-database.html)
- [IP Security Database documentation](https://ipgeolocation.io/documentation/ip-security-database.html)
- [IP to ASN Database documentation](https://ipgeolocation.io/documentation/ip-asn-database-lite.html)
- [IP to Company Database documentation](https://ipgeolocation.io/documentation/ip-company-database.html)
- [IPGeolocation.io Nginx module, for sites that run Nginx](https://ipgeolocation.io/documentation/nginx-integration)
- [All IPGeolocation.io integrations](https://ipgeolocation.io/integrations.html)
