# Real-Time Proxy and VPN Detection

Detect VPNs, proxies and residential proxies live in the browser, at the moment a user acts.

Product page: <https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html>

Blocklists miss residential proxies, because those IPs belong to ordinary consumer ISPs and look clean. This runs at request time and watches how the connection actually behaves, so it catches anonymization no list has caught yet, and it gives you graded scores instead of a bare yes or no.

---

## 1. Setup

The script runs in the browser and carries no API key. Requests are authorized by **origin** instead, so you have to register the domain you will call from as a **Request Origin** in your IPGeolocation dashboard before your first call works.

1. Sign in at <https://ipgeolocation.io>.
2. Add your origin, for example `https://app.example.com`.
3. Save.

> **Guide:** [How to add a Request Origin in the IPGeolocation dashboard](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use)

---

## 2. Quick start

```html
<script src="https://static.ipgeolocation.io/web-assets/static/security/session-analysis.js"></script>

<script>
    const includeIPSecurity = false;

    Analysis
            .startMonitoring(includeIPSecurity)
            .get()
            .then(result => console.log(result))
            .catch(error => console.error(error));
</script>
```

Response:

```json
{
  "live_vpn_proxy_detection": {
    "is_anonymous": false,
    "confidence": 100,
    "proxy_score": 0,
    "vpn_score": 0
  }
}
```

In plain words: this connection is not anonymized, and we are certain about it.


---

## 3. `live_vpn_proxy_detection`

Always present. Four fields, and they are meant to be read together.

| Field | Range     | Meaning |
|---|-----------|---|
| `is_anonymous` | boolean   | The headline answer. `true` means the connection is anonymized, either through a VPN or through a proxy. It does not tell you which of the two. |
| `confidence` | up to 100 | How certain the service is about the `is_anonymous` value above. **This is not a risk score.** `confidence: 100` with `is_anonymous: false` means "confidently clean". A low value means the verdict is weak evidence either way. |
| `proxy_score` | 0 to 100  | Chance that the connection runs through a **proxy**, including residential and rotating proxies. |
| `vpn_score` | 0 to 100  | Chance that the connection runs through a **VPN**. |

The mental model: `is_anonymous` tells you whether the connection is hiding, `confidence` tells you how much to trust that yes or no, and `proxy_score` / `vpn_score` tell you what kind of hiding it looks like.

The two scores are independent likelihoods, not two halves of a split, so they do not add up to 100.

---

## 4. What to do at each confidence level

This is the part worth getting right. When `is_anonymous` is `true`, `confidence` decides how much friction the user deserves. Blocking everything the service flags will cost you real customers, and ignoring the flag defeats the point of running the check, so work in bands.

| `confidence` (with `is_anonymous: true`) | What it means | Recommended action |
|---|---|---|
| **Below 30** | Very weak signal. Expect false positives in this band. | **Allow.** Log the result for later analysis, add nothing to the user's path. Do not challenge on this alone. |
| **30 to 50** | Uncertain. Could easily be an unusual network, a mobile carrier or a corporate gateway. | **Allow and monitor.** Let the action through, tag the session as slightly elevated risk, feed it into your wider risk score. Add friction only if another signal agrees, such as a brand new account or a mismatched billing country. |
| **51 to 80** | Likely anonymized. Enough for friction, not enough for a hard denial on its own. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. On payouts and first orders, hold for manual review instead of denying outright. |
| **81 and above** | Confidently anonymized. | **Block or restrict**, with the score type deciding how hard. High `proxy_score` usually means abuse: block, or hold the order and send it to review. High `vpn_score` deserves a hard challenge instead, since plenty of ordinary people browse through a VPN. Two caveats: a high `proxy_score` next to a named consumer VPN in `vpn_provider_names` is not automatically abuse, since it can come from a browser extension, a desktop or CLI client, or a residential proxy (see section 5), and an outright block is safest when a second signal backs it up, such as `threat_score >= 70`, `is_known_attacker` or `is_bot`. |

Two things to keep in mind. Tune these numbers against your own confirmed fraud data, because the right cutoff depends on your traffic and on how much a false positive costs you. And since `confidence` describes certainty about `is_anonymous`, these bands apply only when `is_anonymous` is `true`. High confidence with `is_anonymous: false` is a clean visitor, not a risky one.


---

## 5. Full response when `includeIPSecurity` is `true`

### `security` is the block that matters

It holds the threat score, the anonymizer type and the name of the VPN or proxy provider. Everything else in `ip_security` is context around it, so if you read one block, read this one.

```json
"security": {
  "threat_score": 90,
  "is_tor": false,
  "is_proxy": false,
  "proxy_provider_names": [],
  "proxy_confidence_score": 0,
  "proxy_last_seen": "",
  "is_residential_proxy": false,
  "is_vpn": true,
  "vpn_provider_names": ["Browsec VPN"],
  "vpn_confidence_score": 80,
  "vpn_last_seen": "2026-06-25",
  "is_relay": false,
  "relay_provider_name": "",
  "is_anonymous": true,
  "is_known_attacker": true,
  "is_bot": false,
  "is_spam": true,
  "is_cloud_provider": true,
  "cloud_provider_name": "SIA Digitalas Ekonomikas Attistibas Centrs"
}
```

| Field | Why it matters |
|---|---|
| **`threat_score`** (0 to 100) | The most useful single number in the response. Treat `>= 70` as strong corroboration for any anonymization verdict. |
| **`is_vpn`**, **`is_proxy`**, **`is_residential_proxy`**, **`is_relay`**, **`is_tor`** | The anonymizer type, as five separate booleans instead of one label. This is how you split VPN handling from proxy handling, and `is_residential_proxy` is the one most worth its own rule, since residential proxies correlate with abuse far more than a commercial VPN does. |
| **`vpn_provider_names`**, **`proxy_provider_names`** | Arrays of named services, for example `["Browsec VPN"]`. Lets you allow a corporate VPN you recognize, block a provider you keep seeing in confirmed fraud, and tell a support agent exactly what the user is on. Empty when nothing is identified. |
| **`vpn_confidence_score`**, **`proxy_confidence_score`** | How sure the database is about that VPN or proxy classification, 0 to 100. Separate from the live `confidence` field in `live_vpn_proxy_detection`, which is about the live verdict. |
| `vpn_last_seen`, `proxy_last_seen` | Date the IP was last observed acting as a VPN or proxy, for example `2026-06-25`. A recent date makes the listing stronger evidence, an old one makes it weaker. Empty when never seen. |
| `relay_provider_name` | The named relay service when `is_relay` is `true`. Empty otherwise. |
| `is_known_attacker`, `is_spam`, `is_bot` | Hard evidence of past abuse. `is_known_attacker` is strong enough to act on by itself. |
| `is_cloud_provider`, `cloud_provider_name` | A hosting network rather than a consumer ISP. Very common for VPN exit nodes, and a poor fit for a genuine retail customer. |
| `is_anonymous` | The database view of the IP. |

```js
const sec = result.ip_security?.security;
sec.threat_score;           // 90
sec.is_vpn;                 // true
sec.vpn_provider_names;     // ["Browsec VPN"]
sec.vpn_confidence_score;   // 80
sec.vpn_last_seen;          // "2026-06-25"
```

**There are two `is_anonymous` fields, and the pair is useful.** `live_vpn_proxy_detection.is_anonymous` is the live view. `ip_security.security.is_anonymous` is the IP reputation database view. Both `true` gives you strong corroboration. Database only means the IP is listed but may not be anonymizing right now. Live only means anonymization no blocklist has caught yet, which is the residential proxy case and the whole reason this product exists.

### The complete response

```json
{
  "live_vpn_proxy_detection": {
    "is_anonymous": true,
    "confidence": 100,
    "proxy_score": 100,
    "vpn_score": 30
  },
  "ip_security": {
    "ip": "37.203.37.112",
    "location": {
      "continent_code": "EU",
      "continent_name": "Europe",
      "country_code2": "DE",
      "country_code3": "DEU",
      "country_name": "Germany",
      "state_prov": "Hesse",
      "state_code": "DE-HE",
      "district": "Frankfurt",
      "city": "Frankfurt",
      "zipcode": "60311",
      "latitude": "50.11090",
      "longitude": "8.68210",
      "is_eu": true,
      "country_flag": "https://ipgeolocation.io/static/flags/de_64.png",
      "geoname_id": "12218247",
      "country_emoji": "🇩🇪",
      "...": "..."
    },
    "country_metadata": { "calling_code": "+49", "tld": ".de", "languages": ["de"] },
    "network": {
      "connection_type": "",
      "route": "37.203.37.0/24",
      "is_anycast": false
    },
    "currency": { "code": "EUR", "name": "Euro", "symbol": "€" },
    "asn": {
      "as_number": "AS215373",
      "organization": "Sabiedriba ar ierobezotu atbildibu DELSKA Latvia",
      "country": "LV",
      "type": "ISP",
      "domain": "delska.com",
      "date_allocated": "2025-09-10",
      "rir": "RIPE"
    },
    "company": {
      "name": "SIA Digitalas Ekonomikas Attistibas Centrs",
      "type": "GOVERNMENT",
      "domain": "deac.eu"
    },
    "security": {
      "threat_score": 90,
      "is_tor": false,
      "is_proxy": false,
      "proxy_provider_names": [],
      "proxy_confidence_score": 0,
      "proxy_last_seen": "",
      "is_residential_proxy": false,
      "is_vpn": true,
      "vpn_provider_names": ["Browsec VPN"],
      "vpn_confidence_score": 80,
      "vpn_last_seen": "2026-06-25",
      "is_relay": false,
      "relay_provider_name": "",
      "is_anonymous": true,
      "is_known_attacker": true,
      "is_bot": false,
      "is_spam": true,
      "is_cloud_provider": true,
      "cloud_provider_name": "SIA Digitalas Ekonomikas Attistibas Centrs"
    },
    "time_zone": {
      "name": "Europe/Berlin",
      "offset": 1,
      "offset_with_dst": 2,
      "current_time": "2026-07-29 12:26:56.456+0200",
      "current_time_unix": 1785320816.456,
      "current_tz_abbreviation": "CEST",
      "is_dst": true,
      "dst_savings": 1,
      "dst_exists": true,
      "dst_start": { "utc_time": "2026-03-29 TIME 01:00", "duration": "+1.00H", "gap": true, "...": "..." },
      "dst_end": { "utc_time": "2026-10-25 TIME 01:00", "duration": "-1.00H", "gap": false, "...": "..." },
      "...": "..."
    }
  }
}
```

| Block | Key fields |
|---|---|
| `location` | Country, state, city, zip, latitude, longitude, `is_eu`, flag URL, emoji |
| `country_metadata` | `calling_code`, `tld`, `languages` |
| `network` | `connection_type`, `route`, `is_anycast` |
| `currency` | `code`, `name`, `symbol` |
| `asn` | `as_number`, `organization`, `country`, `type`, `domain`, `date_allocated`, `rir` |
| `company` | `name`, `type`, `domain` |
| `security` | The risk block. See above. |
| `time_zone` | `name`, `offset`, `current_time`, DST fields |

### Reading that example

Almost every signal points the same way, which makes it a good case to walk through.

`confidence: 100` puts it in the 81 and above band immediately. The `security` block is what turns that into a block rather than a challenge: `threat_score: 90`, a known attacker, spam history, a hosting network, and a named VPN endpoint (Browsec VPN, `vpn_confidence_score: 80`, last seen `2026-06-25`).

Two details worth noticing:

- **`location` says Frankfurt, Germany while `asn.country` is `LV`** and the AS belongs to a Latvian hosting company. A hosting ASN in one country geolocating to a datacenter city in another is a classic VPN exit node fingerprint, so do not treat Germany as where this user actually is.
- **The live scores and the database disagree on the type.** Live says `proxy_score: 100`, the database says `is_vpn: true` with `is_proxy: false`. That is not a contradiction, and you take the higher risk reading of the two.

> **Why a VPN service shows up as a proxy connection.** Browsec is sold as a VPN and listed as one, yet connecting through its browser extension makes our live tests see a proxy. That is normal, and you will meet it often.
>
> A browser extension cannot open an operating system level tunnel, because it cannot install a virtual network adapter, so the only hook it has is the browser's proxy configuration. That shows up in the protocol: extensions speak proxy protocols (HTTP or HTTPS `CONNECT`, SOCKS4 and SOCKS5), while desktop and mobile apps speak real VPN protocols (OpenVPN, IKEv2/IPsec, WireGuard). The two look nothing alike on the wire, so a live test separates them even within one brand.
>
> Providers ship extensions anyway because they install in one click with no admin rights or driver, work on locked down and ChromeOS machines, leave other applications untouched, and allow per site routing a full device tunnel cannot. "VPN" is what users search for, so the whole product line gets that name.
>
> So read the live scores for how the connection behaves, and the provider fields for who owns the endpoint. Neither is wrong.

Verdict: **block**, and log the payload so you can point at the specific reason if the user appeals.

**Signals that should push a decision harder:** `threat_score >= 70` escalates one band. `is_known_attacker`, `is_spam` or `is_bot` escalates to block or hard review. `is_cloud_provider: true` on a consumer flow means automation, so rate limit or challenge regardless of confidence. An IP country that does not match the billing country escalates one band on payment flows. One signal alone rarely justifies a block, but two or three pointing the same way usually do.

---


## 6. Do and don't

**Do:** run it on high value actions (signup, login, checkout, payout, promo redemption, referral claims), start monitoring on load and call `.get()` at the decision point, always gate on `confidence`, treat VPN and proxy as different risks, store payloads so you can tune your thresholds against real outcomes, and prefer a challenge over a hard block.

**Don't:** call it on every page view, hard block every VPN user (that burns real revenue), trust the client payload as your only authority, act on low confidence verdicts, or let a detection error break the user's flow.

**Privacy:** mention the fraud prevention processing in your privacy policy and consent notices, note it in your GDPR and CCPA records of processing, apply a retention limit to stored payloads, and give users a human review path for automated declines where the law requires one.

---

## 7. Common questions

**Do I need an API key in the frontend?** No. Authorization is by origin, so there is no secret to leak.

**Can I use this from a backend or a mobile app?** Not this script, it is browser only. Use the [IP Security API](https://ipgeolocation.io/ip-security-api.html) instead.

**How often should I call it?** Once per meaningful action, not once per page view.

**Does it detect residential proxies?** Yes, and that is the main reason it exists. Blocklists miss them because the IPs belong to ordinary consumer ISPs, while live analysis catches the anonymization itself.

**Is `confidence` a risk score?** No, it is certainty about `is_anonymous`. Risk lives in `proxy_score`, `vpn_score` and `threat_score`.

**Do `proxy_score` and `vpn_score` add up to 100?** No. They are independent likelihoods, so both can be high, and a 100 in one does not force a 0 in the other.

**Why is `proxy_score` 100 when the `security` block says `is_vpn: true` and `is_proxy: false`?** The live scores describe how the connection behaves right now, while `is_vpn` and `vpn_provider_names` name the service that owns the endpoint. One common cause is a browser VPN extension, which is a proxy at the transport level, but the same reading can come from a desktop or CLI client or a residential proxy. See the note in section 5.

**What is the difference between `confidence` and `vpn_confidence_score`?** `confidence` is certainty about the live `is_anonymous` verdict. `vpn_confidence_score` and `proxy_confidence_score` are how sure the IP reputation database is about its own VPN or proxy classification.

**Should I block every anonymized user?** Usually not. Many are ordinary privacy conscious or corporate users, which is what the bands in section 4 are for.

**What about Tor?** `ip_security.security.is_tor` flags known Tor exit nodes when `includeIPSecurity` is `true`.

**Will it slow down my page?** It is lightweight and promise based. Load it with `async` and never await a verdict during your initial render.

**Can users bypass it?** Client side code can always be tampered with, which is why the result should feed a server side decision alongside your own signals rather than being the decision.

---

**Related:** [IP Security API](https://ipgeolocation.io/ip-security-api.html) · [Guides and SDKs](https://github.com/IPGeolocation/ipgeolocation-guides) · [Full documentation](https://ipgeolocation.io/documentation) · <support@ipgeolocation.io>