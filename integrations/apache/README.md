# IPGeolocation.io with Apache HTTP Server

MaxMind's [`mod_maxminddb`](https://github.com/maxmind/mod_maxminddb) module reads any MMDB file, including IPGeolocation.io databases. It sets environment variables with country, city, ASN and security data (VPN, proxy, Tor, threat score) for each request, from local files, with no API calls. Apache can then pass the values to your application as headers, write them to the access log, or block requests with them.

> [!TIP]
> `mod_maxminddb` takes a path into the record for each variable, so it works with IPGeolocation.io's schema as it is. Use IPGeolocation.io's paths, such as `location/country/code2`, not MaxMind's `country/iso_code`. See the [field reference](../mmdb-field-reference/README.md) (written with dots; `mod_maxminddb` uses slashes).

---

## Tested with

| Component | Version |
| --- | --- |
| Apache HTTP Server | 2.4 (Debian 12 `apache2` package) |
| `mod_maxminddb` | 1.3.0, built from source |
| Databases | IPGeolocation.io Location, Security and ASN MMDB files |

## Prerequisites

- Apache HTTP Server 2.4 with `mod_remoteip` and `mod_headers` (both ship with Apache).
- `mod_maxminddb` and `libmaxminddb`. Debian and Ubuntu do not package the module, so build it from source (below).
- IPGeolocation.io MMDB files, for example `db-ip-location.mmdb`, `db-ip-security.mmdb` and `db-ip-asn.mmdb`. Download them from your [IPGeolocation.io account](https://app.ipgeolocation.io), or start with the free sample databases.

## Install mod_maxminddb

On Debian or Ubuntu:

```sh
sudo apt-get install -y apache2 apache2-dev libmaxminddb-dev build-essential curl
curl -fsSL https://github.com/maxmind/mod_maxminddb/releases/download/1.3.0/mod_maxminddb-1.3.0.tar.gz | tar xz
cd mod_maxminddb-1.3.0 && ./configure && sudo make install
echo 'LoadModule maxminddb_module /usr/lib/apache2/modules/mod_maxminddb.so' | sudo tee /etc/apache2/mods-available/maxminddb.load
sudo a2enmod remoteip headers maxminddb
```

## Configuration

Save this as `/etc/apache2/conf-available/ipgeolocation.conf`, enable it with `sudo a2enconf ipgeolocation`, and reload Apache.

```apache
# Take the client address from X-Forwarded-For, trusting only your proxy.
RemoteIPHeader X-Forwarded-For
RemoteIPInternalProxy 127.0.0.1

<IfModule mod_maxminddb.c>
    MaxMindDBEnable On
    MaxMindDBFile LOCATION /usr/local/share/ipgeolocation/db-ip-location.mmdb
    MaxMindDBFile SECURITY /usr/local/share/ipgeolocation/db-ip-security.mmdb
    MaxMindDBFile ASN      /usr/local/share/ipgeolocation/db-ip-asn.mmdb

    MaxMindDBEnv IPGEO_COUNTRY_CODE LOCATION/location/country/code2
    MaxMindDBEnv IPGEO_CITY         LOCATION/location/city/name/en
    MaxMindDBEnv IPGEO_TIME_ZONE    LOCATION/time_zone
    MaxMindDBEnv IPGEO_IS_TOR       SECURITY/is_tor
    MaxMindDBEnv IPGEO_IS_VPN       SECURITY/is_vpn
    MaxMindDBEnv IPGEO_THREAT_SCORE SECURITY/threat_score
    MaxMindDBEnv IPGEO_ASN          ASN/asn/as_number
</IfModule>

# Pass the values to the application. Remove client-supplied copies first.
RequestHeader unset X-IPGeo-Country-Code
RequestHeader set X-IPGeo-Country-Code "%{IPGEO_COUNTRY_CODE}e" env=IPGEO_COUNTRY_CODE

# Block Tor exit nodes and threat scores above 80.
<Location "/">
    <RequireAll>
        Require all granted
        Require not expr "%{ENV:IPGEO_IS_TOR} == 'true'"
        Require not expr "-n %{ENV:IPGEO_THREAT_SCORE} && %{ENV:IPGEO_THREAT_SCORE} -gt 80"
    </RequireAll>
</Location>
```

What it does:

- `RemoteIPHeader` and `RemoteIPInternalProxy` make Apache use the real client address when it sits behind a load balancer or CDN. List only your own proxies, or clients can choose their address. Leave both lines out if clients connect to Apache directly.
- Each `MaxMindDBEnv` line sets a variable from a path in a database. The paths use slashes.
- `RequestHeader` sends a value to the application behind Apache (for example with `ProxyPass`). Add one line per header you need.
- Security flags are the strings `"true"` and `"false"`, so the block rule compares with `'true'`. Addresses without a security record are not blocked.

You can also log the values, for example by adding `%{IPGEO_COUNTRY_CODE}e` to a `LogFormat`.

## Check it

To see the values while testing, temporarily echo them back as response headers:

```apache
Header set X-Demo-Country "%{IPGEO_COUNTRY_CODE}e" env=IPGEO_COUNTRY_CODE
Header set X-Demo-City "%{IPGEO_CITY}e" env=IPGEO_CITY
```

Then send requests from the local proxy address with different client addresses:

```sh
curl -s -o /dev/null -D - -H 'X-Forwarded-For: <address in your database>' http://localhost/
```

On a test database the results were:

| Client address | Result |
| --- | --- |
| A Pakistan address with no security record | `200`, `X-Demo-Country: PK`, `X-Demo-City: Lahore` |
| A Tor exit node | `403 Forbidden` |
| An address with threat score 85 | `403 Forbidden` |
| An IPv6 address in Japan | `200`, `X-Demo-Country: JP` |

Remove the `X-Demo-*` lines afterwards.

## Notes

- **Other databases.** The ISP database is flat: use `ISP/country/code2`, `ISP/isp` and `ISP/asn`. See the [field reference](../mmdb-field-reference/README.md).
- **Languages.** Replace `/en` in a name path with `/de`, `/fr`, `/ja` or another supported language code.
- **Updates.** Replace each file atomically (download to a temporary file in the same directory, then rename it), then run `sudo apachectl graceful` so Apache reopens the databases.

## Troubleshooting

**No variables are set.** Check that `mod_maxminddb` is loaded (`apachectl -M | grep maxminddb`), that the paths in `MaxMindDBFile` are readable by the Apache user, and that the address is in the database: `mmdbio read --db db-ip-location.mmdb --ip <address>`.

**Every request gets your load balancer's location.** `mod_remoteip` is not configured, or `RemoteIPInternalProxy` (or `RemoteIPTrustedProxy`) does not list the load balancer.

## Related

- [Field reference](../mmdb-field-reference/README.md)
- [Nginx module](https://github.com/IPGeolocation/ngx_http_ipgeolocation_module), if you use Nginx instead of Apache
- [IPGeolocation.io database documentation](https://ipgeolocation.io/documentation/databases.html)
