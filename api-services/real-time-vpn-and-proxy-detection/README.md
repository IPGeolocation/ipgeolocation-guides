# Real-Time VPN and Proxy Detection

Detect VPNs, proxies and residential proxies live in the browser, at the moment a user acts. See what the service does and how it is priced on the [Real-Time VPN and Proxy Detection product page](https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html).

> **Note:** This is a lightweight client side script, much like a CAPTCHA widget. Because it runs live tests on the connection instead of just a database lookup, a verdict takes a few seconds. Start it early and ask for the result at your decision point, and never hold up your initial render waiting on it.

---

## Setup

The script runs in the browser and carries no API key. Requests are authorized by **origin** instead, so you have to register the domain you will call from as a [**Request Origin**](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use) in your IPGeolocation dashboard before your first call works.

1. [Sign up](https://app.ipgeolocation.io/signup) for your IPGeolocation account.
2. Add your origin, for example `https://app.example.com`.
3. Save.

> **Credits:** each call costs **3 credits** with `includeIPSecurity` set to `false`. Turning it on adds **2 credits**, for **5 credits** per call.

> **Availability:** this is a paid plan feature. A free trial is available, so [contact our support team](https://ipgeolocation.io/contact.html) to arrange one.

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

Response:

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

In plain words: this connection is not anonymized, we are certain about it, and the address the visitor is actually on is the same one we see.


---

## The address the request arrives from

Two fields sit at the top level of every response, outside both objects:

| Field | Meaning |
|---|---|
| `public_ip` | The address your server would see on the request. Behind a VPN or proxy this is the exit node, not the user. |
| `public_ip_country_code` | Two letter ISO country code for that address. Same caveat: behind an anonymizer this is where the exit node is, not where the user is. |

These two are the "apparent" side of the response. The `visitor_actual_location` object below is the "real" side, and comparing the two is the point.

---

## The live verdict and its four fields

Every response carries a `live_vpn_proxy_detection` object. It is the live verdict on the connection, four fields that are meant to be read together.

| Field | Range     | Meaning |
|---|-----------|---|
| `is_anonymous` | boolean   | The headline answer. `true` means the connection is anonymized, either through a VPN or through a proxy. It does not tell you which of the two. |
| `confidence_score` | up to 100 | How certain the service is about the `is_anonymous` value above. **This is not a risk score.** `confidence_score: 100` with `is_anonymous: false` means "confidently clean". A low value means the verdict is weak evidence either way. |
| `proxy_score` | 0 to 100  | Chance that the connection runs through a **proxy**, including residential and rotating proxies. |
| `vpn_score` | 0 to 100  | Chance that the connection runs through a **VPN**. |

The mental model: `is_anonymous` tells you whether the connection is hiding, `confidence_score` tells you how much to trust that yes or no, and `vpn_score` / `proxy_score` tell you what kind of hiding it looks like.

The two scores are independent likelihoods, not two halves of a split, so they do not add up to 100.

---

## Where the visitor actually is

`visitor_actual_location` is where the live tests earn their keep. Detecting that a connection is anonymized only tells you the exit node is not the user. This object tells you where the user is instead.

| Field | Range | Meaning |
|---|---|---|
| `actual_ip` | string | The visitor's real address behind the VPN or proxy, as seen by the live tests rather than by the request headers. When the connection is not anonymized, there is nothing hidden and this matches `public_ip`. |
| `actual_country_code` | string | Two letter ISO country code for `actual_ip`. This is the country to use for risk decisions, fraud rules and compliance checks. `public_ip_country_code` is the country the user is presenting. |
| `confidence_score` | up to 100 | How certain the service is about the actual location above. Same meaning as the `confidence_score` in `live_vpn_proxy_detection`, but scoped to this object: it is certainty, not risk. A low value means the real address was only partially recovered and should not carry a decision on its own. |

**The two country codes are the signal.** `public_ip_country_code` matching `actual_country_code` means the user is not misrepresenting where they are, even if they are on a VPN. A mismatch means the anonymizer is moving them across a border, which is what matters for geo restricted content, regional pricing, sanctions screening and billing country checks. Gate on `visitor_actual_location.confidence_score` before you act on a mismatch, in the same bands you use for the live verdict.

> **Note:** `actual_ip` is a stronger identifier than `public_ip`, since it survives the user switching VPN exit nodes. That makes it useful for rate limiting and for linking repeat abuse across sessions, and it also makes it personal data. Handle it under the same retention and disclosure rules as any other IP you store, and see the privacy note in [Dos and don'ts](#dos-and-donts).

---

## What to do at each confidence level

This is the part worth getting right. When `is_anonymous` is `true`, `confidence_score` decides how much friction the user deserves. Blocking everything the service flags will cost you real customers, and ignoring the flag defeats the point of running the check, so work in bands.

> **Note:** Every band in the following table assumes `is_anonymous` is `true`.

| Confidence | What it means | Recommended action |
|---|---|---|
| **Below 30** | Very weak signal. Expect false positives in this band. | **Allow.** Log the result for later analysis, add nothing to the user's path. Do not challenge on this alone. |
| **30 to 50** | Uncertain. Could easily be an unusual network, a mobile carrier or a corporate gateway. | **Allow and monitor.** Let the action through, tag the session as slightly elevated risk, feed it into your wider risk score. Add friction only if another signal agrees, such as a brand new account, a mismatched billing country, or an `actual_country_code` that does not match `public_ip_country_code`. |
| **51 to 80** | Likely anonymized. Enough for friction, not enough for a hard denial on its own. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. On payouts and first orders, hold for manual review instead of denying outright. |
| **81 and above** | Confidently anonymized. | **Block or restrict**, with the score type deciding how hard. High `proxy_score` usually means abuse: block, or hold the order and send it to review. High `vpn_score` deserves a hard challenge instead, since plenty of ordinary people browse through a VPN. Two caveats: a high `proxy_score` next to a named consumer VPN in `vpn_provider_names` is not automatically abuse, since it can come from a browser extension, a desktop or CLI client, or a residential proxy (see the [flagged example](#walking-through-the-flagged-example)), and an outright block is safest when a second signal backs it up, such as `threat_score >= 70`, `is_known_attacker` or `is_bot`. |

Tune these numbers against your own confirmed fraud data, because the right cutoff depends on your traffic and on how much a false positive costs you.


---

## Adding known IP reputation to the live verdict

Setting `includeIPSecurity` to `true` adds an `ip_security` object, resolved from our IP intelligence database in the same round trip. It is a flat block of reputation and risk signals for the address: the threat score, the anonymizer flags, and the name of the VPN or proxy provider behind the endpoint. The two halves answer different questions: `live_vpn_proxy_detection` describes how the connection behaves right now, while `ip_security` describes what is already known about the address it arrives from.

> **Note:** `ip_security` carries risk signals only. There is no geolocation, network, ASN, company, currency or timezone data inside it, and no nested `security` object either, so read the fields straight off `ip_security`. The only location the response gives you is country level, in `public_ip_country_code` and `visitor_actual_location.actual_country_code`. If you need city, region, network or ASN context for either address, look it up server side with the [IP Geolocation API](https://ipgeolocation.io/ip-location-api.html).

### What the enriched response looks like

A clean address, which is what most of your traffic will look like:

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
  },
  "ip_security": {
    "threat_score": 0,
    "is_tor": false,
    "is_proxy": false,
    "proxy_provider_names": [],
    "proxy_confidence_score": 0,
    "proxy_last_seen": "",
    "is_residential_proxy": false,
    "is_vpn": false,
    "vpn_provider_names": [],
    "vpn_confidence_score": 0,
    "vpn_last_seen": "",
    "is_relay": false,
    "relay_provider_name": "",
    "is_anonymous": false,
    "is_known_attacker": false,
    "is_bot": false,
    "is_spam": false,
    "is_cloud_provider": false,
    "cloud_provider_name": ""
  }
}
```

Every field is always present. Booleans default to `false`, scores to `0`, strings to `""` and provider lists to `[]`, so an unremarkable address gives you the payload above rather than missing keys. Check the values, not `hasOwnProperty`. Note that on a clean connection `visitor_actual_location.actual_ip` repeats `public_ip` rather than coming back empty, because there is no hidden address to recover.

And the same shape for an address that is flagged:

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
    "is_anonymous": true,
    "is_known_attacker": false,
    "is_bot": false,
    "is_spam": false,
    "is_cloud_provider": true,
    "cloud_provider_name": "UpCloud Ltd"
  }
}
```

### What each risk field is for

| Field | Why it matters |
|---|---|
| `threat_score` (0 to 100) | The most useful single number in the response. Treat `>= 70` as strong corroboration for any anonymization verdict. |
| `is_vpn`, `is_proxy`, `is_residential_proxy`, `is_relay`, `is_tor` | The anonymizer type, as five separate booleans instead of one label. This is how you split VPN handling from proxy handling, and `is_residential_proxy` is the one most worth its own rule, since residential proxies correlate with abuse far more than a commercial VPN does. |
| `vpn_provider_names`, `proxy_provider_names` | Arrays of named services, for example `["Browsec VPN"]`. Lets you allow a corporate VPN you recognize, block a provider you keep seeing in confirmed fraud, and tell a support agent exactly what the user is on. Empty when nothing is identified. |
| `vpn_confidence_score`, `proxy_confidence_score` | How sure the database is about that VPN or proxy classification, 0 to 100. Separate from the `confidence_score` fields in `live_vpn_proxy_detection` and `visitor_actual_location`, which are about the live verdict and the recovered address. |
| `vpn_last_seen`, `proxy_last_seen` | Date the IP was last observed acting as a VPN or proxy, for example `2026-08-07`. A recent date makes the listing stronger evidence, an old one makes it weaker. Empty when never seen. |
| `relay_provider_name` | The named relay service when `is_relay` is `true`. Empty otherwise. |
| `is_known_attacker`, `is_spam`, `is_bot` | Hard evidence of past abuse. `is_known_attacker` is strong enough to act on by itself. |
| `is_cloud_provider`, `cloud_provider_name` | A hosting network rather than a consumer ISP, named when known. Very common for VPN exit nodes, and a poor fit for a genuine retail customer. |
| `is_anonymous` | The database view of the IP. |

```js
const sec = result.ip_security;
sec.threat_score;           // 50
sec.is_vpn;                 // true
sec.vpn_provider_names;     // ["Browsec VPN"]
sec.vpn_confidence_score;   // 99
sec.vpn_last_seen;          // "2026-08-07"
sec.cloud_provider_name;    // "UpCloud Ltd"

const live = result.live_vpn_proxy_detection;
const real = result.visitor_actual_location;
live.confidence_score;      // 90
real.actual_country_code;   // "PK"
result.public_ip_country_code;               // "DE"
real.actual_country_code !== result.public_ip_country_code;   // true, the anonymizer crosses a border
```

**There are two `is_anonymous` fields, and the pair is useful.** `live_vpn_proxy_detection.is_anonymous` is the live view. `ip_security.is_anonymous` is the IP reputation database view. Both `true` gives you strong corroboration. Database only means the IP is listed but may not be anonymizing right now. Live only means anonymization that no blocklist has caught yet, which is the residential proxy case and the whole reason this product exists.

### Walking through the flagged example

The flagged payload is worth walking through, because the signals in it do not all carry the same weight.

`confidence_score: 90` on `is_anonymous: true` puts the live verdict in the 81 and above band. `ip_security` agrees on the fact of anonymization and is emphatic about it: a named VPN endpoint (Browsec VPN) with `vpn_confidence_score: 99` and `proxy_confidence_score: 99`, last seen as recently as `2026-08-07`, on a datacenter network belonging to UpCloud.

What it does not give you is a history of abuse. `threat_score: 50` is middling, and `is_known_attacker`, `is_spam` and `is_bot` are all `false`. This is a confidently anonymized address, not a known bad one.

Details worth noticing:

- **The two countries disagree.** `public_ip_country_code` is `DE`, `actual_country_code` is `PK` at `confidence_score: 80`. The user presents as German traffic while actually connecting from Pakistan. On a geo restricted or regionally priced flow that is the whole decision. On a payment flow, compare `actual_country_code` with the billing country rather than `public_ip_country_code`, which is only the exit node's.
- **`is_cloud_provider: true` with a hosting company in `cloud_provider_name`.** This address sits on a datacenter network, not a consumer ISP, which is exactly where VPN exit nodes live and where a genuine retail customer almost never does. Treat any location you look up for `public_ip` as the exit node's, and use `actual_ip` for the user's.
- **The live scores and the database emphasize different types.** Live leans proxy (`proxy_score: 90`, `vpn_score: 10`), the database flags both `is_vpn` and `is_proxy` at 99. That is not a contradiction, and you take the higher risk reading of the two.
- **`proxy_last_seen` is empty while `is_proxy` is `true`.** An empty date means no observation date is recorded, not that the classification is stale. Read it alongside `proxy_confidence_score`, which here is 99.

> **Why a VPN service shows up as a proxy connection.** Browsec is sold and listed as a VPN, yet its browser extension looks like a proxy to our live tests. That is normal and common: an extension cannot install a virtual network adapter, so its only hook is the browser's proxy configuration. Extensions therefore speak proxy protocols (HTTP or HTTPS `CONNECT`, SOCKS4, SOCKS5) while desktop and mobile apps speak VPN protocols (OpenVPN, IKEv2/IPsec, WireGuard), and a live test separates the two even within one brand.
>
> Read the live scores for how the connection behaves and the provider fields for who owns the endpoint. Neither is wrong.

Verdict: **challenge hard, and hold anything high value for review** rather than blocking outright. The anonymization is certain and the country mismatch is real, but nothing here says this address has attacked anyone. Block it on a flow where the mismatch itself is disqualifying, such as licensed content or a sanctions check. Log the payload either way, so you can point at the specific reason if the user appeals. Had `threat_score` been 90 with `is_known_attacker: true` and `is_spam: true`, as an abusive endpoint typically looks, a straight block would be the right call.

**Signals that should push a decision harder:** `threat_score >= 70` escalates one band. `is_known_attacker`, `is_spam` or `is_bot` escalates to block or hard review. `is_cloud_provider: true` on a consumer flow means automation, so rate limit or challenge regardless of confidence. `actual_country_code` not matching `public_ip_country_code` escalates one band, and on payment flows so does `actual_country_code` not matching the billing country, provided `visitor_actual_location.confidence_score` is high enough to lean on. One signal alone rarely justifies a block, but two or three pointing the same way usually do.

---


## Dos and don'ts

**Do:** run it on high value actions (signup, login, checkout, payout, promo redemption, referral claims), start monitoring on load and call `.get()` at the decision point, always gate on `confidence_score`, use `actual_country_code` rather than `public_ip_country_code` wherever the user's real country is what matters, treat VPN and proxy as different risks, store payloads so you can tune your thresholds against real outcomes, and prefer a challenge over a hard block.

**Don't:** call it on every page view, hard block every VPN user (that burns real revenue), trust the client payload as your only authority, act on low confidence verdicts, treat a country mismatch as fraud on its own, or let a detection error break the user's flow.

**Privacy:** mention the fraud prevention processing in your privacy policy and consent notices, note it in your GDPR and CCPA records of processing, apply a retention limit to stored payloads, and give users a human review path for automated declines where the law requires one. `visitor_actual_location.actual_ip` deserves particular care: it is personal data that the user was actively trying not to disclose, so store it only where you have a fraud prevention or compliance basis for it, keep it out of logs and analytics that do not need it, and expect to explain it in a subject access request.

---

## FAQs

<details>
<summary><strong>Do I need an API key in the frontend?</strong></summary>
No. Authorization is by origin, so there is no secret to leak.
</details>

<details>
<summary><strong>Can I use this from a backend or a mobile app?</strong></summary>
Not this script, it is browser only. Use the <a href="https://ipgeolocation.io/ip-security-api.html">IP Security API</a> instead.
</details>

<details>
<summary><strong>How often should I call it?</strong></summary>
Once per meaningful action, not once per page view.
</details>

<details>
<summary><strong>Does it detect residential proxies?</strong></summary>
Yes, and that is the main reason it exists. Blocklists miss them because the IPs belong to ordinary consumer ISPs, while live analysis catches the anonymization itself.
</details>

<details>
<summary><strong>Is confidence_score a risk score?</strong></summary>
No, it is certainty about <code>is_anonymous</code>. Risk lives in <code>vpn_score</code>, <code>proxy_score</code> and <code>threat_score</code>.
</details>

<details>
<summary><strong>What is visitor_actual_location?</strong></summary>
The visitor's real IP and country behind the VPN or proxy, with its own certainty score. <code>public_ip</code> is where the request appears to come from, <code>actual_ip</code> is where the user actually is. See <a href="#where-the-visitor-actually-is">Where the visitor actually is</a>.
</details>

<details>
<summary><strong>Which country code should I use for my geo rules?</strong></summary>
<code>visitor_actual_location.actual_country_code</code>, gated on its <code>confidence_score</code>. <code>public_ip_country_code</code> is the exit node's country, which is exactly the value a user on a VPN is choosing.
</details>

<details>
<summary><strong>What does visitor_actual_location return when the user is not on a VPN?</strong></summary>
<code>actual_ip</code> and <code>actual_country_code</code> match <code>public_ip</code> and <code>public_ip_country_code</code>, because there is no hidden address to recover.
</details>

<details>
<summary><strong>Do vpn_score and proxy_score add up to 100?</strong></summary>
No. They are independent likelihoods, so both can be high, and a 100 in one does not force a 0 in the other.
</details>

<details>
<summary><strong>Why is proxy_score 100 when ip_security says is_vpn true and is_proxy false?</strong></summary>
The live scores describe how the connection behaves right now, while <code>is_vpn</code> and <code>vpn_provider_names</code> name the service that owns the endpoint. One common cause is a browser VPN extension, which is a proxy at the transport level, but the same reading can come from a desktop or CLI client. See <a href="#walking-through-the-flagged-example">the walkthrough of the flagged example</a> for the full explanation.
</details>

<details>
<summary><strong>What is the difference between the confidence_score fields?</strong></summary>
<code>live_vpn_proxy_detection.confidence_score</code> is certainty about the live <code>is_anonymous</code> verdict. <code>visitor_actual_location.confidence_score</code> is certainty about the recovered real address. <code>vpn_confidence_score</code> and <code>proxy_confidence_score</code> are how sure the IP reputation database is about its own VPN or proxy classification. All four are certainty, none of them is risk.
</details>

<details>
<summary><strong>Should I block every anonymized user?</strong></summary>
Usually not. Many are ordinary privacy conscious or corporate users, which is what <a href="#what-to-do-at-each-confidence-level">the confidence bands</a> are for.
</details>

<details>
<summary><strong>What about Tor?</strong></summary>
<code>ip_security.is_tor</code> flags known Tor exit nodes when <code>includeIPSecurity</code> is <code>true</code>.
</details>

<details>
<summary><strong>Will it slow down my page?</strong></summary>
It is lightweight and promise based. Load it with <code>async</code> and never await a verdict during your initial render.
</details>

<details>
<summary><strong>Can users bypass it?</strong></summary>
Client side code can always be tampered with, which is why the result should feed a server side decision alongside your own signals rather than being the decision.
</details>

---
