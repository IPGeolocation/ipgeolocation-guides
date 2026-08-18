# Real-Time VPN and Proxy Detection

## Overview

Detect VPNs, proxies and residential proxies live in the browser, at the moment a user acts. See what the service does and try it on the [Real-Time VPN and Proxy Detection product page](https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html).

> **Note:** A lightweight client-side script, much like a CAPTCHA widget. A verdict takes a few seconds, so start it early, read it at your decision point, and never hold up your initial render.

---

## Setup

The script carries no API key. Requests are authorized by origin, so register your domain before your first call.

1. [Sign up](https://app.ipgeolocation.io/signup) for your IPGeolocation account.
2. Add your origin as a [Request Origin](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use), for example `https://app.example.com`.
3. Save.

---

## Quick start

```html
<script src="https://static.ipgeolocation.io/web-assets/static/security/session-analysis.js"></script>

<script>
    const includeIPSecurity = false;

    Analysis
        .startMonitoring(includeIPSecurity)
        .get()
        .then(result => console.log(result))
        .catch(error => console.error(error instanceof Error ? error.message : String(error)));
</script>
```

Response for a clean connection:

```json
{
  "public_ip": "119.156.x.x",
  "public_ip_country_code": "PK",
  "live_vpn_proxy_detection": {
    "is_anonymous": false,
    "confidence_score": 100,
    "proxy_score": 0,
    "vpn_score": 0
  },
  "visitor_actual_location": {
    "actual_ip": "119.156.x.x",
    "actual_country_code": "PK",
    "confidence_score": 100
  }
}
```

In plain words: this connection is not anonymized, the service is certain about it, and the visitor is where the request says they are.

---

## Response fields

### The IP address the request comes from

Two top-level fields, describing the IP address the user appears to be on:

| Field | Type | Meaning |
|---|---|---|
| `public_ip` | string | The IP address your server sees on the request. Behind a VPN or proxy this is the exit node's IP address, not the user's. |
| `public_ip_country_code` | string | Two-letter ISO country code for that IP address. Behind a VPN or proxy, this is the exit node's country, not the user's. |

### The live verdict: `live_vpn_proxy_detection`

Four fields, meant to be read together:

| Field | Type | Meaning |
|---|---|---|
| `is_anonymous` | boolean | **The headline answer.** `true` means the connection is anonymized through a VPN or a proxy. It does not say which; the two scores below do. |
| `confidence_score` | up to 100 | **How much to trust that yes or no.** `100` with `is_anonymous: false` means confidently clean; a low value is weak evidence either way. |
| `proxy_score` | 0 to 100 | **Looks like a proxy**, including residential and rotating proxies. |
| `vpn_score` | 0 to 100 | **Looks like a VPN.** |

### Where the visitor really is: `visitor_actual_location`

Where the user actually is, recovered by the live tests:

| Field | Type | Meaning |
|---|---|---|
| `actual_ip` | string | **The visitor's real IP address** behind the VPN or proxy. Falls back to `public_ip` when nothing is hidden, or when the real IP cannot be detected. |
| `actual_country_code` | string | Two-letter ISO country code of the visitor's actual location. |
| `confidence_score` | up to 100 | Certainty about that actual country. Gate on it before acting on a country mismatch. |

### What is already known about the IP: `ip_security`

Present only when `includeIPSecurity` is `true`. Every field is documented in the [IP Security API response reference](https://ipgeolocation.io/documentation/ip-security-api.html#reference-to-ip-security-api-response).

```json
{
  "public_ip": "94.237.x.x",
  "public_ip_country_code": "DE",
  "live_vpn_proxy_detection": {
    "is_anonymous": true,
    "confidence_score": 90,
    "proxy_score": 90,
    "vpn_score": 10
  },
  "visitor_actual_location": {
    "actual_ip": "119.156.x.x",
    "actual_country_code": "PK",
    "confidence_score": 80
  },
  "ip_security": {
    "threat_score": 50,
    "is_tor": false,
    "is_proxy": true,
    "proxy_provider_names": [],
    "proxy_confidence_score": 99,
    "proxy_last_seen": "",
    "is_residential_proxy": false,
    "is_vpn": true,
    "vpn_provider_names": ["Browsec VPN"],
    "vpn_confidence_score": 99,
    "vpn_last_seen": "2026-08-07",
    "is_relay": false,
    "relay_provider_name": "",
    "is_anonymous": false,
    "is_known_attacker": false,
    "is_bot": false,
    "is_spam": false,
    "is_cloud_provider": true,
    "cloud_provider_name": "UpCloud Ltd"
  }
}
```

Reading it in code:

```js
const liveDetection  = result.live_vpn_proxy_detection;
const actualLocation = result.visitor_actual_location;
const ipSecurity     = result.ip_security;

liveDetection.confidence_score;                                        // 90
actualLocation.actual_country_code;                                    // "PK"
actualLocation.actual_country_code !== result.public_ip_country_code;  // true: presenting DE, actually in PK
ipSecurity.vpn_provider_names;                                         // ["Browsec VPN"]
ipSecurity.threat_score;                                               // 50
```

---

## Acting on the verdict

When `is_anonymous` is `true`, `confidence_score` decides how much friction the session deserves. Blocking everything the service flags costs real customers; ignoring the flag defeats the check. Work in bands, and tune the cutoffs against your own confirmed fraud outcomes:

| `confidence_score` | What it means | Recommended action |
|---|---|---|
| Below 30 | Very weak signal. Expect false positives. | **Allow.** Log the result, add nothing to the user's path. |
| 30 to 50 | Uncertain. Could be an unusual network, a mobile carrier, or a corporate gateway. | **Allow and monitor.** Feed it into your wider risk score; add friction only if another signal agrees. |
| 51 to 80 | Likely anonymized. Enough for friction, not for a hard denial alone. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. Hold payouts and first orders for review. |
| 81 and above | Confidently anonymized. | **Block or restrict.** High `proxy_score`: block or hold for review. High `vpn_score`: prefer a hard challenge, since many ordinary people browse through VPNs. |

These bands apply only when `is_anonymous` is `true`. High confidence with `is_anonymous: false` is a clean visitor, not a risky one.

---

## Credits

| Configuration | Credits per call |
|---|---|
| `includeIPSecurity: false` | 3 |
| `includeIPSecurity: true` | 5 |

> **Note:** Calls draw from your plan's shared credit pool. For how credits are counted and consumed across APIs, see the [credits usage guide](https://ipgeolocation.io/documentation/credits-usage.html); for plan allowances, see [pricing](https://ipgeolocation.io/pricing.html).

---

## FAQs

<details>
<summary><strong>Do I need an API key in the frontend?</strong></summary>
No. Authorization is by <a href="https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use">Request Origin</a>: register your domain once and requests from it and its subdomains authenticate automatically. No secret ships to the browser.
</details>

<details>
<summary><strong>Can I use this from a backend or a mobile app?</strong></summary>
No, the script is browser-only. For backend, mobile, or bulk checks, use the <a href="https://ipgeolocation.io/documentation/ip-security-api.html">IP Security API</a>, which returns only the reputation data through a key-authenticated API call.
</details>

<details>
<summary><strong>How long does a verdict take?</strong></summary>
Usually 1 to 5 seconds, because live network tests run against the connection rather than a single database lookup. Start monitoring on page load so the wait is over before the user reaches your decision point.
</details>

<details>
<summary><strong>Can it tell where a VPN or proxy user actually is?</strong></summary>
Yes. <code>visitor_actual_location</code> returns the <code>actual_ip</code> and <code>actual_country_code</code> recovered by the live tests, with its own <code>confidence_score</code>. Use that country for fraud and compliance rules, and gate on the confidence before acting on a mismatch.
</details>

<details>
<summary><strong>Does it detect residential proxies?</strong></summary>
Yes, and that is the main reason it exists. Static lists miss them because the IPs come from constantly refreshed pools of ordinary consumer ISP addresses with no listing history yet; live analysis catches the anonymization behavior itself.
</details>

<details>
<summary><strong>How is this different from the VPN and proxy flags in the IP Security API?</strong></summary>
<code>ip_security.is_vpn</code> and <code>is_proxy</code> come from our IP intelligence database, updated daily: they tell you an IP address is known to be associated with VPN or proxy infrastructure. The live <code>vpn_score</code> and <code>proxy_score</code> come from inspecting the current connection, so they describe this session rather than the address's history.
</details>

<details>
<summary><strong>Does real-time detection run on top of the IP Security database?</strong></summary>
No. It classifies a session with live connection-level techniques only, and does not consult the IP Security database or traditional IP intelligence to reach its verdict. <code>includeIPSecurity: true</code> simply attaches the database view next to the live one.
</details>

<details>
<summary><strong>Why can a database alone not settle a residential proxy?</strong></summary>
A residential IP is a real household address that may also be part of a proxy network. The homeowner's own request and an anonymous request routed through the network can arrive from it seconds apart. A database can say the address is associated with residential proxy infrastructure, but not how it is being used on this connection, so acting on the association alone can affect legitimate users too.
</details>

<details>
<summary><strong>Will it catch VPNs and proxies no database has seen yet?</strong></summary>
Yes. No offline database has complete coverage: new VPN endpoints, private proxies, and rotating residential pools may not be listed yet. Real-time detection does not depend on prior knowledge of the IP address.
</details>

<details>
<summary><strong>How accurate is it?</strong></summary>
In our testing, classification accuracy is around 98%. In the remaining cases, <code>confidence_score</code> was below 35, so gating on confidence gives you an extra signal on the sessions most likely to be misread.
</details>

<details>
<summary><strong>Should I use this or the IP Security API?</strong></summary>
Both, where you can. The IP Security API gives broader IP-level context and known associations; real-time detection tells you what the connection is doing right now. Together they support better decisions without unnecessarily blocking legitimate users.
</details>

<details>
<summary><strong>Is <code>confidence_score</code> a risk score?</strong></summary>
No. It measures certainty about the verdict it sits next to, in either direction: <code>100</code> with <code>is_anonymous: false</code> means confidently clean. Risk lives in <code>proxy_score</code>, <code>vpn_score</code>, and <code>ip_security.threat_score</code>.
</details>

<details>
<summary><strong>Should I block every anonymized user?</strong></summary>
Usually not. Many are ordinary privacy-conscious or corporate users. Work the confidence bands: log the weak signals, challenge the likely ones, and reserve blocks for high confidence backed by a second signal.
</details>

<details>
<summary><strong>What does it cost?</strong></summary>
3 credits per call, or 5 with <code>includeIPSecurity: true</code>, from your plan's normal credit pool. It is a paid plan feature; contact <a href="https://ipgeolocation.io/contact.html">our support team</a> for a free trial.
</details>

---
