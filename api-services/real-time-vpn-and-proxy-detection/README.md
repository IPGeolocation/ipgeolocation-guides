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
| `actual_country_code` | string | **The country to build rules on** for risk decisions, fraud, and compliance. `public_ip_country_code` is only the country the user is presenting. |
| `confidence_score` | up to 100 | **Certainty about the recovered location.** Gate on it before acting on a country mismatch: a low value means the real address was only partially recovered and should not carry a decision alone. |

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
const live = result.live_vpn_proxy_detection;
const real = result.visitor_actual_location;
const sec  = result.ip_security;

live.confidence_score;                                        // 90
real.actual_country_code;                                     // "PK"
real.actual_country_code !== result.public_ip_country_code;   // true: presenting DE, actually in PK
sec.vpn_provider_names;                                       // ["Browsec VPN"]
sec.threat_score;                                             // 50
```

---

## Acting on the verdict

When `is_anonymous` is `true`, `confidence_score` decides how much friction the session deserves. Blocking everything the service flags costs real customers; ignoring the flag defeats the check. Work in bands, and tune the cutoffs against your own confirmed fraud outcomes:

| `confidence_score` (with `is_anonymous: true`) | What it means | Recommended action |
|---|---|---|
| Below 30 | Very weak signal. Expect false positives. | **Allow.** Log the result, add nothing to the user's path. |
| 30 to 50 | Uncertain. Could be an unusual network, a mobile carrier, or a corporate gateway. | **Allow and monitor.** Feed it into your wider risk score; add friction only if another signal agrees. |
| 51 to 80 | Likely anonymized. Enough for friction, not for a hard denial alone. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. Hold payouts and first orders for review. |
| 81 and above | Confidently anonymized. | **Block or restrict.** High `proxy_score` usually means abuse: block or hold for review. High `vpn_score` deserves a hard challenge instead, since many ordinary people browse through VPNs. |

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

**Do I need an API key in the frontend?**
No. Authorization is by Request Origin: register your domain once and requests from it and its subdomains authenticate automatically. No secret ships to the browser.

**Can I use this from a backend or a mobile app?**
No, the script is browser-only. For backend, mobile, or bulk checks, use the [IP Security API](https://ipgeolocation.io/documentation/ip-security-api.html), which returns the same reputation data through a key-authenticated API call.

**How long does a verdict take?**
1 to 5 seconds, because live network tests run against the connection rather than a single database lookup. Start monitoring on page load so the wait is over before the user reaches your decision point.

**Can it tell where a VPN or proxy user actually is?**
Yes. `visitor_actual_location` returns the `actual_ip` and `actual_country_code` recovered by the live tests, with its own `confidence_score`. Use that country for fraud and compliance rules, and gate on the confidence before acting on a mismatch.

**Does it detect residential proxies?**
Yes, and that is the main reason it exists. Blocklists miss residential proxies because the IPs belong to ordinary consumer ISPs; live analysis catches the anonymization behavior itself.

**Is `confidence_score` a risk score?**
No. It measures certainty about the verdict it sits next to, in either direction: `100` with `is_anonymous: false` means confidently clean. Risk lives in `proxy_score`, `vpn_score`, and `ip_security.threat_score`.

**Should I block every anonymized user?**
Usually not. Many are ordinary privacy-conscious or corporate users. Work the confidence bands: log the weak signals, challenge the likely ones, and reserve blocks for high confidence backed by a second signal.

**What does it cost?**
3 credits per call, or 5 with `includeIPSecurity: true`, from your plan's normal credit pool. It is a paid plan feature; contact [our support team](https://ipgeolocation.io/contact.html) for a free trial.

**Can users bypass it?**
Client-side code can always be tampered with, which is why the result should feed a server-side decision alongside your own signals rather than being the decision.

**Why did my call fail with the `Analysis` global undefined or a rejected promise?**
The script had not loaded yet (keep the tag above your integration code, or run from its `load` event), or the calling domain is not registered as a Request Origin. Fail open for that session either way.

---

