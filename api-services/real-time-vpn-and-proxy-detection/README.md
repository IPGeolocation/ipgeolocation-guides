# Real-Time VPN and Proxy Detection

Detect VPNs, proxies and residential proxies live in the browser, at the moment a user acts. See what the service does and how it is priced on the [Real-Time VPN and Proxy Detection product page](https://ipgeolocation.io/real-time-proxy-and-vpn-detection.html).

> **Note:** This is a lightweight client side script, much like a CAPTCHA widget. Because it runs live tests on the connection instead of just database lookup, a verdict takes a few seconds. Start it early and ask for the result at your decision point, and never hold up your initial render waiting on it.

---

## Setup

The script runs in the browser and carries no API key. Requests are authorized by **origin** instead, so you have to [register the domain you will call from as a **Request Origin** in your IPGeolocation dashboard](https://ipgeolocation.io/tutorials/secure-api-key-before-production#set-up-request-origin-cors-for-client-side-use) before your first call works.

1. [Sign in to your IPGeolocation account](https://ipgeolocation.io).
2. Add your origin, for example `https://app.example.com`.
3. Save.

> **Credits:** each call costs **3 credits** with `includeIPSecurity` set to `false`. Turning it on adds **2 credits**, for **5 credits** per call.

> **Availability:** this is a paid plan feature. A free trial is available, so [contact our support team](mailto:support@ipgeolocation.io) to arrange one.

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

## The live verdict and its four fields

Every response carries a `live_vpn_proxy_detection` object. It is the live verdict on the connection, four fields that are meant to be read together.

| Field | Range     | Meaning |
|---|-----------|---|
| `is_anonymous` | boolean   | The headline answer. `true` means the connection is anonymized, either through a VPN or through a proxy. It does not tell you which of the two. |
| `confidence` | up to 100 | How certain the service is about the `is_anonymous` value above. **This is not a risk score.** `confidence: 100` with `is_anonymous: false` means "confidently clean". A low value means the verdict is weak evidence either way. |
| `proxy_score` | 0 to 100  | Chance that the connection runs through a **proxy**, including residential and rotating proxies. |
| `vpn_score` | 0 to 100  | Chance that the connection runs through a **VPN**. |

The mental model: `is_anonymous` tells you whether the connection is hiding, `confidence` tells you how much to trust that yes or no, and `vpn_score` / `proxy_score` tell you what kind of hiding it looks like.

The two scores are independent likelihoods, not two halves of a split, so they do not add up to 100.

---

## What to do at each confidence level

This is the part worth getting right. When `is_anonymous` is `true`, `confidence` decides how much friction the user deserves. Blocking everything the service flags will cost you real customers, and ignoring the flag defeats the point of running the check, so work in bands.

> **Note:** Every band in the following table assumes `is_anonymous` is `true`.

| Confidence | What it means | Recommended action |
|---|---|---|
| **Below 30** | Very weak signal. Expect false positives in this band. | **Allow.** Log the result for later analysis, add nothing to the user's path. Do not challenge on this alone. |
| **30 to 50** | Uncertain. Could easily be an unusual network, a mobile carrier or a corporate gateway. | **Allow and monitor.** Let the action through, tag the session as slightly elevated risk, feed it into your wider risk score. Add friction only if another signal agrees, such as a brand new account or a mismatched billing country. |
| **51 to 80** | Likely anonymized. Enough for friction, not enough for a hard denial on its own. | **Challenge.** MFA, email or SMS verification, or a CAPTCHA. On payouts and first orders, hold for manual review instead of denying outright. |
| **81 and above** | Confidently anonymized. | **Block or restrict**, with the score type deciding how hard. High `proxy_score` usually means abuse: block, or hold the order and send it to review. High `vpn_score` deserves a hard challenge instead, since plenty of ordinary people browse through a VPN. Two caveats: a high `proxy_score` next to a named consumer VPN in `vpn_provider_names` is not automatically abuse, since it can come from a browser extension, a desktop or CLI client, or a residential proxy (see [why a VPN service can show up as a proxy connection](#walking-through-the-flagged-example)), and an outright block is safest when a second signal backs it up, such as `threat_score >= 70`, `is_known_attacker` or `is_bot`. |

Tune these numbers against your own confirmed fraud data, because the right cutoff depends on your traffic and on how much a false positive costs you.


---

## Adding known IP reputation to the live verdict

Setting `includeIPSecurity` to `true` adds an `ip_security` object, resolved from our IP intelligence database in the same round trip. It is a flat block of reputation and risk signals for the address: the threat score, the anonymizer flags, and the name of the VPN or proxy provider behind the endpoint. The two halves answer different questions: `live_vpn_proxy_detection` describes how the connection behaves right now, while `ip_security` describes what is already known about the address it arrives from.

> **Note:** `ip_security` carries risk signals only. There is no geolocation, network, ASN, company, currency or timezone data in this response, and no nested `security` object either, so read the fields straight off `ip_security`. If you need location or network context for the same address, look it up server side with the [IP Geolocation API](https://ipgeolocation.io/ip-location-api.html).

### What the enriched response looks like

A clean address, which is what most of your traffic will look like:

```json
{
  "live_vpn_proxy_detection": {
    "is_anonymous": false,
    "confidence": 100,
    "proxy_score": 0,
    "vpn_score": 0
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

Every field is always present. Booleans default to `false`, scores to `0`, strings to `""` and provider lists to `[]`, so an unremarkable address gives you the payload above rather than missing keys. Check the values, not `hasOwnProperty`.

And the same shape for an address that is flagged:

```json
{
  "live_vpn_proxy_detection": {
    "is_anonymous": true,
    "confidence": 100,
    "proxy_score": 100,
    "vpn_score": 30
  },
  "ip_security": {
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
}
```

### What each risk field is for

| Field | Why it matters |
|---|---|
| `threat_score` (0 to 100) | The most useful single number in the response. Treat `>= 70` as strong corroboration for any anonymization verdict. |
| `is_vpn`, `is_proxy`, `is_residential_proxy`, `is_relay`, `is_tor` | The anonymizer type, as five separate booleans instead of one label. This is how you split VPN handling from proxy handling, and `is_residential_proxy` is the one most worth its own rule, since residential proxies correlate with abuse far more than a commercial VPN does. |
| `vpn_provider_names`, `proxy_provider_names` | Arrays of named services, for example `["Browsec VPN"]`. Lets you allow a corporate VPN you recognize, block a provider you keep seeing in confirmed fraud, and tell a support agent exactly what the user is on. Empty when nothing is identified. |
| `vpn_confidence_score`, `proxy_confidence_score` | How sure the database is about that VPN or proxy classification, 0 to 100. Separate from the live `confidence` field in `live_vpn_proxy_detection`, which is about the live verdict. |
| `vpn_last_seen`, `proxy_last_seen` | Date the IP was last observed acting as a VPN or proxy, for example `2026-06-25`. A recent date makes the listing stronger evidence, an old one makes it weaker. Empty when never seen. |
| `relay_provider_name` | The named relay service when `is_relay` is `true`. Empty otherwise. |
| `is_known_attacker`, `is_spam`, `is_bot` | Hard evidence of past abuse. `is_known_attacker` is strong enough to act on by itself. |
| `is_cloud_provider`, `cloud_provider_name` | A hosting network rather than a consumer ISP, named when known. Very common for VPN exit nodes, and a poor fit for a genuine retail customer. |
| `is_anonymous` | The database view of the IP. |

```js
const sec = result.ip_security;
sec.threat_score;           // 90
sec.is_vpn;                 // true
sec.vpn_provider_names;     // ["Browsec VPN"]
sec.vpn_confidence_score;   // 80
sec.vpn_last_seen;          // "2026-06-25"
sec.cloud_provider_name;    // "SIA Digitalas Ekonomikas Attistibas Centrs"
```

**There are two `is_anonymous` fields, and the pair is useful.** `live_vpn_proxy_detection.is_anonymous` is the live view. `ip_security.is_anonymous` is the IP reputation database view. Both `true` gives you strong corroboration. Database only means the IP is listed but may not be anonymizing right now. Live only means anonymization no blocklist has caught yet, which is the residential proxy case and the whole reason this product exists.

### Walking through the flagged example

Almost every signal in the flagged payload points the same way, which makes it a good case to walk through.

`confidence: 100` puts it in the 81 and above band immediately. `ip_security` is what turns that into a block rather than a challenge: `threat_score: 90`, a known attacker, spam history, and a named VPN endpoint (Browsec VPN, `vpn_confidence_score: 80`, last seen `2026-06-25`).

Two details worth noticing:

- **`is_cloud_provider: true` with a hosting company in `cloud_provider_name`.** This address sits on a datacenter network, not a consumer ISP, which is exactly where VPN exit nodes live and where a genuine retail customer almost never does. Treat any location you have for this IP as the exit node's, not the user's.
- **The live scores and the database disagree on the type.** Live says `proxy_score: 100`, the database says `is_vpn: true` with `is_proxy: false`. That is not a contradiction, and you take the higher risk reading of the two.

> **Why a VPN service shows up as a proxy connection.** Browsec is sold and listed as a VPN, yet its browser extension looks like a proxy to our live tests. That is normal and common: an extension cannot install a virtual network adapter, so its only hook is the browser's proxy configuration. Extensions therefore speak proxy protocols (HTTP or HTTPS `CONNECT`, SOCKS4, SOCKS5) while desktop and mobile apps speak VPN protocols (OpenVPN, IKEv2/IPsec, WireGuard), and a live test separates the two even within one brand.
>
> Read the live scores for how the connection behaves and the provider fields for who owns the endpoint. Neither is wrong.

Verdict: **block**, and log the payload so you can point at the specific reason if the user appeals.

**Signals that should push a decision harder:** `threat_score >= 70` escalates one band. `is_known_attacker`, `is_spam` or `is_bot` escalates to block or hard review. `is_cloud_provider: true` on a consumer flow means automation, so rate limit or challenge regardless of confidence. An IP country that does not match the billing country escalates one band on payment flows, though that country now has to come from your own geolocation lookup rather than from this response. One signal alone rarely justifies a block, but two or three pointing the same way usually do.

---


## Dos and don'ts

**Do:** run it on high value actions (signup, login, checkout, payout, promo redemption, referral claims), start monitoring on load and call `.get()` at the decision point, always gate on `confidence`, treat VPN and proxy as different risks, store payloads so you can tune your thresholds against real outcomes, and prefer a challenge over a hard block.

**Don't:** call it on every page view, hard block every VPN user (that burns real revenue), trust the client payload as your only authority, act on low confidence verdicts, or let a detection error break the user's flow.

**Privacy:** mention the fraud prevention processing in your privacy policy and consent notices, note it in your GDPR and CCPA records of processing, apply a retention limit to stored payloads, and give users a human review path for automated declines where the law requires one.

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
<summary><strong>Is confidence a risk score?</strong></summary>
No, it is certainty about <code>is_anonymous</code>. Risk lives in <code>vpn_score</code>, <code>proxy_score</code> and <code>threat_score</code>.
</details>

<details>
<summary><strong>Do vpn_score and proxy_score add up to 100?</strong></summary>
No. They are independent likelihoods, so both can be high, and a 100 in one does not force a 0 in the other.
</details>

<details>
<summary><strong>Why is proxy_score 100 when ip_security says is_vpn true and is_proxy false?</strong></summary>
The live scores describe how the connection behaves right now, while <code>is_vpn</code> and <code>vpn_provider_names</code> name the service that owns the endpoint. One common cause is a browser VPN extension, which is a proxy at the transport level, but the same reading can come from a desktop or CLI client See <a href="#walking-through-the-flagged-example">the walkthrough of the flagged example</a> for the full explanation.
</details>

<details>
<summary><strong>What is the difference between confidence and vpn_confidence_score?</strong></summary>
<code>confidence</code> is certainty about the live <code>is_anonymous</code> verdict. <code>vpn_confidence_score</code> and <code>proxy_confidence_score</code> are how sure the IP reputation database is about its own VPN or proxy classification.
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

