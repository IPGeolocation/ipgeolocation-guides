# IP Geolocation API JavaScript Client Side Plugin

## Overview
This document provides a comprehensive guide for setting up and using the IP Geolocation API client-side JavaScript plugin. It includes detailed descriptions of configuration options, example usage, and error handling. The plugin lets you use two APIs: the [IP Geolocation API](https://ipgeolocation.io/documentation/ip-location-api.html) and the [IP Security API](https://ipgeolocation.io/documentation/ip-security-api.html), so you can add geolocation and IP threat detection features to your website or app.

This plugin helps you get accurate geolocation details for any IPv4 or IPv6 address. It provides important information such as country, state, city, ZIP code, latitude, longitude, accuracy radius, confidence level, DMA code, connection type, ISP/company information, and ASN data. These details are useful for showing location-based content or understanding where your users are coming from. You can also get abuse contact information, which is helpful for reporting suspicious or malicious activity.

Along with location data, the plugin also includes security lookup capability if you have a paid plan subscription. It can detect VPNs, proxy usage, residential proxies, Tor connections, relay networks, anonymous activity, known attackers, bots, spam sources, and cloud hosting providers, and also provides a threat score to assess risk.

The plugin further offers extra features like timezone details, currency information, and device data parsed from the User-Agent string. These details can help you customize content, set correct time zones, or display prices in the user's local currency.

## Client Side Plugin Purpose

With our **Free plan**, you get basic location details like country, state, city, ZIP code, currency, basic ASN, and timezone. **Paid plans** give progressively more information which includes network details (ASN type, domain, RIR, company), user-agent parsing, hostname resolution, security threat detection, abuse contact data, geo accuracy fields, and DMA codes.

> [!IMPORTANT]
> For a side-by-side comparison of features in each plan, please see the table on our [Pricing page](https://ipgeolocation.io/pricing.html).

This plugin allows developers to integrate and utilize IP geolocation data within their web projects. Developers can use this plugin for various purposes, such as:

1. **Geographic Redirection:** Redirect visitors to region-specific site versions to enhance user experience and compliance with local regulations.
2. **Analytics Enhancement:** Use geolocation data to analyze web traffic and user behavior, enhancing site and content strategies.
3. **Marketing Geo-targeting & Region Segmentation:** Personalize content, offers, or marketing campaigns based on the visitor's region, DMA code, or currency for higher engagement and conversion.
4. **Form Automation:** Pre-fill forms with country, state, city, zip code, and currency information, speeding up the checkout and registration processes.
5. **Digital Rights Management:** Restrict or grant access to content based on the user's location to comply with legal restrictions.
6. **ISP & ASN Attribution:** Determine the internet service provider and ASN ownership behind an IP address for auditing, routing analysis, and attribution.
7. **Suspicious Traffic Detection & Filtering:** Detect and mitigate malicious activities by identifying traffic from suspicious locations or those using VPNs, proxies, Tor, relays, and known attackers.
8. **Abuse Contact Identification:** Identify abusive or malicious IPs by retrieving their registered abuse contacts, enabling automated reporting and proactive threat response.
9. **Access Control:** Block or restrict access from specific countries to comply with export controls or to reduce fraud and abuse.
10. **Hostname Resolution (DB, Live, Fallback):** Retrieve the hostname of an IP using a fast internal database, live DNS lookup, or a fallback of both. Useful for reverse DNS auditing and attribution.
11. **Client Environment Profiling:** Gain insights into the client's device, browser, and operating system to support adaptive UI/UX and environment-specific optimizations.
12. **Timezone Synchronization:** Display times and dates in the local timezone for events, posts, or deadlines, ensuring clarity and consistency for global users.
13. **Language and Currency Customization:** Automatically display content in the user's language and currency based on their geographic location.

> [!TIP]
> Our JavaScript plugin lets you easily integrate location and security intelligence directly into your website or web application. It enables you to deliver dynamic, personalized, and secure user experiences by providing real-time features such as geolocation details, security threat detection, and more, without requiring complex backend setup.

## Table of Contents
1. [Requirements](#requirements)
2. [How to Get Your API Key](#how-to-get-your-api-key)
3. [Configurations](#configurations)
   - [IPGeolocationAPI: Authentication & Input](#ipgeolocationapi-authentication--input)
   - [IPGeolocationAPI: Basic Response Customization](#ipgeolocationapi-basic-response-customization)
   - [IPGeolocationAPI: include Options and Plan Availability](#ipgeolocationapi-include-options-and-plan-availability)
   - [IPGeolocationAPI: Storage Option](#ipgeolocationapi-storage-option)
   - [IPSecurityAPI: Authentication & Input](#ipsecurityapi-authentication--input)
   - [IPSecurityAPI: Response Customization](#ipsecurityapi-response-customization)
   - [IPSecurityAPI: Storage Option](#ipsecurityapi-storage-option)
4. [Setup Plugin](#setup-plugin)
5. [API Key Usage Example](#api-key-usage-example)
   - [No API Key](#no-api-key-request-origin-is-whitelisted)
   - [With API Key](#with-api-key)
6. [Free Plan Examples](#free-plan-examples)
   - [Get Default Geolocation Fields](#get-default-geolocation-fields)
   - [Filtering Specific Fields from the Response](#filtering-specific-fields-from-the-response)
7. [Paid Plan Examples](#paid-plan-examples)
   - [Geolocation with Default Fields](#geolocation-with-default-fields)
   - [Retrieving Geolocation Data in Multiple Languages](#retrieving-geolocation-data-in-multiple-languages)
   - [Include Hostname and User-Agent](#include-hostname-and-user-agent)
   - [Include Geo Accuracy Fields](#include-geo-accuracy-fields)
   - [Include DMA Code](#include-dma-code)
   - [Include Abuse Contact Information](#include-abuse-contact-information)
   - [Include Security Information](#include-security-information)
   - [Include All Optional Modules](#include-all-optional-modules)
8. [IPSecurityAPI Examples](#ipsecurityapi-examples)
   - [Basic Security Lookup](#basic-security-lookup)
   - [Return Only Specific Security Fields](#return-only-specific-security-fields)
   - [Exclude Specific Security Fields](#exclude-specific-security-fields)
9. [Error Handling](#error-handling)

---

## Requirements
- Active internet connection
- **API Key**: Sign up at [IPGeolocation.io](https://ipgeolocation.io)

## How to Get Your API Key

1. **Sign up** here: [https://app.ipgeolocation.io/signup](https://app.ipgeolocation.io/signup)
2. **(Optional)** Verify your email if you signed up using an email address.
3. **Log in** to your account: [https://app.ipgeolocation.io/login](https://app.ipgeolocation.io/login)
4. After logging in, navigate to your **Dashboard** to find your API key: [https://app.ipgeolocation.io/dashboard](https://app.ipgeolocation.io/dashboard)

---

## Configurations

### IPGeolocationAPI: Authentication & Input
| Option      | Type   | Description                                                                                  |
|-------------|--------|----------------------------------------------------------------------------------------------|
| `apiKey`    | string | Your API key. Required unless request origin is whitelisted (available only on paid plans).  |
| `ipAddress` | string | IPv4, IPv6, or domain name to query. If omitted, the plugin auto-detects the user's IP.      |

### IPGeolocationAPI: Basic Response Customization
| Option     | Type   | Description                                                                           |
|------------|--------|---------------------------------------------------------------------------------------|
| `fields`   | string | Comma-separated list of fields to include in the response. Available on all plans.    |
| `excludes` | string | Comma-separated list of fields to exclude from the response. Available on all plans.  |
| `lang`     | string | Language code for translated fields (default: `en`). Requires a paid plan. Supported values: `de`, `ru`, `ja`, `fr`, `cn`, `es`, `cs`, `it`, `ko`, `fa`, `pt`. |

### IPGeolocationAPI: include Options and Plan Availability
| Option                        | Type    | Description                                                          | Free | Paid |
|-------------------------------|---------|----------------------------------------------------------------------|:----:|:----:|
| `includeHostname`             | boolean | Include hostname from IPGeolocation's IP-Hostname database.          |  ✖   |  ✔   |
| `includeLiveHostname`         | boolean | Use live DNS sources to resolve hostname.                            |  ✖   |  ✔   |
| `includeHostnameFallbackLive` | boolean | Try database first, fall back to live sources if not found.          |  ✖   |  ✔   |
| `includeUserAgent`            | boolean | Include parsed client browser and device info.                       |  ✖   |  ✔   |
| `includeGeoAccuracy`          | boolean | Add `accuracy_radius`, `confidence`, and `locality` to `location`.  |  ✖   |  ✔   |
| `includeDmaCode`              | boolean | Add `dma_code` to the `location` object (US IPs only).              |  ✖   |  ✔   |
| `includeAbuse`                | boolean | Include IP abuse contact information.                                |  ✖   |  ✔   |
| `includeSecurity`             | boolean | Include VPN, proxy, Tor, relay, and threat detection.                |  ✖   |  ✔   |
| `includeAll`                  | boolean | Include all optional modules in a single request (`include=*`).      |  ✖   |  ✔   |

> [!NOTE]
> No `include` options are available on the Free plan. Timezone data (`time_zone` object) is included by default in the response on paid plans without requiring a separate include option.

> [!NOTE]
> Only one of `includeHostname`, `includeLiveHostname`, or `includeHostnameFallbackLive` should be set to `true` at a time. If multiple are passed, the API applies precedence: `liveHostname` → `hostname` → `hostnameFallbackLive`.

### IPGeolocationAPI: Storage Option
| Option                 | Type    | Description                                                                                                                                                                  |
|------------------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `saveToSessionStorage` | boolean | If enabled, stores the geolocation response in `sessionStorage` to avoid repeated API calls during the same browser session, improving performance and reducing API usage. |

---

### IPSecurityAPI: Authentication & Input
| Option      | Type   | Description                                                                                 |
|-------------|--------|---------------------------------------------------------------------------------------------|
| `apiKey`    | string | Your API key. Required unless request origin is whitelisted (available only on paid plans). |
| `ipAddress` | string | IPv4 or IPv6 address to query. If omitted, the plugin auto-detects the user's IP.           |

### IPSecurityAPI: Response Customization
| Option     | Type   | Description                                                                                    |
|------------|--------|------------------------------------------------------------------------------------------------|
| `fields`   | string | Comma-separated list of `security` object fields to include (e.g. `security.threat_score`).  |
| `excludes` | string | Comma-separated list of `security` object fields to exclude (e.g. `security.is_tor`).        |

### IPSecurityAPI: Storage Option
| Option                 | Type    | Description                                                                                                                                                               |
|------------------------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `saveToSessionStorage` | boolean | If enabled, stores the security response in `sessionStorage` to avoid repeated API calls during the same browser session, improving performance and reducing API usage. |

> [!NOTE]
> The `IPSecurityAPI` class calls the dedicated `/v3/security` endpoint and returns only the `security` object. It does not support geolocation, timezone, user-agent, or other fields. Each successful security lookup costs **2 API credits**.For more details about API credits and usage limits, please visit our [Credits usage page](https://ipgeolocation.io/documentation/credits-usage.html). To combine security data with geolocation in one response, use `IPGeolocationAPI` with `includeSecurity: true`.

---

## Setup Plugin
To access this service, add the following JavaScript tag (usually within the `<head>` block of your pages).

```html
<script src="https://static.ipgeolocation.io/ipgeolocation-api-plugin.v3.0.0.js"></script>
```

---

## API Key Usage Example
Demonstrates how to use the plugin with and without an API key. The API key is optional if the request originates from a whitelisted domain (available only on paid plans).

### No API Key (Request Origin is Whitelisted)
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI();

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```

### With API Key
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```

---

## Free Plan Examples

### Get Default Geolocation Fields
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "8.8.8.8",
  "location": {
    "continent_code": "NA",
    "continent_name": "North America",
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "country_capital": "Washington, D.C.",
    "state_prov": "California",
    "state_code": "US-CA",
    "district": "Santa Clara",
    "city": "Mountain View",
    "zipcode": "94043-1351",
    "latitude": "37.42240",
    "longitude": "-122.08421",
    "is_eu": false,
    "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
    "geoname_id": "6301403",
    "country_emoji": "🇺🇸"
  },
  "country_metadata": {
    "calling_code": "+1",
    "tld": ".us",
    "languages": ["en-US", "es-US", "haw", "fr"]
  },
  "currency": {
    "code": "USD",
    "name": "US Dollar",
    "symbol": "$"
  },
  "asn": {
    "as_number": "AS15169",
    "organization": "Google LLC",
    "country": "US"
  },
  "time_zone": {
    "name": "America/Los_Angeles",
    "offset": -8,
    "offset_with_dst": -7,
    "current_time": "2026-03-27 10:00:00.000-0700",
    "current_time_unix": 1743087600.000,
    "current_tz_abbreviation": "PDT",
    "current_tz_full_name": "Pacific Daylight Time",
    "standard_tz_abbreviation": "PST",
    "standard_tz_full_name": "Pacific Standard Time",
    "is_dst": true,
    "dst_savings": 1,
    "dst_exists": true,
    "dst_tz_abbreviation": "PDT",
    "dst_tz_full_name": "Pacific Daylight Time",
    "dst_start": {
      "utc_time": "2026-03-08 TIME 10:00",
      "duration": "+1.00H",
      "gap": true,
      "date_time_after": "2026-03-08 TIME 03:00",
      "date_time_before": "2026-03-08 TIME 02:00",
      "overlap": false
    },
    "dst_end": {
      "utc_time": "2026-11-01 TIME 09:00",
      "duration": "-1.00H",
      "gap": false,
      "date_time_after": "2026-11-01 TIME 01:00",
      "date_time_before": "2026-11-01 TIME 02:00",
      "overlap": true
    }
  }
}
```

### Filtering Specific Fields from the Response
Use `fields` and `excludes` together to get the `location` object while omitting continent fields.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location",
            excludes: "location.continent_code,location.continent_name",
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "8.8.8.8",
  "location": {
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "country_capital": "Washington, D.C.",
    "state_prov": "California",
    "state_code": "US-CA",
    "district": "Santa Clara",
    "city": "Mountain View",
    "zipcode": "94043-1351",
    "latitude": "37.42240",
    "longitude": "-122.08421",
    "is_eu": false,
    "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
    "geoname_id": "6301403",
    "country_emoji": "🇺🇸"
  }
}
```

---

## Paid Plan Examples

### Geolocation with Default Fields
On a paid plan, the default response includes additional objects such as `network`, `asn`, `company`, and richer `asn` fields.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "8.8.8.8",
  "location": {
    "continent_code": "NA",
    "continent_name": "North America",
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "country_capital": "Washington, D.C.",
    "state_prov": "California",
    "state_code": "US-CA",
    "district": "Santa Clara",
    "city": "Mountain View",
    "zipcode": "94043-1351",
    "latitude": "37.42240",
    "longitude": "-122.08421",
    "is_eu": false,
    "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
    "geoname_id": "6301403",
    "country_emoji": "🇺🇸"
  },
  "country_metadata": {
    "calling_code": "+1",
    "tld": ".us",
    "languages": ["en-US", "es-US", "haw", "fr"]
  },
  "network": {
    "connection_type": "wired",
    "route": "8.8.8.0/24",
    "is_anycast": true
  },
  "currency": {
    "code": "USD",
    "name": "US Dollar",
    "symbol": "$"
  },
  "asn": {
    "as_number": "AS15169",
    "organization": "Google LLC",
    "country": "US",
    "type": "HOSTING",
    "domain": "about.google",
    "date_allocated": "2000-03-30",
    "rir": "ARIN"
  },
  "company": {
    "name": "Google LLC",
    "type": "HOSTING",
    "domain": "google.com"
  },
  "time_zone": {
    "name": "America/Los_Angeles",
    "offset": -8,
    "offset_with_dst": -7,
    "current_time": "2026-03-27 10:00:00.000-0700",
    "current_time_unix": 1743087600.000,
    "current_tz_abbreviation": "PDT",
    "current_tz_full_name": "Pacific Daylight Time",
    "standard_tz_abbreviation": "PST",
    "standard_tz_full_name": "Pacific Standard Time",
    "is_dst": true,
    "dst_savings": 1,
    "dst_exists": true,
    "dst_tz_abbreviation": "PDT",
    "dst_tz_full_name": "Pacific Daylight Time",
    "dst_start": {
      "utc_time": "2026-03-08 TIME 10:00",
      "duration": "+1.00H",
      "gap": true,
      "date_time_after": "2026-03-08 TIME 03:00",
      "date_time_before": "2026-03-08 TIME 02:00",
      "overlap": false
    },
    "dst_end": {
      "utc_time": "2026-11-01 TIME 09:00",
      "duration": "-1.00H",
      "gap": false,
      "date_time_after": "2026-11-01 TIME 01:00",
      "date_time_before": "2026-11-01 TIME 02:00",
      "overlap": true
    }
  }
}
```

### Retrieving Geolocation Data in Multiple Languages
Here is an example to get the geolocation data for the client's IP address in Russian (`ru`):

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            lang: "ru",
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```

> [!TIP]
> Check out all available language options here: [Response in multiple languages](https://ipgeolocation.io/documentation/ip-location-api.html#response-in-multiple-languages).

### Include Hostname and User-Agent
`includeHostname`, `includeLiveHostname`, and `includeHostnameFallbackLive` are available on paid plans. Use only one at a time.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location.country_name,location.country_capital",
            includeHostname: true,
            includeUserAgent: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "4.5.6.7",
  "hostname": "4.5.6.7",
  "location": {
    "country_name": "United States",
    "country_capital": "Washington, D.C."
  },
  "user_agent": {
    "user_agent_string": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",
    "name": "Chrome",
    "type": "Browser",
    "version": "143.0.0.0",
    "version_major": "143",
    "device": {
      "name": "Linux Desktop",
      "type": "Desktop",
      "brand": "Unknown",
      "cpu": "Intel x86_64"
    },
    "engine": {
      "name": "Blink",
      "type": "Browser",
      "version": "143.0.0.0",
      "version_major": "143"
    },
    "operating_system": {
      "name": "Linux",
      "type": "Desktop",
      "version": "??",
      "version_major": "??",
      "build": "??"
    }
  }
}
```

> [!NOTE]
> The three hostname options behave as follows:
> - `includeHostname`: Returns hostname from IPGeolocation's local IP-Hostname database. Fastest option.
> - `includeLiveHostname`: Performs a live DNS lookup. More accurate but adds latency.
> - `includeHostnameFallbackLive`: Checks the local database first; performs a live lookup only if no hostname is found. Best balance of speed and accuracy.
>
> If a hostname cannot be resolved, the `hostname` field returns the queried IP address.

### Include Geo Accuracy Fields
Pass `includeGeoAccuracy: true` to add `accuracy_radius`, `confidence`, and `locality` to the `location` object.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location",
            includeGeoAccuracy: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response (relevant `location` fields shown):
```json
{
  "ip": "91.128.103.196",
  "location": {
    "continent_code": "EU",
    "continent_name": "Europe",
    "country_code2": "SE",
    "country_code3": "SWE",
    "country_name": "Sweden",
    "country_name_official": "Kingdom of Sweden",
    "country_capital": "Stockholm",
    "state_prov": "Stockholms län",
    "state_code": "SE-AB",
    "district": "Stockholm",
    "city": "Stockholm",
    "locality": "Stockholm",
    "accuracy_radius": "9.148",
    "confidence": "high",
    "zipcode": "164 40",
    "latitude": "59.40510",
    "longitude": "17.95510",
    "is_eu": true,
    "country_flag": "https://ipgeolocation.io/static/flags/se_64.png",
    "geoname_id": "9972319",
    "country_emoji": "🇸🇪"
  }
}
```

### Include DMA Code
The DMA (Designated Market Area) code is a US-only field used for marketing and regional targeting. Pass `includeDmaCode: true` to include it.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location",
            includeDmaCode: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response (US IP):
```json
{
  "ip": "107.161.145.197",
  "location": {
    "continent_code": "NA",
    "continent_name": "North America",
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "country_capital": "Washington, D.C.",
    "state_prov": "Pennsylvania",
    "state_code": "US-PA",
    "district": "Philadelphia County",
    "city": "Philadelphia",
    "dma_code": "504",
    "zipcode": "19102",
    "latitude": "39.95258",
    "longitude": "-75.16522",
    "is_eu": false,
    "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
    "geoname_id": "9849057",
    "country_emoji": "🇺🇸"
  }
}
```

> [!NOTE]
> `dma_code` will be non-empty only for IPs located in the United States.

### Include Abuse Contact Information
Pass `includeAbuse: true` to add the `abuse` object to the response.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location.city,location.country_name",
            includeAbuse: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "91.128.103.196",
  "location": {
    "city": "Stockholm",
    "country_name": "Sweden"
  },
  "abuse": {
    "route": "91.128.0.0/14",
    "country": "SE",
    "name": "Swipnet Staff",
    "organization": "",
    "kind": "group",
    "address": "Tele2 AB/Swedish IP Network\nIP Registry\nTorshamnsgatan 17 164 40 Kista SWEDEN",
    "emails": ["abuse@tele2.com"],
    "phone_numbers": ["+46 8 5626 42 10"]
  }
}
```

> [!NOTE]
> Adding `includeAbuse` costs **1 additional credit** on top of the base lookup credit, for a total of **2 credits** per request.

### Include Security Information
Pass `includeSecurity: true` to add the `security` object to the geolocation response.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location.city,location.country_name",
            includeSecurity: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "2.56.188.34",
  "location": {
    "city": "Dallas",
    "country_name": "United States"
  },
  "security": {
    "threat_score": 80,
    "is_tor": false,
    "is_proxy": true,
    "proxy_provider_names": ["Zyte Proxy"],
    "proxy_confidence_score": 80,
    "proxy_last_seen": "2025-12-12",
    "is_residential_proxy": true,
    "is_vpn": true,
    "vpn_provider_names": ["Nord VPN"],
    "vpn_confidence_score": 80,
    "vpn_last_seen": "2026-01-19",
    "is_relay": false,
    "relay_provider_name": "",
    "is_anonymous": true,
    "is_known_attacker": true,
    "is_bot": false,
    "is_spam": false,
    "is_cloud_provider": true,
    "cloud_provider_name": "Packethub S.A."
  }
}
```

> [!NOTE]
> Adding `includeSecurity` costs **2 additional credits** on top of the base lookup credit, for a total of **3 credits** per request.

### Include All Optional Modules
Pass `includeAll: true` to request every available optional module (`security`, `abuse`, `user_agent`, `hostname`, `geo_accuracy`, `dma_code`) in a single call. Equivalent to passing `include=*` to the API.

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            includeAll: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```

> [!NOTE]
> Using `includeAll` costs **4 API credits** per request: 1 for the base lookup, 2 for the `security` module, and 1 for the `abuse` module.

> [!NOTE]
> All features available on the Free plan are also included on paid plans.

---

## IPSecurityAPI Examples
The `IPSecurityAPI` class calls the dedicated `/v3/security` endpoint and returns only IP threat signals. Use it when you need security data alone. To combine security data with geolocation, use `IPGeolocationAPI` with `includeSecurity: true` instead.

### Basic Security Lookup

```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            ipAddress: "2.56.188.34",
        });

        const resp = await ipSecAPI.getSecurity();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "2.56.188.34",
  "security": {
    "threat_score": 80,
    "is_tor": false,
    "is_proxy": true,
    "proxy_provider_names": ["Zyte Proxy"],
    "proxy_confidence_score": 80,
    "proxy_last_seen": "2025-12-12",
    "is_residential_proxy": true,
    "is_vpn": true,
    "vpn_provider_names": ["Nord VPN"],
    "vpn_confidence_score": 80,
    "vpn_last_seen": "2026-01-19",
    "is_relay": false,
    "relay_provider_name": "",
    "is_anonymous": true,
    "is_known_attacker": true,
    "is_bot": false,
    "is_spam": false,
    "is_cloud_provider": true,
    "cloud_provider_name": "Packethub S.A."
  }
}
```

### Return Only Specific Security Fields
Use `fields` to limit the response to only the security properties you need.

```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            ipAddress: "2.56.188.34",
            fields: "security.threat_score,security.is_vpn,security.is_proxy",
        });

        const resp = await ipSecAPI.getSecurity();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "2.56.188.34",
  "security": {
    "threat_score": 80,
    "is_proxy": true,
    "is_vpn": true
  }
}
```

### Exclude Specific Security Fields
Use `excludes` to return the full `security` object while omitting fields you don't need.

```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            ipAddress: "2.56.188.34",
            excludes: "security.is_tor,security.is_cloud_provider",
        });

        const resp = await ipSecAPI.getSecurity();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response (`is_tor` and `is_cloud_provider` omitted):
```json
{
  "ip": "2.56.188.34",
  "security": {
    "threat_score": 80,
    "is_proxy": true,
    "proxy_provider_names": ["Zyte Proxy"],
    "proxy_confidence_score": 80,
    "proxy_last_seen": "2025-12-12",
    "is_residential_proxy": true,
    "is_vpn": true,
    "vpn_provider_names": ["Nord VPN"],
    "vpn_confidence_score": 80,
    "vpn_last_seen": "2026-01-19",
    "is_relay": false,
    "relay_provider_name": "",
    "is_anonymous": true,
    "is_known_attacker": true,
    "is_bot": false,
    "is_spam": false,
    "cloud_provider_name": "Packethub S.A."
  }
}
```

---

## Error Handling
Inspect the `error_message` field in the response to detect any errors. A missing `error_message` typically indicates a successful request, while its presence signals an issue. For example, if you are using a free plan and trying to request security information, you will encounter an error as shown below:

```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_API_KEY",
            includeSecurity: true,
        });

        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample error response:
```json
{
  "error_message": "IP-hostname lookup, IP-security lookup and user-agent parsing are not supported on your free subscription. These features are available to all paid subscriptions only.",
  "error_status": 401
}
```


> [!NOTE]
> More information about error codes and messages can be found in the [error codes](https://ipgeolocation.io/documentation/ip-location-api.html#error-codes) section of our documentation. For detailed API documentation and additional features, please refer to the official [IPGeolocation API documentation](https://ipgeolocation.io/documentation.html).
>
> For details specific to the IP Security API, including security checks and threat intelligence fields, please refer to the [IP Security API documentation](https://ipgeolocation.io/documentation/ip-security-api.html).
