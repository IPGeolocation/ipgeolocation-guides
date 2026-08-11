<!-- ---
canonical: https://ipgeolocation.io/documentation/workato-integration
meta-application-name: IP Geolocation API
meta-category: IP Geolocation API Integrations
meta-description: Step-by-step guide to connect IPGeolocation.io with Workato. Configure 12 actions for IP geolocation, security, ASN, timezone, astronomy, and user agent data.
meta-generator: IPGeolocation
meta-keywords: ipgeolocation workato integration, workato ipgeolocation, connect ipgeolocation to workato, ip geolocation workato connector, workato ip security integration, ip intelligence workato
meta-og:description: Step-by-step guide to connect IPGeolocation.io with Workato. Configure 12 actions for IP geolocation, security, ASN, timezone, and user agent lookups.
meta-og:image: https://static.ipgeolocation.io/web-assets/images/brand/ipgeo-og.jpeg
meta-og:image:alt: IPGeolocation.io Logo
meta-og:title: IPGeolocation.io Workato Integration Documentation
meta-og:type: website
meta-og:url: https://ipgeolocation.io/documentation/workato-integration
meta-twitter:card: summary_large_image
meta-twitter:description: Step-by-step guide to connect IPGeolocation.io with Workato. Configure 12 actions for IP geolocation, security, ASN, timezone, and more.
meta-twitter:image: https://static.ipgeolocation.io/web-assets/images/brand/ipgeolocation-logo-with-bg.svg
meta-twitter:image:alt: IPGeolocation.io Logo
meta-twitter:site: @ipgeolocationio
meta-twitter:title: IPGeolocation.io Workato Integration Documentation
meta-viewport: width=device-width, initial-scale=1
title: IPGeolocation.io Workato Integration Documentation | Setup & Action Reference
--- -->

<!-- [Home](https://ipgeolocation.io/)1. [Integrations](https://ipgeolocation.io/integrations.html)
2. [Workato](https://ipgeolocation.io/integrations/workato)
3. Workato Integration Documentation -->

# IPGeolocation Workato Integration

## Overview

The [IPGeolocation.io connector for Workato](https://www.workato.com/integrations/ipgeolocation.io) brings the full [IPGeolocation.io v3](https://ipgeolocation.io) API into Workato recipes as 12 ready-made actions. There is no need to build HTTP requests, parse raw JSON, or manage authentication headers by hand. You add the connector to a recipe, connect your API key once, and every action is available across your workspace from that point on.

With this integration, every recipe you build can pull real-time intelligence on any IP address, including:

- Geolocation data such as country, region, city, ZIP code, and coordinates
- Security and threat intelligence, including proxy, VPN, Tor, and cloud provider detection with a threat score
- Network ownership and ASN details
- Abuse contact information for the organization behind an IP
- Timezone data and time conversion across time zones, coordinates, IATA, ICAO, and UN/LOCODE inputs
- Astronomy data, including sunrise, sunset, and moon phase, for a single day or a date range
- User agent parsing for browser, operating system, and device detection

This data feeds directly into fraud checks, lead enrichment, content personalization, compliance workflows, and security alerting, without writing a single line of custom integration code.

The connector groups its 12 actions into 6 categories:

| Category | Actions |
| --- | --- |
| **Geolocation** | Get IP Geolocation, Get Bulk IP Geolocation |
| **IP Security** | Get IP Security, Get Bulk IP Security |
| **Network Intelligence** | Lookup ASN Information, Find IP Abuse Contact |
| **Time Services** | Find Timezone Information, Convert Timezone |
| **Astronomy** | Lookup Astronomy Data, Lookup Astronomy Timeseries |
| **User Agent Parsing** | Parse User Agent, Parse Bulk User Agents |

---

## Connect IPGeolocation.io to Workato

You need a valid IPGeolocation.io API key to use this connector. Follow these steps to set up the connection.

---

### 1. Create or log in to your IPGeolocation.io account

- Go to [IPGeolocation.io](https://ipgeolocation.io/).
- New to IPGeolocation.io? Click [Sign Up](https://app.ipgeolocation.io/signup) and complete the registration.
- Already have an account? Click [Sign in](https://app.ipgeolocation.io/login) and enter your credentials.

---

### 2. Get your API key

- Once logged in, open your [dashboard](https://app.ipgeolocation.io/dashboard).
- Copy your **API Key**. You will paste this into Workato in a later step.

---

### 3. Open Workato and start a recipe

- Log in to your [Workato](https://www.workato.com/) workspace.
- Create a new recipe, or open an existing one where you want to add IP intelligence.
- Add a new action step and search for **IPGeolocation.io** in the app picker.
- Choose the action you need, for example **Get IP Geolocation** or **Get IP Security**.

---

### 4. Add a new connection

- On the connection screen, click to add a new connection.
- Give the connection a name so it is easy to identify if you connect multiple API keys later.
- Paste the **API Key** you copied from your IPGeolocation.io dashboard into the required field.

![Add a new IPGeolocation.io connection in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/add-connection.png)

---

### 5. Confirm the connection

- Click to save and connect.
- Workato validates the API key against IPGeolocation.io and confirms the connection is active.
- Once connected, the connection is saved to your workspace and ready to use.

![Successful IPGeolocation.io connection confirmation in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/successful-connection.png)

---

### 6. Reuse the connection across recipes

- Any action step that uses IPGeolocation.io in this workspace can reuse the same connection.
- Configure the action inputs, such as an IP address, IP array, or user agent string, using data pills from earlier steps or static values.
- Test the step and continue building your recipe.

---

## Action Availability by Plan

IPGeolocation.io offers two API plans: **Developer (Free)** and **Paid**. Your API key determines which actions you can use in Workato.

| Action                          | Developer (Free) | Paid |
| ------------------------------- | ---------------- | ---- |
| **Get IP Geolocation**          | ✔                | ✔    |
| **Get Bulk IP Geolocation**     |   ✖              | ✔    |
| **Get IP Security**             |  ✖               | ✔    |
| **Get Bulk IP Security**        |    ✖             | ✔    |
| **Lookup ASN Information**      |    ✖             | ✔    |
| **Find IP Abuse Contact**       |    ✖             | ✔    |
| **Find Timezone Information**   | ✔                | ✔    |
| **Convert Timezone**            | ✔                | ✔    |
| **Lookup Astronomy Data**       | ✔                | ✔    |
| **Lookup Astronomy Timeseries** | ✔                | ✔    |
| **Parse User Agent**            |    ✖             | ✔    |
| **Parse Bulk User Agents**      |    ✖             | ✔    |

See the [IPGeolocation.io pricing page](https://ipgeolocation.io/pricing.html) for plan details and information about upgrading your API key.

---

## Actions Reference

Each Workato action connects to an IPGeolocation.io API endpoint. The sections below describe the inputs and outputs available in each action. For the complete response schema, see the linked API documentation.

### 1. Geolocation

#### Get IP Geolocation

Retrieves geolocation data for a single IPv4 or IPv6 address.

**Inputs**

- **IP:** IPv4 or IPv6 address. Paid plans also support domain names.
- **Include:** Adds optional data modules to the response. See the [IP Geolocation API Include options](https://ipgeolocation.io/documentation/ip-location-api.html).
- **Fields:** Returns only the fields you specify.
- **Excludes:** Removes fields or objects from the response.
- **Language:** Returns location text in a supported language.

**Outputs**

Returns location, country metadata, currency, ASN, and time zone data, along with additional data available to the request. See the [IP Geolocation response fields](https://ipgeolocation.io/documentation/ip-location-api.html#reference-to-ipgeolocation-api-response).

![Get IP Geolocation in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/ipgeo.png)

#### Get Bulk IP Geolocation

Retrieves geolocation data for up to 50,000 IP addresses in a single request.

**Inputs**

- **IP Addresses:** Array of IPv4 or IPv6 addresses. Paid plans also support domain names.
- **Include:** Adds optional data modules to the response.
- **Fields:** Returns only the fields you specify.
- **Excludes:** Removes fields or objects from the response.

**Outputs**

Returns a collection of geolocation objects, one for each submitted address. See the [IP Geolocation API response reference](https://ipgeolocation.io/documentation/ip-location-api.html#reference-to-ipgeolocation-api-response).

![Get Bulk IP Geolocation in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/ipgeo-bulk.png)

---

### 2. IP Security

#### Get IP Security

Returns security and threat information for a single IP address, including proxy, VPN, Tor, relay, cloud provider, and threat score data.

**Inputs**

- **IP:** IPv4 or IPv6 address.
- **Fields:** Returns only the security fields you specify.
- **Excludes:** Removes security fields or objects from the response.

**Outputs**

Returns security data such as `security.is_proxy`, `security.is_vpn`, `security.is_relay`, `security.is_tor`, `security.threat_score`, and `security.is_cloud_provider`. See the [IP Security response fields](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference).

![Get IP Security in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/ip-security.png)

#### Get Bulk IP Security

Checks multiple IP addresses for security and threat information in a single request.

**Inputs**

- **IP Addresses:** Array of IPv4 or IPv6 addresses.
- **Fields:** Returns only the security fields you specify.
- **Excludes:** Removes security fields or objects from the response.

**Outputs**

Returns a collection of security objects. See the [IP Security API response fields](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference).

![Get Bulk IP Security in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/ip-security-bulk.png)

---

### 3. Network Intelligence

#### Lookup ASN Information

Retrieves information about an Autonomous System Number (ASN) and its associated IPv4 and IPv6 address ranges.

**Inputs**

- **IP or ASN:** Enter an IP address or ASN number.
- **Include:** Adds related network data to the response.
- **Fields:** Returns only the ASN fields you specify.
- **Excludes:** Removes ASN fields or objects from the response.

**Outputs**

Returns ASN information such as `asn.as_number`, `asn.organization`, `asn.country`, and additional ASN data. See the [ASN API response fields](https://ipgeolocation.io/documentation/asn-api.html#reference-to-asn-api-response).

![Lookup ASN in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/asn-api.png)

#### Find IP Abuse Contact

Retrieves abuse contact information associated with an IP address.

**Inputs**

- **IP:** IPv4 or IPv6 address.
- **Fields:** Returns only the abuse contact fields you specify.
- **Excludes:** Removes abuse contact fields or objects from the response.

**Outputs**

Returns abuse contact data such as `abuse.emails`, `abuse.kind`, `abuse.organization`, and other available contact details. See the [IP Abuse Contact API response fields](https://ipgeolocation.io/documentation/ip-abuse-contact-api.html#reference-to-abuse-contact-api-response).

![Find IP Abuse Contact in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/abuse-contact-api.png)

---

### 4. Time Services

#### Find Timezone Information

Retrieves time zone information using one of the supported input types.

**Inputs**

- **Time Zone:** IANA time zone name.
- **Location:** City or other supported address.
- **Coordinates:** Latitude and longitude.
- **IP:** IPv4 or IPv6 address.
- **IATA:** IATA airport code.
- **ICAO:** ICAO airport code.
- **UN/LOCODE:** United Nations Code for Trade and Transport Locations.

**Language:** Returns location text in a supported language

**Outputs**

Returns time zone data such as `time_zone.name`, `time_zone.current_time`, and `time_zone.date_time_wti`, along with other available time zone information. See the [Time Zone API response fields](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-zone-api-response).

![Find Timezone Information in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/timezone-api.png)

#### Convert Timezone

Converts a date and time between supported locations or time zones.

**Inputs**

- **Time:** Date and time to convert. If omitted, the API uses the current time.
- **From:** Source time zone, location, coordinates, IATA code, ICAO code, or UN/LOCODE.
- **To:** Destination time zone, location, coordinates, IATA code, ICAO code, or UN/LOCODE.

**Outputs**

Returns the converted date and time. See the [Time Conversion API response fields](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-conversion-api-response).

![Convert Timezone in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/time-conversion.png)

---

### 5. Astronomy

#### Lookup Astronomy Data

Returns Sun and Moon data for a location and date.

**Inputs**

- **Location:** Latitude and longitude, location name, or IP address.
- **Date:** Date for the astronomy data.
- **Elevation:** Elevation in meters.
- **Time Zone:** IANA time zone.
- **Language:** Returns location text in a supported language.

**Outputs**

Returns data such as `astronomy.sunrise`, `astronomy.sunset`, `astronomy.moonrise`, `astronomy.moonset`, `astronomy.moon_phase`, and other astronomy information. See the [Astronomy API response fields](https://ipgeolocation.io/documentation/astronomy-api.html#reference-to-astronomy-api-response).

![Lookup Astronomy Data in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/astronomy-api.png)

#### Lookup Astronomy Timeseries

Returns daily Sun and Moon data across a date range.

**Inputs**

- **Start Date:** Start of the date range.
- **End Date:** End of the date range.
- **Location:** Latitude and longitude, location name, or IP address.
- **Elevation:** Elevation in meters.
- **Time Zone:** IANA time zone.
- **Language:** Returns location text in a supported language.

**Outputs**

Returns daily astronomy data for each date in the requested range. See the [Astronomy Timeseries API response fields](https://ipgeolocation.io/documentation/astronomy-api.html#reference-to-astronomy-api-response).

![Lookup Astronomy Timeseries in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/astronomy-timeseries.png)

---

### 6. User Agent Parsing

#### Parse User Agent

Parses a user agent string into structured client information.

**Inputs**

- **User Agent:** Raw user agent string.

**Outputs**

Returns browser, device, operating system, and other user agent information. See the [User Agent API response fields](https://ipgeolocation.io/documentation/user-agent-api.html#reference-to-user-agent-api-response).

![Parse User Agent in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/user-agent.png)

#### Parse Bulk User Agents

Parses multiple user agent strings in a single request.

**Inputs**

- **User Agents:** Array of user agent strings.

**Outputs**

Returns a collection of parsed user agent objects. See the [User Agent API response fields](https://ipgeolocation.io/documentation/user-agent-api.html#reference-to-user-agent-api-response).

![Parse Bulk User Agents in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/bulk-user-agents.png)

## Troubleshooting Connection Errors

Most connection issues in Workato come down to one of two causes. Both are easy to fix.

### Invalid credentials

If Workato shows an invalid credentials error when you save the connection, the API key was not accepted.

![IPGeolocation.io invalid credentials error in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/invalid-credentials.png)

- Open your [dashboard](https://app.ipgeolocation.io/dashboard) and confirm you copied the full API key with no extra spaces.
- Confirm the key belongs to an active account and has not been regenerated or revoked.
- Create a new connection in Workato with the confirmed key rather than editing the failed one.

---

### Paid action requested with a free API key

If a recipe step fails on an action such as Get IP Security, Get Bulk IP Geolocation, or Parse User Agent, and your key is on the Developer (Free) plan, you will see an error stating the action requires a paid plan.

![IPGeolocation.io paid action requested by a free API key error in Workato](https://static.ipgeolocation.io/web-assets/images/integrations/workato/if-paid-action-by-free-api-key.png)

- Check the [Action Availability by Plan](#action-availability-by-plan) table above to confirm which plan an action needs.
- Upgrade your API key on the [pricing page](https://ipgeolocation.io/pricing.html) to unlock the action.
- Re-run the recipe step once your plan is active. No changes to the recipe itself are required.

---

## Recipe Use Cases

### 1. Enrich Leads with IP Geolocation

Add location data to new leads before sending them to your CRM or other business systems.

**Recipe:** New lead → Get IP Geolocation → Update CRM

Use **Get IP Geolocation** to retrieve the lead's country, region, and city from the IP address. Pass the returned data to the next action in your Workato recipe.

---

### 2. Detect Risky IP Addresses

Check IP security data during login or transaction workflows and alert your team when an IP matches your security rules.

**Recipe:** Login or transaction event → Get IP Security → IF/THEN → Slack or email

Use **Get IP Security** to check signals such as proxy, VPN, Tor, relay, cloud provider, and threat score data. Use an **IF/THEN condition** to decide when the recipe should send an alert or take another action.

---

### 3. Personalize Workflows by Location

Use a customer's location to route records or apply different actions based on country or region.

**Recipe:** New customer or form submission → Get IP Geolocation → IF/THEN → Application action

Use **Get IP Geolocation** to retrieve location data from the customer's IP address. Use an **IF/THEN condition** to route the recipe based on the returned country, region, or city.

---

### 4. Route Support Workflows by Time Zone

Use a customer's location to determine their time zone and apply time-based rules to support workflows.

**Recipe:** New support ticket → Get IP Geolocation → Find Timezone Information → Support action

Use **Get IP Geolocation** to identify the customer's location, then use **Find Timezone Information** to determine the relevant time zone. Use the result to route the ticket or schedule the next action according to your workflow rules.

---

## Frequently Asked Questions

<details>
<summary><strong>Is there a free way to try the IPGeolocation.io Workato integration?</strong></summary>

Yes. IPGeolocation.io offers a <strong>Developer (Free)</strong> API key that works with <strong>Get IP Geolocation</strong>, <strong>Find Timezone Information</strong>, <strong>Convert Timezone</strong>, <strong>Lookup Astronomy Data</strong>, and <strong>Lookup Astronomy Timeseries</strong> in Workato. No credit card is required to start.

</details>

<details>
<summary><strong>How many actions does the IPGeolocation.io Workato connector include?</strong></summary>

The connector includes <strong>12 actions</strong> across geolocation, IP security, network intelligence, time services, astronomy, and user agent parsing. See the <a href="#actions-reference">Actions Reference</a> for the complete list.

</details>

<details>
<summary><strong>Do I need to write code to use IPGeolocation.io in a Workato recipe?</strong></summary>

No. Each action is available as a <strong>pre-built step</strong> in the Workato recipe editor. Connect your API key and configure the action inputs using <strong>data pills</strong>. You do not need to make manual API requests or parse JSON responses.

</details>

<details>
<summary><strong>Can I look up multiple IP addresses in a single Workato action?</strong></summary>

Yes. <strong>Get Bulk IP Geolocation</strong> and <strong>Get Bulk IP Security</strong> accept an array of IP addresses and return a collection of results in a single action. <strong>Parse Bulk User Agents</strong> also accepts multiple user agent strings and returns a collection of parsed results.

</details>

<details>
<summary><strong>Why is my Workato connection to IPGeolocation.io failing?</strong></summary>

Check that your <strong>API key is correct and active</strong>. Also check that the action you are using is available on your API plan. Actions that require the <strong>Paid</strong> plan will not work with a <strong>Developer (Free)</strong> API key. See <a href="#troubleshooting-connection-errors">Troubleshooting Connection Errors</a> for more information.

</details>

<details>
<summary><strong>Does the Workato integration support timezone conversion?</strong></summary>

Yes. <strong>Find Timezone Information</strong> retrieves timezone data, including the current time and offset, for supported locations. <strong>Convert Timezone</strong> converts a specific date and time between supported time zones using inputs such as time zone names, coordinates, IP addresses, and location codes.

</details>


---

## Related Integrations

IPGeolocation.io connects to more than 20 platforms beyond Workato. If your stack includes one of these, the same API key works everywhere.

- [Zapier Integration Documentation](https://ipgeolocation.io/documentation/zapier-integration)
- [Make Integration Documentation](https://ipgeolocation.io/documentation/make-integration)
- [n8n Integration Documentation](https://ipgeolocation.io/documentation/n8n-integration)
- [Pipedream Integration Documentation](https://ipgeolocation.io/documentation/pipedream-integration)
- [View all integrations](https://ipgeolocation.io/integrations.html)