# Developer Docs for Real-Time Proxy and VPN Detection

## Overview

Detect VPNs, proxies and residential proxies live in the browser, at the moment a user acts. See what the service does and how it is priced on the [Real-Time VPN and Proxy Detection product page](https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html).

> **Note:** A lightweight client-side script, much like a CAPTCHA widget. A verdict takes a few seconds, so start it early, read it at your decision point, and never hold up your initial render.

---

## Setup

Real-time detection is a paid plan feature; a free trial is available through [our support team](https://ipgeolocation.io/contact.html). The script carries no API key, so register the domain you will call from as a Request Origin:

1. [Sign up](https://app.ipgeolocation.io/signup) for your IPGeolocation account.
2. Add your origin, for example `https://app.example.com`.
3. Save.

> **Note:** Registration covers the domain and all of its subdomains, and requests from unregistered origins are rejected. Step-by-step walkthrough: [set up a Request Origin](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use).

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

## API reference

| Call | Description |
|---|---|
| `Analysis.startMonitoring(includeIPSecurity)` | Starts monitoring the current session and returns a handle. Call it as early as possible. `includeIPSecurity` (boolean): `false` returns the live verdict and actual location for 3 credits; `true` adds the `ip_security` object for 5 credits total. |
| `.get()` | Returns a `Promise` that resolves with the result object in 1 to 5 seconds and rejects with an `Error` when the analysis cannot complete. Always attach a `.catch`. |

The script is lightweight and promise-based. Load it with `async` or `defer` if you like, but run your integration code only after it has loaded (for example from the tag's `load` event), because the `Analysis` global does not exist before then.

Recommended pattern:

```html
<script src="https://static.ipgeolocation.io/web-assets/static/security/session-analysis.js"></script>

<script>
    // Start early so the verdict is ready by the decision point.
    const session = Analysis.startMonitoring(true);

    document.getElementById('checkout-btn').addEventListener('click', async () => {
        try {
            const { live_vpn_proxy_detection: live } = await session.get();
            if (live.is_anonymous && live.confidence_score >= 51) {
                // Challenge: MFA, email or SMS verification, or a CAPTCHA.
            }
            // Send the payload to your backend and make the final call there.
        } catch (error) {
            // Fail open: never let a detection error break the flow.
            console.error(error instanceof Error ? error.message : String(error));
        }
    });
</script>
```

> **Important:** Treat the browser result as a signal, not the decision. Client-side code can be tampered with, so decide server-side, next to your own risk data.

---

## Response fields

### Top level

| Field | Type | Meaning |
|---|---|---|
| `public_ip` | string | The address your server sees on the request. Behind a VPN or proxy this is the exit node, not the user. |
| `public_ip_country_code` | string | Two-letter ISO country code for that address. Same caveat: behind an anonymizer it is where the exit node is. |

### live_vpn_proxy_detection

The live verdict on the connection, four fields meant to be read together:

1. **`is_anonymous`** (boolean) — the headline answer. `true` means the connection is anonymized through a VPN or a proxy; it does not say which. Read the two scores for the type.
2. **`confidence_score`** (number, 0 to 100) — certainty about the `is_anonymous` value. `100` with `is_anonymous: false` means confidently clean; a low value is weak evidence either way.
3. **`proxy_score`** (number, 0 to 100) — likelihood the connection runs through a proxy, including residential and rotating proxies.
4. **`vpn_score`** (number, 0 to 100) — likelihood the connection runs through a VPN.

> **Note:** `proxy_score` and `vpn_score` are independent likelihoods, not two halves of a split. Both can be high, and a 100 in one does not force a 0 in the other.

### visitor_actual_location

Where the visitor really is, recovered from behind the anonymizer:

1. **`actual_ip`** (string) — the visitor's real address behind the VPN or proxy, recovered by the live tests rather than read from request headers. Matches `public_ip` when nothing is hidden.
2. **`actual_country_code`** (string) — country of `actual_ip`. Use this one for risk decisions, fraud rules, and compliance checks; `public_ip_country_code` is only the country the user is presenting.
3. **`confidence_score`** (number, 0 to 100) — certainty about the recovered location. Gate on it before acting on a country mismatch; a low value means the real address was only partially recovered and should not carry a decision on its own.

> **Important:** Every `confidence_score` is certainty, not risk. Risk lives in `proxy_score`, `vpn_score`, and `ip_security.threat_score`.

### ip_security (when includeIPSecurity is true)

Adds the IP's reputation from the IP Security database as a flat object: `threat_score`, the anonymizer flags (`is_vpn`, `is_proxy`, `is_residential_proxy`, `is_relay`, `is_tor`), provider names with confidence scores and last-seen dates, the abuse flags (`is_known_attacker`, `is_spam`, `is_bot`), and the cloud-provider fields.

These fields belong to the IP Security API, so they are documented once, there: see the [IP Security API response reference](https://ipgeolocation.io/documentation/ip-security-api.html#reference-to-ip-security-api-response) for every field's type and description.

A flagged session with `includeIPSecurity: true`:

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

> **Note:** The live verdict and `ip_security` can disagree on the type, and that is normal. Browser VPN extensions route through proxy protocols, so live tests read them as proxies while the database lists the brand as a VPN. There are also two `is_anonymous` fields: live `true` with database `false` means anonymization no blocklist has caught yet, which is the residential proxy case. Read live scores for behavior, `ip_security` for reputation, and take the higher-risk reading.

---

## Acting on the verdict

When `is_anonymous` is `true`, `confidence_score` decides how much friction the session deserves. Blocking everything the service flags costs real customers; ignoring the flag defeats the check. Work in bands, and tune the cutoffs against your own confirmed fraud outcomes:

| `confidence_score` (with `is_anonymous: true`) | What it means | Recommended action |
|---|---|---|
| Below 30 | Very weak signal. Expect false positives. | **Allow.** Log the result, add nothing to the user's path. |
| 30 to 50 | Uncertain. Could be an unusual network, a mobile carrier, or a corporate gateway. | **Allow and monitor.** Feed it into your wider risk score; add friction only if another signal agrees. |
| 51 to 80 | Likely anonymized. Enough for friction, not for a hard denial alone. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. Hold payouts and first orders for review. |
| 81 and above | Confidently anonymized. | **Block or restrict.** High `proxy_score` usually means abuse: block or hold for review. High `vpn_score` deserves a hard challenge instead, since many ordinary people browse through VPNs. |

Signals that push a decision harder:

- `ip_security.threat_score >= 70` escalates one band.
- `is_known_attacker`, `is_spam`, or `is_bot` escalates to block or hard review.
- `is_cloud_provider: true` on a consumer flow means automation: rate limit or challenge regardless of confidence.
- `actual_country_code` differing from `public_ip_country_code` (backed by a solid `visitor_actual_location.confidence_score`) escalates on payment and compliance flows.
- One signal alone rarely justifies a block; two or three pointing the same way usually do.

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

## Related

- [IP Security API documentation](https://ipgeolocation.io/documentation/ip-security-api.html): the server-side counterpart, and the reference for every `ip_security` field.
- [Credits usage guide](https://ipgeolocation.io/documentation/credits-usage.html)
- [Request Origin setup tutorial](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use)
- [Product page and live test](https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html)
- [Plan pricing](https://ipgeolocation.io/pricing.html) and [signup](https://app.ipgeolocation.io/signup)
- [Guides and SDKs on GitHub](https://github.com/IPGeolocation/ipgeolocation-guides)
