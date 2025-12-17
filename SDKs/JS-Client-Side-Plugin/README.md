# IP Geolocation API Javascript Client Side Plugin

## Overview
This document provides a comprehensive guide for setting up and using the IP Geolocation API client side JavaScript SDK. It includes detailed descriptions of configuration options, example usage, and error handling. The SDK lets you use two APIs: the [IP Geolocation API](https://ipgeolocation.io/ip-location-api.html) (Free, Standard or Advance) and the [IP Security API](https://ipgeolocation.io/ip-security-api.html) , so you can add geolocation and IP threat detection features to your website or app.

This plugin helps you get accurate geolocation details for any IPv4 or IPv6 address. It provides important information such as country, state, city, ZIP code, latitude, longitude, accuracy radius, confidence, dma code, connection type, ISP/Company information, and ASN data. These details are useful for showing location-based content or understanding where your users are coming from. You can also get abuse contact information, which is helpful for reporting suspicious or malicious activity.

Along with location data, the plugin also includes security lookups capability if you have a Security plan subscription. It can detect VPNs, proxy usage, Tor connections, anonymous activity, known attackers, bots, spam sources, and cloud hosting providers, and also provides a threat score to assess risk.

The plugin further offers extra features like timezone details, currency information, and device data from the User-Agent string. These details can help you customize content, set correct time zones, or display prices in the user’s local currency.

## Client Side SDK Purpose

With our [Free Plan](https://ipgeolocation.io/ip-location-api.html#documentation-overview), you get basic location details like country, state, city, ZIP code, and currency. The [Standard Plan](https://ipgeolocation.io/ip-location-api.html#documentation-overview) gives more information in addition to geolocation like timezone, network details (ASN and ISP), and user data such as browser and operating system. The [Security Plan](https://ipgeolocation.io/ip-security-api.html#documentation-overview) focuses on online safety and risk detection. It adds advanced security checks like VPN, proxy, and Tor detection, and provides a threat score to help identify suspicious or harmful activity. Our [Advance Plan](https://ipgeolocation.io/ip-location-api.html#documentation-overview) gives you everything, plus extra details like abuse information, privacy detection, connection type, and DMA codes to help with fraud detection, better marketing, and following local rules.

> [!IMPORTANT]
> For a side-by-side comparison of features in each plan, please see the table on our [Billing page](https://ipgeolocation.io/pricing.html).

This SDK allows developers to integrate and utilize IP geolocation data within their web projects. Developers can use this SDK for various purposes, such as:

1. **Geographic Redirection:** Redirect visitors to region-specific site versions to enhance user experience and compliance with local regulations.
2. **Analytics Enhancement:** Use geolocation data to analyze web traffic and user behavior, enhancing site and content strategies.
3. **Marketing Geo-targeting & Region Segmentation:** Personalize content, offers, or marketing campaigns based on the visitor’s region, DMA code, or currency for higher engagement and conversion.
4. **Form Automation:** Pre-fill forms with country, state, city, zip code, and currency information, speeding up the checkout and registration processes.
5. **Digital Rights Management:** Restrict or grant access to content based on the user's location to comply with legal restrictions.
6. **ISP & ASN Attribution:** Determine the internet service provider and ASN ownership behind an IP address for auditing, routing analysis, and attribution.
7. **Suspicious Traffic Detection & Filtering:** Detect and mitigate malicious activities by identifying traffic from suspicious locations or those using VPNs, proxies, tors, and known attackers.
8. **Abuse Contact Identification:** Identify abusive or malicious IPs by retrieving their registered abuse contacts, enabling automated reporting and proactive threat response.
9. **Access Control:** Block or restrict access from specific countries to comply with export controls or to reduce fraud and abuse.
10. **Hostname Resolution (DB, Live, Fallback):** Retrieve the hostname of an IP using a fast internal database, live DNS lookup, or a fallback of both. Useful for reverse DNS auditing and attribution.
11. **Client Environment Profiling:** Gain insights into the client's device, browser, and operating system to support adaptive UI/UX and environment-specific optimizations.
12. **Timezone Synchronization:** Display times and dates in the local timezone for events, posts, or deadlines, ensuring clarity and consistency for global users.
13. **Language and Currency Customization:** Automatically display content in the user's language and currency based on their geographic location.

> [!TIP]
> Our JavaScript plugin let you easily integrate location and security intelligence directly into your website or web application. It enables you to deliver dynamic, personalized, and secure user experiences by providing real-time features such as geolocation details, security threat detection, and more without requiring complex backend setup.

## Table of Contents
1. [Requirements](#requirements)
2. [How to Get Your API Key](#how-to-get-your-api-key)
3. [Configurations](#configurations)
   - [Authentication & Input](#authentication--input)
   - [Basic Response Customization](#basic-response-customization-free--paid)
   - [include fields availability](#include-fields-availability)
   - [Storage Option](#storage-option)
4. [Setup Plugin](#storage-option)
5. [API Key Usage Example](#api-key-usage-example)
   - [No API Key](#no-api-key-request-origin-is-whitelisted)
   - [With API Key](#with-api-key)
6. [Free Plan Examples](#developer-free-plan-examples)
   - [Get IPGeo Default fields](#get-ipgeo-default-fields)
   - [Filtering Specific Fields from the Response](#filtering-specific-fields-from-the-response)
7. [Standard Plan Examples](#standard-plan-examples)
   - [Geolocation with Default Fields](#geolocation-with-default-fields)
   - [Retrieving Geolocation Data in Multiple Languages](#retrieving-geolocation-data-in-multiple-languages)
   - [Include Hostname, Timezone, and User-Agent](#include-hostname-timezone-and-user-agent)
8. [Advanced Plan Example](#advanced-plan-example)
   - [Include DMA, Abuse, and Security](#include-dma-abuse-and-security)
9. [Security Plan Examples](#security-plan-examples)
   - [Combine IP Security with Geolocation](#combine-ip-security-with-geolocation)
   - [Combine IP Security with all fields](#combine-ip-security-with-all-fields)
   - [Field Filtering for security fields](#request-with-field-filtering-for-security-fields)
10. [Error Handling](#error-handling)

## Requirements
- Active internet connection
- **API Key**: Sign up [IPGeolocation.io](https://ipgeolocation.io)

## How to Get Your API Key

1. **Sign up** here: [https://app.ipgeolocation.io/signup](https://app.ipgeolocation.io/signup)
2. **(optional)** Verify your email, if you signed up using email.
3. **Log in** to your account: [https://app.ipgeolocation.io/login](https://app.ipgeolocation.io/login)
4. After logging in, navigate to your **Dashboard** to find your API key: [https://app.ipgeolocation.io/dashboard](https://app.ipgeolocation.io/dashboard)

## Configurations
### Authentication & Input
| Option    | Type   | Description                                                                                 |
|-----------|--------|---------------------------------------------------------------------------------------------|
| apiKey    | string | Your API key. Required unless request origin is whitelisted (available only in Paid Plans). |
| ipAddress | string | IPv4, IPv6, or domain name to query. If omitted, the SDK auto-detects the user’s IP.        |

### Basic Response Customization (Free & Paid)
| Option   | Type   | Description                                                                          |
|----------|--------|--------------------------------------------------------------------------------------|
| fields   | string | Comma-separated list of fields to include in the response. Available in all plans.   |
| excludes | string | Comma-separated list of fields to exclude from the response. Available in all plans. |
| lang     | string | Language code for translated fields (default: en). Requires Standard plan or higher. |

### include fields availability
| Option                      | Type    | Description                                                   | Free | Standard | Security | Advance |
|-----------------------------|---------|---------------------------------------------------------------|:----:|:--------:|:--------:|:-------:|
| includeHostname             | boolean | Include hostname from IPGeolocation database.                 |  ✖   |    ✔     |    ✔     |    ✔    |
| includeLiveHostname         | boolean | Use live sources to resolve hostname.                         |  ✖   |    ✔     |    ✔     |    ✔    |
| includeHostnameFallbackLive | boolean | Try DB first, fallback to live sources.                       |  ✖   |    ✔     |    ✔     |    ✔    |
| includeTimezone             | boolean | Include timezone information.                                 |  ✖   |    ✔     |    ✔     |    ✔    |
| includeUserAgent            | boolean | Include client browser and device info.                       |  ✖   |    ✔     |    ✔     |    ✔    |
| includeLocation             | boolean | Include geolocation details such as city, state, and country. |  ✖   |    ✖     |    ✔     |    ✖    |
| includeNetwork              | boolean | Include network-related data like ASN and ISP.                |  ✖   |    ✖     |    ✔     |    ✖    |
| includeCurrency             | boolean | Include currency information based on geolocation.            |  ✖   |    ✖     |    ✔     |    ✖    |
| includeCountryMetadata      | boolean | Include extra metadata about the user's country.              |  ✖   |    ✖     |    ✔     |    ✖    |
| includeSecurity             | boolean | Include VPN/proxy/Tor detection.                              |  ✖   |    ✖     |    ✖     |    ✔    |
| includeAbuse                | boolean | Include IP abuse contact information.                         |  ✖   |    ✖     |    ✖     |    ✔    |
| includeDMA                  | boolean | Include Designated Market Area (DMA) code.                    |  ✖   |    ✖     |    ✖     |    ✔    |

> [!NOTE]
> No include field available in the Free plan.

### Storage Option
| Option               | Type    | Description                                                                                                                                                                 |
|----------------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| saveToSessionStorage | boolean | If enabled, stores the geolocation data in sessionStorage to avoid making repeated API calls during the same browser session, improving performance and reducing API usage. |

## Setup Plugin
To access this service, add the following JavaScript call (usually within the head block of your pages).
```html
<script src="https://static.ipgeolocation.io/ipgeolocation-api-plugin.v2.0.0.js"></script>
```

## API Key Usage Example
Demonstrates how to use the Plugin with and without an API key. API key is optional if the request originates from a whitelisted domain (available only in paid plans).
### No API Key (Request Origin is Whitelisted)
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI();

        // fetch the geolocation information from the API and use it as you want
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

        // fetch the geolocation information from the API and use it as you want
        const resp = await ipGeoAPI.getGeolocation();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```

## Developer (Free) Plan Examples
### Get IPGeo Default fields
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
        });

        // fetch the geolocation information from the API and use it as you want
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
    "languages": [
      "en-US",
      "es-US",
      "haw",
      "fr"
    ]
  },
  "currency": {
    "code": "USD",
    "name": "US Dollar",
    "symbol": "$"
  }
}
```
### Filtering Specific Fields from the Response
Use of 'exclude' and 'fields' to get the location fields and exclude continent information.
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location",
            excludes: "location.continent_code,location.continent_name",
        });

        // fetch the geolocation information from the API and use it as you want
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
    "locality": "Mountain View",
    "accuracy_radius": "5",
    "confidence": "High",
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
## Standard Plan Examples
### Geolocation with Default Fields
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
        });

        // fetch the geolocation information from the API and use it as you want
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
    "languages": [
      "en-US",
      "es-US",
      "haw",
      "fr"
    ]
  },
  "network": {
    "asn": {
      "as_number": "AS15169",
      "organization": "Google LLC",
      "country": "US"
    },
    "company": {
      "name": "Google LLC"
    }
  },
  "currency": {
    "code": "USD",
    "name": "US Dollar",
    "symbol": "$"
  }
}
```
### Retrieving Geolocation Data in Multiple Languages
Here is an example to get the geolocation data for the client IP address in Russian (ru) language:
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            lang: "ru"
        });

        // fetch the geolocation information from the API and use it as you want
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
> Checkout all available languages options here: [https://ipgeolocation.io/ip-location-api.html#response-in-multiple-languages](https://ipgeolocation.io/ip-location-api.html#response-in-multiple-languages).

### Include Hostname, Timezone, and User-Agent
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location.country_name,location.country_capital",
            includeHostname: true,
            includeTimezone: true,
            includeUserAgent: true,
        });

        // fetch the geolocation information from the API and use it as you want
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
  "time_zone": {
    "name": "America/Chicago",
    "offset": -6,
    "offset_with_dst": -5,
    "current_time": "2025-07-11 04:50:39.537-0500",
    "current_time_unix": 1752227439.537,
    "is_dst": true,
    "dst_savings": 1,
    "dst_exists": true,
    "dst_start": {
      "utc_time": "2025-03-09 TIME 08",
      "duration": "+1H",
      "gap": true,
      "date_time_after": "2025-03-09 TIME 03",
      "date_time_before": "2025-03-09 TIME 02",
      "overlap": false
    },
    "dst_end": {
      "utc_time": "2025-11-02 TIME 07",
      "duration": "-1H",
      "gap": false,
      "date_time_after": "2025-11-02 TIME 01",
      "date_time_before": "2025-11-02 TIME 02",
      "overlap": true
    }
  },
  "user_agent": {
    "user_agent_string": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",
    "name": "Chrome",
    "type": "Browser",
    "version": "143",
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
      "version": "143",
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
## Advanced Plan Example
### Include DMA, Abuse, and Security
```html
<script>
    (async () => {
        const ipGeoAPI = new IPGeolocationAPI({
            apiKey: "YOUR_IPGEOLOCATION_API_KEY",
            fields: "location.country_flag,location.country_emoji",
            includeDMA: true,
            includeAbuse: true,
            includeSecurity: true,
        });

        // fetch the geolocation information from the API and use it as you want
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
    "geoname_id": "6301403",
    "accuracy_radius": "",
    "locality": "Mountain View",
    "dma_code": "807"
  },
  "country_metadata": {
    "calling_code": "+1",
    "tld": ".us",
    "languages": [
      "en-US",
      "es-US",
      "haw",
      "fr"
    ]
  },
  "network": {
    "asn": {
      "as_number": "AS15169",
      "organization": "Google LLC",
      "country": "US",
      "asn_name": "GOOGLE",
      "type": "BUSINESS",
      "domain": "about.google",
      "date_allocated": "",
      "allocation_status": "assigned",
      "num_of_ipv4_routes": "984",
      "num_of_ipv6_routes": "104",
      "rir": "ARIN"
    },
    "connection_type": "",
    "company": {
      "name": "Google LLC",
      "type": "Business",
      "domain": "googlellc.com"
    }
  },
  "currency": {
    "code": "USD",
    "name": "US Dollar",
    "symbol": "$"
  },
  "security": {
    "threat_score": 0,
    "is_tor": false,
    "is_proxy": false,
    "proxy_type": "",
    "proxy_provider": "",
    "is_anonymous": false,
    "is_known_attacker": false,
    "is_spam": false,
    "is_bot": false,
    "is_cloud_provider": false,
    "cloud_provider": ""
  },
  "abuse": {
    "route": "8.8.8.0/24",
    "country": "",
    "handle": "ABUSE5250-ARIN",
    "name": "Abuse",
    "organization": "Abuse",
    "role": "abuse",
    "kind": "group",
    "address": "1600 Amphitheatre Parkway\nMountain View\nCA\n94043\nUnited States",
    "emails": [
      "network-abuse@google.com"
    ],
    "phone_numbers": [
      "+1-650-253-0000"
    ]
  }
}
```
> [!NOTE]
> All features available in the Free plan are also included in the Standard and Advanced plans. Similarly, all features of the Standard plan are available in the Advanced plan.

## Security Plan Examples
You can use following parameters to customize the API response according to your requirements.

### Combine IP Security with Geolocation
```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_SECURITY_API_KEY",
            includeLocation: true,
        });

        // fetch the security information from the API and use it as you want
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
  "hostname": "2.56.188.34",
  "security": {
    "threat_score": 80,
    "is_tor": false,
    "is_proxy": true,
    "proxy_type": "VPN",
    "proxy_provider": "Nord VPN",
    "is_anonymous": true,
    "is_known_attacker": true,
    "is_spam": false,
    "is_bot": false,
    "is_cloud_provider": true,
    "cloud_provider": "Packethub S.A."
  },
  "location": {
    "continent_code": "NA",
    "continent_name": "North America",
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "country_capital": "Washington, D.C.",
    "state_prov": "Texas",
    "state_code": "US-TX",
    "district": "Dallas",
    "city": "Dallas",
    "zipcode": "75201",
    "latitude": "32.77822",
    "longitude": "-96.79512",
    "is_eu": false,
    "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
    "geoname_id": "4684902",
    "country_emoji": "🇺🇸"
  }
}
```
### Combine IP Security with all fields
```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_SECURITY_API_KEY",
            includeLocation: true,
            includeNetwork: true,
            includeTimezone: true,
            includeUserAgent: true,
            includeCurrency: true,
            includeCountryMetadata: true,
            includeHostname: true,
        });

        // fetch the security information from the API and use it as you want
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
   "hostname": "2.56.188.34",
   "security": {
      "threat_score": 80,
      "is_tor": false,
      "is_proxy": true,
      "proxy_type": "VPN",
      "proxy_provider": "Nord VPN",
      "is_anonymous": true,
      "is_known_attacker": true,
      "is_spam": false,
      "is_bot": false,
      "is_cloud_provider": true,
      "cloud_provider": "Packethub S.A."
   },
   "location": {
      "continent_code": "NA",
      "continent_name": "North America",
      "country_code2": "US",
      "country_code3": "USA",
      "country_name": "United States",
      "country_name_official": "United States of America",
      "country_capital": "Washington, D.C.",
      "state_prov": "Texas",
      "state_code": "US-TX",
      "district": "Dallas",
      "city": "Dallas",
      "zipcode": "75201",
      "latitude": "32.77822",
      "longitude": "-96.79512",
      "is_eu": false,
      "country_flag": "https://ipgeolocation.io/static/flags/us_64.png",
      "geoname_id": "4684902",
      "country_emoji": "🇺🇸"
   },
   "network": {
      "asn": {
         "as_number": "AS62240",
         "organization": "Clouvider Limited",
         "country": "GB"
      },
      "company": {
         "name": "Packethub S.A."
      }
   },
   "time_zone": {
      "name": "America/Chicago",
      "offset": -6,
      "offset_with_dst": -5,
      "current_time": "2025-07-16 11:00:50.605-0500",
      "current_time_unix": 1752681650.605,
      "is_dst": true,
      "dst_savings": 1,
      "dst_exists": true,
      "dst_start": {
         "utc_time": "2025-03-09 TIME 08",
         "duration": "+1H",
         "gap": true,
         "date_time_after": "2025-03-09 TIME 03",
         "date_time_before": "2025-03-09 TIME 02",
         "overlap": false
      },
      "dst_end": {
         "utc_time": "2025-11-02 TIME 07",
         "duration": "-1H",
         "gap": false,
         "date_time_after": "2025-11-02 TIME 01",
         "date_time_before": "2025-11-02 TIME 02",
         "overlap": true
      }
   },
   "user_agent": {
      "user_agent_string": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",
      "name": "Chrome",
      "type": "Browser",
      "version": "143",
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
         "version": "143",
         "version_major": "143"
      },
      "operating_system": {
         "name": "Linux",
         "type": "Desktop",
         "version": "??",
         "version_major": "??",
         "build": "??"
      }
   },
   "country_metadata": {
      "calling_code": "+1",
      "tld": ".us",
      "languages": [
         "en-US",
         "es-US",
         "haw",
         "fr"
      ]
   },
   "currency": {
      "code": "USD",
      "name": "US Dollar",
      "symbol": "$"
   }
}
```
> [!NOTE]
> You can get all the available fields in standard plan in combination with security data, when subscribed to security plan.

### Request with Field Filtering for security fields
```html
<script>
    (async () => {
        const ipSecAPI = new IPSecurityAPI({
            apiKey: "YOUR_SECURITY_API_KEY",
            includeLocation: true,
            includeNetwork: true,
            fields: "security.threat_score,location.city,network.asn.as_number"
        });

        // fetch the security information from the API and use it as you want
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
    "threat_score": 75
  },
  "location": {
    "city": "Brisbane"
  },
  "network": {
    "asn": {
      "as_number": "AS62240"
    }
  }
}
```

## Error Handling
Inspect the status field in the response to detect any errors. A missing status field typically indicates a successful request, while its presence signals an issue. For example, if you are using developer plan and are trying to query the security information for an ipAddress, you will encounter an error as shown below:
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
  "error_message": "IP-hostname lookup, IP-security lookup and user-agent parsing are not supported on your free subscription. These features are available to all paid subscriptions only",
  "error_status": 401
}
```

> [!NOTE]
> More information about error codes and messages can be found in the [error codes](https://ipgeolocation.io/ip-location-api.html#error-codes) section of our documentation. For detailed API documentation and additional features, Please refer to the official [IPGeolocation API documentation](https://ipgeolocation.io/documentation.html).
> 
> For details specific to the IP Security API, including detailed security checks and threat intelligence, Please refer to the [IP Security API documentation](https://ipgeolocation.io/ip-security-api.html).

