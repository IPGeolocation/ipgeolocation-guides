# IPGeolocation Pabbly Connect Integration

## Overview

The Pabbly Connect IPGeolocation Integration lets you use the full [IPGeolocation.io v3](https://ipgeolocation.io) APIs inside your Pabbly Connect workflows without writing code. You add an IPGeolocation.io action to a workflow, map an IP address or a list of IP addresses, and enriched data flows to the next step.

With the IPGeolocation Pabbly integration you can retrieve:

- Geolocation details for any IPv4 or IPv6 address
- Threat intelligence and security risk signals
- Network ownership and ASN information
- Abuse contact details for an IP address
- Time zone data and time conversion results
- Astronomy information such as sunrise and sunset
- User agent details including browser, operating system, and device

The integration exposes 12 actions grouped into 6 categories. This makes Pabbly Connect IP Geolocation automation useful for lead enrichment, fraud detection, security alerting, CRM enrichment, and geo-based notifications.

| Category | Actions |
| --- | --- |
| Geolocation | Get IP Geolocation, Find Bulk IP Geolocation |
| IP Security | Find IP Security, Find Bulk IP Security |
| Network Intelligence | Lookup ASN, Lookup Abuse Contact |
| Time Services | Lookup Timezone Information, Time Conversion |
| Astronomy | Lookup Astronomy, Lookup Astronomy Timeseries |
| User Agent Parsing | Parse User Agent, Parse Bulk User Agents |

---

## Prerequisites

Before you set up the integration, make sure you have the following:

- An active IPGeolocation.io account. You can [sign up](https://app.ipgeolocation.io/signup) for free.
- An IPGeolocation.io API key from your account dashboard.
- An active [Pabbly Connect](https://connect.pabbly.com/) account.

A free IPGeolocation.io plan is enough to test the integration. Some actions require a paid plan, as described in the [action availability](#action-availability-by-plan) table below.

---

## Create an API Key

The integration authenticates with your IPGeolocation.io API key. To obtain it:

1. Go to [IPGeolocation.io](https://ipgeolocation.io/) and [sign in](https://app.ipgeolocation.io/login). If you do not have an account, [sign up](https://app.ipgeolocation.io/signup) first and verify your email if prompted.
2. Open your [dashboard](https://app.ipgeolocation.io/dashboard).
3. In the API Keys section, copy your API key.

Keep this key private. For more detail on authentication, see the [IPGeolocation.io API Authentication](https://ipgeolocation.io/documentation/api-authentication.html).

![IPGeolocation.io dashboard showing the API Keys section with the copy button.](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/copy-api-key-from-dashboard.png)

---

## Connect IPGeolocation to Pabbly Connect

Follow these steps to add IPGeolocation.io as an action in a Pabbly Connect workflow.

1. Log in to [Pabbly Connect](https://connect.pabbly.com/) and open or create a workflow.
2. Add or select an action step, then open the **Choose App** field under the **Action Setup** tab.
3. Type **IPGeolocation** in the search box and select **IPGeolocation.io** under the **Public Apps** tab.

   ![Pabbly Connect Choose App panel with IPGeolocation.io listed under Public Apps.](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/search-ipgeolocation.png)

4. Open the **App Event** dropdown and select the action you want to run, for example **Get IP Geolocation**.
5. Switch to the **Connections** tab and select **Add New Connection**.
6. Enter a **New Connection Name** to identify this connection.
7. Paste the **API Key** you copied from your IPGeolocation.io dashboard into the **API Key** field.
8. Click **Save** to store the connection.

   ![Pabbly Connect connection window with the IPGeolocation.io API Key field](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/add-connection.png)

Once saved, the connection is available for every IPGeolocation.io action in your Pabbly Connect workflows. Certain APIs, including the IP Security API, ASN API, Abuse Contact API, User Agent API, and all bulk actions, require a paid plan, so make sure the key you enter has the required plan permissions for the action you want to run.

---

## Test the Connection

To confirm the connection works:

1. Select an action such as **Get IP Geolocation** under the **App Event** dropdown.
2. Enter a value in the required field. For **Get IP Geolocation**, enter an IP address such as `8.8.8.8` in the **IP Address** field.
3. Click **Save & Send Test Request**.
4. Review the **Response Received** section. A successful test returns the requested data fields for the IP address.

![Pabbly Connect Response Received section with IPGeolocation.io geolocation data](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/successful-test-response.png)

If the test fails, the response usually indicates the cause. Two common messages are documented below.

An invalid API key returns:

```bash
Provided API key is not valid. Contact technical support for assistance at support@ipgeolocation.io
```

![Pabbly Connect response showing an invalid IPGeolocation.io API key message](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/invalid-api-key.png)

Using a free key for a paid action returns the following message:

```bash
IP Security Lookup is not supported on your current subscription. This feature is available to Paid subscriptions only.
```

![Pabbly Connect response showing a paid subscription requirement for IPGeolocation.io](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/if-free-key-is-used-for-paid-actions.png)

See [Troubleshooting](#troubleshooting) for how to resolve these.

---

## Available Actions in the Pabbly Connect IPGeolocation Integration

The integration provides 12 actions. Each action below lists its purpose, required input, optional inputs, and output, with a link to the official field reference for the returned data.

A few parameters appear across several actions:

- **Fields**: return only the fields you name, using dot notation for nested fields (for example `location.city` or `security.threat_score`) or full object names (for example `security` or `currency`). Available on all plans, including the free plan.
- **Excludes**: remove specific fields or whole objects you do not need. Available on all plans.
- **Include**: add optional data modules to the response that are not returned by default. This parameter is available on paid plans only. This parameter is available only in `Get IP Geolocation`, `Find Bulk IP Geolocation` and `Lookup ASN` actions.
- **Language**: return location text in a supported language. This parameter is available on paid plans only.

**Action availability by plan**

IPGeolocation.io offers a free (Developer) plan and paid plans. The table below shows which actions each plan can run.

| Action | Developer (Free) | Paid |
| --- | --- | --- |
| Get IP Geolocation | Yes | Yes |
| Find Bulk IP Geolocation | No | Yes |
| Find IP Security | No | Yes |
| Find Bulk IP Security | No | Yes |
| Lookup ASN | No | Yes |
| Lookup Abuse Contact | No | Yes |
| Lookup Timezone Information | Yes | Yes |
| Time Conversion | Yes | Yes |
| Lookup Astronomy | Yes | Yes |
| Lookup Astronomy Timeseries | Yes | Yes |
| Parse User Agent | No | Yes |
| Parse Bulk User Agents | No | Yes |

For more information about IPGeolocation.io plans, see the [IPGeolocation.io pricing page](https://ipgeolocation.io/pricing.html).

### Geolocation

**Get IP Geolocation**

Retrieves geolocation, country metadata and currency related information for a single IPv4 or IPv6 address. Paid plans also support resolving domain names and returning additional enrichment data such as network, company, security, abuse, hostname, user agent, and advanced ASN information.

- Required input: IP Address or Domain (paid-only), for example `8.8.8.8`, or `example.com`.
- Optional inputs:
  - **Include** (paid only): add one or more optional modules. Accepted values are `geo_accuracy`, `dma_code`, `user_agent`, `security`, `abuse`, and the hostname options `hostname`, `liveHostname`, and `hostnameFallbackLive`. Use `*` to add every module at once.
  - **Fields**: return only the objects or fields you name.
  - **Excludes**: remove objects or fields you do not need.
  - **Language** (paid only): return location text in English (en), German (de), Russian (ru), Japanese (ja), French (fr), Chinese Simplified (cn), Spanish (es), Czech (cs), Italian (it), Korean (ko), Persian (fa), Portuguese (pt), or Standard Arabic (ar). The default is English.
- Output: the `location`, `country_metadata`, `currency`, `asn`, and `time_zone` objects, plus additional objects on paid plans as described below. See the [IP Geolocation field reference](https://ipgeolocation.io/documentation/ip-location-api.html#reference-to-ipgeolocation-api-response).

What Get IP Geolocation returns by plan:

| Response content | Developer (Free) | Paid |
| --- | --- | --- |
| `ip` | Yes | Yes |
| `location` core fields (country, state, city, zip code, latitude, longitude, flag, and more) | Yes | Yes |
| `country_metadata` (calling code, TLD, languages) | Yes | Yes |
| `currency` (code, name, symbol) | Yes | Yes |
| `asn` basic fields (`as_number`, `organization`, `country`) | Yes | Yes |
| `time_zone` (name, offset, current time, DST details) | Yes | Yes |
| `network` (connection type, route, is anycast) | No | Yes |
| `company` (name, type, domain) | No | Yes |
| `asn` extended fields (`type`, `domain`, `date_allocated`, `rir`) | No | Yes |
| `location` accuracy fields via Include `geo_accuracy` (`locality`, `accuracy_radius`, `confidence`) | No | Yes |
| `location.dma_code` via Include `dma_code` (US only) | No | Yes |
| `hostname` via Include `hostname`, `liveHostname`, or `hostnameFallbackLive` | No | Yes |
| `user_agent` object via Include `user_agent` | No | Yes |
| `security` object via Include `security` | No | Yes |
| `abuse` object via Include `abuse` | No | Yes |
| Response language other than English | No | Yes |

On the free plan, the Include parameter is not available. A free plan request returns the default objects listed above. The `fields` and `excludes` parameters work on all plans.

![Pabbly Connect Get IP Geolocation action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/ipgeo.png)

**Find Bulk IP Geolocation**

Retrieves geolocation and related data for multiple IPv4/IPv6 addresses or Domains in a single request. Available on `paid` plans, with support for the same optional enrichment modules as the single IP Geolocation action.

- Required input: IP Addresses, a list of up to 50,000 IPv4 or IPv6 addresses or Domains, for example `["2.2.2.2","3.3.3.3"]`.
- Optional inputs: **Include** (same module values as Get IP Geolocation), **Fields**, and **Excludes**.
- Output: an array of geolocation objects, one entry per submitted address. Invalid or private addresses return a descriptive `message` entry instead. See the [IP Geolocation field reference](https://ipgeolocation.io/documentation/ip-location-api.html#reference-to-ipgeolocation-api-response).

![Pabbly Connect Find Bulk IP Geolocation action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/bulk-ipgeo.png)

### IP Security

**Find IP Security**

Returns security and threat intelligence for a single IP address. Requires a `paid` plan.

- Required input: IP, for example `2.56.188.34`.
- Optional inputs: **Fields** (for example `security.threat_score`), **Excludes** (for example `security.is_tor`, `security.is_cloud_provider`).
- Output: the `security` object, including `threat_score` (0 to 100), `is_proxy`, `is_residential_proxy`, `is_vpn`, `is_tor`, `is_relay`, `is_bot`, `is_spam`, `is_known_attacker`, `is_anonymous`, and `is_cloud_provider`, plus provider names and confidence and last-seen fields where identifiable. See the [IP Security field reference](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference).

![Pabbly Connect Find IP Security action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/ip-security.png)

**Find Bulk IP Security**

Returns security intelligence for many IP addresses in a single request. Requires a `paid` plan.

- Required input: IPs, a list of up to 50,000 IPv4 or IPv6 addresses, for example `["2.56.188.34", "159.26.99.225"]`.
- Optional inputs: **Fields** and **Excludes**.
- Output: an array of `security` objects, one entry per submitted address. See the [IP Security field reference](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference).

![Pabbly Connect Find Bulk IP Security action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/bulk-ip-security.png)

### Network Intelligence

**Lookup ASN**

Retrieves Autonomous System Number details for an IP address or an ASN number. Requires a `paid` plan.

- Required input: one of IP (for example `8.8.8.8`) or ASN (for example `AS2821` or `2821`). If both are provided, the ASN value takes priority and the response does not include the `ip` field.
- Optional inputs:
  - **Include**: add related network data. Accepted values are `peers`, `downstreams`, `upstreams`, `routes`, and `whois_response`.
  - **Fields**: return only the fields you name, for example `asn.organization,asn.country,asn.downstreams`.
  - **Excludes**: remove fields you do not need, for example `asn.date_allocated,asn.allocation_status`.
- Output: the `asn` object, including `as_number`, `organization`, `country`, `asn_name`, `type`, `domain`, `date_allocated`, `rir`, `allocation_status`, `num_of_ipv4_routes`, and `num_of_ipv6_routes`, plus `peers`, `downstreams`, `upstreams`, `routes`, and `whois_response` when requested through Include. See the [ASN field reference](https://ipgeolocation.io/documentation/asn-api.html#reference-to-asn-api-response).

![Pabbly Connect Lookup ASN action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/asn-api.png)

**Lookup Abuse Contact**

Retrieves abuse contact information for an IP address. Requires a `paid` plan.

- Required input: IP, for example `8.8.8.8`.
- Optional inputs: **Fields** (for example `abuse.emails,abuse.organization`) and **Excludes** (for example `abuse.emails`).
- Output: the `abuse` object, including `route`, `country`, `name`, `organization`, `kind`, `address`, `emails`, and `phone_numbers`. See the [Abuse Contact field reference](https://ipgeolocation.io/documentation/ip-abuse-contact-api.html#reference-to-abuse-contact-api-response).

![Pabbly Connect Lookup Abuse Contact action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/abuse-contact.png)

### Time Services

**Lookup Timezone Information**

Returns time zone and related location/airport information. The action accepts several lookup inputs, so you provide whichever identifies your target.

- Lookup inputs (provide one): Timezone name in IANA format (for example `America/Los_Angeles`), Location as a city or state level address, Latitude and Longitude together, IP address, IATA airport code, ICAO airport code, or UN/LOCODE. If none is provided, the caller IP is used. When more than one is supplied, the API resolves in this order: time zone name, coordinates, location, IP, IATA, ICAO, UN/LOCODE.
- Optional input: **Language** (paid only) for the location text.
- Output: the `time_zone` object, including `name`, `offset`, `offset_with_dst`, `current_time`, `current_time_unix`, current and standard abbreviations and full names, `is_dst`, and the DST transition details. The response also includes enrichment based on the input type: geolocation for an IP, location metadata for an address, airport details for IATA or ICAO codes, and lo_code details for a UN/LOCODE. See the [Time Zone field reference](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-zone-api-response).

![Pabbly Connect Lookup Timezone Information action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/timezone-api.png)

**Time Conversion**

Converts a time from a source location or zone to a destination location or zone.

- Optional input: **Time**, the datetime to convert, in `yyyy-MM-dd HH:mm` or `yyyy-MM-dd HH:mm:ss` format. If omitted, the current time is used.
- Source and destination inputs (provide a matching pair): time zone names (From Timezone and To Timezone in IANA format), coordinates (From Latitude and From Longitude, To Latitude and To Longitude, with latitude from -90 to 90 and longitude from -180 to 180), locations, IATA codes, ICAO codes, or UN/LOCODEs.
- Output: the converted date and time between the two locations. See the [Time Conversion field reference](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-conversion-api-response).

![Pabbly Connect Time Conversion action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/time-conversion.png)

### Astronomy

**Lookup Astronomy**

Returns Sun and Moon data for a location on a single date.

- Lookup inputs (provide one): Latitude and Longitude together, a Location name such as `Lahore, Pakistan`, or an IP address. If none is provided, the caller IP is used. When more than one is supplied, coordinates take priority, then location, then IP.
- Optional inputs: **Date** in `YYYY-MM-DD` format (defaults to the current date), **Elevation** in meters (default 0, maximum 10,000), **Time Zone** to return event times in a specific IANA zone, and **Language** (paid only) for the location text.
- Output: a `location` object and an `astronomy` object with fields such as `sunrise`, `sunset`, `solar_noon`, `day_length`, `mid_night`, `night_begin`, `night_end`, `moonrise`, `moonset`, `moon_phase`, `moon_illumination_percentage`, `sun_altitude`, `sun_azimuth`, `sun_distance`, `moon_altitude`, `moon_azimuth`, `moon_distance`, and the morning and evening twilight, blue hour, and golden hour times. See the [Astronomy field reference](https://ipgeolocation.io/documentation/astronomy-api.html#reference-to-astronomy-api-response).

![Pabbly Connect Lookup Astronomy action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/astronomy-api.png)

**Lookup Astronomy Timeseries**

Returns daily Sun and Moon data across a date range.

- Required inputs: Start Date and End Date in `YYYY-MM-DD` format. The maximum range is 90 days, and the start date must be before or equal to the end date.
- Lookup inputs (provide one): Latitude and Longitude together, a Location name, or an IP address.
- Optional inputs: **Elevation** in meters, **Time Zone**, and **Language** (paid only).
- Output: a `location` object and an `astronomy` array with one entry per day. Each entry includes the daily rise, set, twilight, `solar_noon`, `day_length`, `moon_phase`, `moonrise`, and `moonset` values. See the [Astronomy field reference](https://ipgeolocation.io/documentation/astronomy-api.html#reference-to-astronomy-api-response).

![Pabbly Connect Lookup Astronomy Timeseries action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/astronomy-timeseries.png)

### User Agent Parsing

**Parse User Agent**

Parses a single user agent string into structured client details. Requires a `paid` plan.

- Required input: User Agent, a user agent string, for example `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3`.
- Output: the `user_agent` object, including `name`, `type`, `version`, and `version_major`, plus the nested `device` (name, type, brand, cpu), `engine` (name, type, version), and `operating_system` (name, type, version, build) details. See the [User Agent field reference](https://ipgeolocation.io/documentation/user-agent-api.html#reference-to-user-agent-api-response).

![Pabbly Connect Parse User Agent action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/user-agent.png)

**Parse Bulk User Agents**

Parses many user agent strings in a single request. Requires a `paid` plan.

- Required input: User Agents, a list of up to 50,000 user agent strings, for example `["UA_String1", "UA_String2"]`.
- Output: an array of parsed `user_agent` objects, one entry per string. See the [User Agent field reference](https://ipgeolocation.io/documentation/user-agent-api.html#reference-to-user-agent-api-response).

![Pabbly Connect Parse Bulk User Agents action setup with input fields](https://static.ipgeolocation.io/web-assets/images/integrations/pabbly/bulk-user-agents.png)

---

## Example Workflow

This example enriches incoming IP data and routes a notification based on the visitor country.

1. **Trigger (Webhook):** A webhook receives an event that contains a visitor IP address.
2. **Get IP Geolocation:** Map the IP address from the webhook into the IP Address field to return the country, city, and other location fields.
3. **Router or Filter:** Branch the workflow based on the returned country name.
4. **Send a notification:** Send a message to your team channel or email tool for the branch you care about, including the location fields from step 2.

```bash
Webhook (visitor IP)
      |
      v
Get IP Geolocation
      |
      v
Router / Filter on country
      |
      v
Send notification (Slack, email, or other app)
```

You can extend this workflow by adding **Find IP Security** after the geolocation step to check whether the IP is a proxy, VPN, or Tor exit before deciding how to route the event.

---

## Common Use Cases

- **Lead enrichment:** Add country, city, ISP, and ASN to inbound leads using **Get IP Geolocation**, then write the enriched record to your CRM or spreadsheet.
- **Fraud detection:** Screen signup or checkout IP addresses with **Find IP Security** to flag proxy, VPN, Tor, or high threat score addresses for review.
- **Security automation:** Combine **Get IP Geolocation** and **Find IP Security** to detect logins from unusual locations and send an alert to your team.
- **CRM enrichment:** Append location and network fields to contact or account records so your team has context on where a lead is based.
- **Geo-based notifications:** Route messages, assign tickets, or trigger campaigns based on the country or time zone returned by **Get IP Geolocation** or **Lookup Timezone Information**.

---

## Troubleshooting

| Issue | Likely cause | Resolution |
| --- | --- | --- |
| Invalid API key | The API key is missing, mistyped, or inactive. The response reads "Provided API key is not valid." | Re-copy the key from your [dashboard](https://app.ipgeolocation.io/dashboard) and paste it again into the connection. Confirm the key is active. |
| Authentication failure | The connection was saved with an incorrect key. | Open the Connections tab, add a new connection with the correct key, and select it for the action. |
| Paid feature on a free plan | A paid action, such as IP Security, ASN, Abuse Contact, User Agent, or any bulk action, was run with a free plan key. The response indicates the feature is available to paid subscriptions only. | Upgrade on the [pricing page](https://ipgeolocation.io/pricing.html) or switch the action to one available on your plan. |
| Missing required fields | A required field, such as IP Address or User Agent, is empty. | Fill in the required field for the action, or map a value from an earlier step, then run the test again. |
| Include or Language ignored on a free plan | The Include and Language parameters are paid features. | Use a paid plan key, or rely on the Fields and Excludes parameters, which work on all plans. |
| Daily usage limit reached | The free plan includes a daily request allowance. | Wait for the daily reset or upgrade your plan for a higher limit. See the [pricing page](https://ipgeolocation.io/pricing.html). |

If the problem continues, contact IPGeolocation.io support at [support@ipgeolocation.io](mailto:support@ipgeolocation.io).

---

## Best Practices

- **Keep API keys secure.** Treat your API key like a password. Do not share it in public workflows or screenshots.
- **Use variables.** Map values from earlier steps into IPGeolocation.io fields rather than hardcoding them, so the workflow adapts to each event.
- **Request only what you need.** Use the Fields and Excludes parameters to keep responses small and focused, which reduces processing on downstream steps.
- **Test workflows before enabling.** Use **Save & Send Test Request** to confirm each action returns the fields you expect before you turn the workflow on.
- **Handle API failures.** Add a filter or conditional step so the workflow behaves predictably when an action returns an error or an empty result.
- **Validate IP addresses.** Confirm the incoming value is a valid IPv4 or IPv6 address before sending it to an action to avoid unnecessary failed requests.
- **Match the action to your plan.** Confirm your key has the required plan permissions before using paid actions such as IP Security, ASN, Abuse Contact, User Agent, and bulk lookups.

---

## Related Documentation

- [IP Geolocation API](https://ipgeolocation.io/documentation/ip-location-api.html)
- [IP Security API](https://ipgeolocation.io/documentation/ip-security-api.html)
- [ASN API](https://ipgeolocation.io/documentation/asn-api.html)
- [IP Abuse Contact API](https://ipgeolocation.io/documentation/ip-abuse-contact-api.html)
- [Time Zone API](https://ipgeolocation.io/documentation/timezone-api.html)
- [Astronomy API](https://ipgeolocation.io/documentation/astronomy-api.html)
- [User-Agent API](https://ipgeolocation.io/documentation/user-agent-api.html)
- [API documentation overview and authentication](https://ipgeolocation.io/documentation.html)
- [Manage your API keys in the dashboard](https://app.ipgeolocation.io/dashboard)
- [All integrations](https://ipgeolocation.io/integrations.html)
- [Pabbly Connect IPGeolocation.io integration page](https://connect.pabbly.com/integrations/ipgeolocation-io)