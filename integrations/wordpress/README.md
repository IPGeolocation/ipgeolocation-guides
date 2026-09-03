# IPGeolocation.io - Geo Redirects & Content Control
## Overview
A plugin that decides what each visitor sees based on their IP address and their location. Redirect people by country, block unwanted IP addresses, and show or hide parts of a page depending on where the visitor is.

Everything is configured from one settings screen. No code required.

[Plugin on WordPress.org](https://wordpress.org/plugins/ipgeolocation-geo-redirects-content-control/)

## At a glance

| What you want to do | Feature | Requires API key |
| --- | --- | --- |
| Send visitors from one country to a different page | [Country redirect rules](#country-redirect-rules) | Yes |
| Close the site to some countries, or open it to only a few | [Country access control](#country-access-control) | Yes |
| Block a scraper, a spammer or a whole IP range | [IP access control](#block-visitors-by-ip-address) | No |
| Lock your login page to your own IP addresses | [Login rule](#the-two-questions) | No |
| Change a banner, price or notice by location | [Conditional shortcodes](#show-or-hide-content) | Yes |
| Print the visitor's city, country or currency | [Display shortcodes](#show-a-single-value) | Yes |
| Keep checkout and cart out of every country rule | [Page exclusions](#page-exclusions) | No |

- IP rules are a local check. They cost no API credits and work before you have a key.
- Country lookups are cached for 24 hours per IP address, so repeat visitors cost nothing.
- Logged-in administrators and known search engine bots are never redirected.
- New here? Do [Setup](#setup) first, then pick a recipe from [Common Use Cases](#common-use-cases).

## What's new in 1.2.0

**IP access control.** Block or allow visitors by IP address, with separate rules for your website and your login page. IP rules run before any location lookup, so they use no API credits and work without an API key.

**IP spoofing fix.** Visitor IP addresses can no longer be faked using request headers. If your site sits behind a CDN or reverse proxy, open the IP section after updating and confirm the IP address shown at the top is actually yours.

The full [changelog](#changelog) is at the bottom of this page.

## Requirements

- WordPress 5.8 or newer
- PHP 7.4 or newer
- An ipgeolocation.io API key for anything country-based. IP blocking works without one.

## Setup

### Get an API key

Sign up at [ipgeolocation.io](https://app.ipgeolocation.io/sign-up), then copy the API key from your account dashboard.

### Install the plugin

In your WordPress dashboard go to **Plugins → Add New**, search for **IPGeolocation.io Geo Redirects & Content Control**, then click **Install Now** and **Activate**.

![Add the plugin](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/add-plugin-1.1.0.png)

### Enter your key and plan

Paste the key into the plugin settings and select your plan type: **Developer** (free) or **Paid**.

![API configuration](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/api-config-1.1.0.png)

> [!IMPORTANT]
> Please make sure to select the correct plan type. Choosing the wrong option may cause the plugin to not work properly.

## Common use cases

Each of these takes about a minute once your API key is in place.

### Send shoppers to their regional store

Create one [country redirect rule](#country-redirect-rules) per market. Set **Country code** to `CA`, **Apply to** to Entire site, **Redirect URL** to `/ca`, **Redirect type** to 302, and **Redirect method** to Popup. Then exclude `/checkout` and `/cart` under [Page exclusions](#page-exclusions) so nobody is bounced mid purchase.

### Stop brute force login attempts

Under [question 2](#the-two-questions), choose **Only the addresses I list** and enter your office or home IP address. Password guessing bots never reach the login form, and your public pages stay open to everyone. If your IP address changes often, use **Anyone except the addresses I list** and block the offenders instead.

### Block a scraper or a spam source

Under [question 1](#the-two-questions), choose **Anyone except the addresses I list** and paste the offending IP address or range, one per line. Blocked requests stop before any location lookup, so a hostile crawler costs you nothing in API credits.

### Keep a staging or unfinished site private

Under [question 1](#the-two-questions), choose **Only the addresses I list** and add your team's IP addresses. Everyone else lands on the page you nominate, or gets a 403.

### Restrict a product or service by country

Use [Country access control](#country-access-control) in **Block mode**, list the country codes, and point the redirect URL at a page that explains why. This is the usual choice for licensing and regulatory restrictions, because it covers the whole site in one rule.

### Show a shipping notice only where it applies

Drop `[ipgeo_if country_code="US"]Free shipping across the US.[/ipgeo_if]` into a header or widget. No redirect and no duplicate page, so search engines still crawl one version of your content.

## How the plugin decides

Each request runs through these checks in order. The first match ends the request.

| Order | Check | Notes |
| --- | --- | --- |
| 1 | Exempt requests | WP-CLI, cron, installs and the recovery constant are never touched |
| 2 | Safe list | Skips every rule, both IP and country |
| 3 | IP rules | Local check, no API call |
| 4 | Page exclusions | Listed pages skip the country rules |
| 5 | Country access control | Allow or block by country |
| 6 | Country redirect rules | Automatic or popup |

Logged-in administrators and known search engine bots are excluded from all country rules. IP rules apply to everyone, including bots.

## Block visitors by IP address

This section is a purely local check. No API key, no credits, no lookup.

![IP access control](https://ps.w.org/ipgeolocation-geo-redirects-content-control/assets/screenshot-5.png)

### Your IP address

The top of the section shows the IP address your server currently sees for you, with a button that adds it to your safe list. If that IP address is not yours, your site is behind a proxy or CDN. Fix that under **Advanced settings** before writing any rules.

### Safe list

IP addresses listed here are never blocked, whatever the rules below say. They skip your country rules too. Add your own IP address first.

### The two questions

**1. Who can visit your website?** Covers your public pages.

**2. Who can reach your login page?** Covers `wp-login.php` and the WordPress dashboard. This is separate from question 1, so you can lock the login page while your site stays open to visitors.

Each question has the same three answers:

| Answer | Result |
| --- | --- |
| Anyone | No restriction. This is the default. |
| Anyone except the addresses I list | Every IP address on the list is blocked. |
| Only the addresses I list | Every IP address not on the list is blocked. |

An empty list blocks nobody, in either mode. That stops a half-finished allow list from taking your site down.

Locking the login page to your office IP address is the strongest option here. Password guessing bots never reach the form, and ordinary visitors are unaffected.

### IP address formats

Every IP address box accepts one entry per line, in any of these formats:

| Format | Meaning |
| --- | --- |
| `203.0.113.9` | One single IP address |
| `203.0.113.0/24` | A whole block, 203.0.113.0 up to 203.0.113.255 |
| `192.0.2.*` | Anything starting with 192.0.2. |
| `198.51.100.10-198.51.100.50` | Everything between those two IP addresses |
| `2001:db8::1` | IPv6 works the same way |
| `# our office` | A note for you, ignored by the plugin |

Lines that cannot be read are skipped and reported back to you when you save.

### What a blocked visitor sees

Set **Send blocked people to this page** to a path on your site, for example `/access-denied`. Leave it empty and they get a plain 403 page instead.

Requests to the REST API, `admin-ajax.php` and `xmlrpc.php` always receive a 403 status rather than a redirect, so scripts get a clear answer.

### Lockout protection

The plugin refuses to save a login rule that would block your own IP address, and tells you what that IP address is so you can add it and save again. To save the rule anyway, tick the confirmation box.

If you do get locked out, add this line to `wp-config.php`, log in, fix the rule, then remove the line:

```php
define( 'IPGEO_DISABLE_IP_ACCESS', true );
```

### Advanced settings

**How your visitor's IP address is read**

Automatic suits most sites. The other options are direct (no proxy), Cloudflare, and a custom proxy whose ranges you supply. Only change this if the detected IP address is wrong, and check it again after saving.

Cloudflare headers are trusted only when the request genuinely came from a Cloudflare server. The plugin refreshes Cloudflare's IP address ranges once a day and falls back to a bundled copy if that request fails.

**Should logged-in users be blocked?**

Choose from: never block administrators (default, and your safety net), never block anyone who is logged in, or block by IP address regardless.

**Also apply the login rule to**

Two optional extras. Leave `admin-ajax.php` off unless you are sure, because many contact forms and shop features use it. `xmlrpc.php` is safe to turn on unless you use the WordPress mobile app or Jetpack.

**If an IP address cannot be read at all, block the request**

Off by default, so an unusual request is let through rather than turning a real visitor away.

## Country redirect rules

Send visitors from a given country to a different URL.

![Country redirect rules](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/redirect-rules.png)

| Field | Description |
| --- | --- |
| Country code | Two letter ISO code, for example `US`, `GB`, `CA`. One code per rule. |
| Apply to | Entire site, one specific page, or a URL pattern such as `/shop/*` |
| Redirect URL | Relative (`/uk-store`) |
| Redirect type | 301 permanent, indexed by search engines, or 302 temporary for campaigns and testing |
| Redirect method | Automatic or popup |

### Popup confirmation

A popup asks before redirecting. **Yes** sends the visitor on, **No** keeps them on the current page. You can edit the message, both button labels, and the text and background colours, with a live preview beside the fields.

![Popup appearance](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/popup-appearance.png)

Use a popup when a visitor might reasonably want to decline, such as a currency or storefront switch. Use automatic for country-specific pages where staying put makes no sense.

## Country access control

Allow or block whole countries.

![Country access control](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/country-access-control.png)

* **Block mode**: visitors from the listed countries are blocked
* **Allow mode**: only visitors from the listed countries get in

Country codes are two letters and case insensitive. The redirect URL must be relative, for example `/not-available`, and you can design that page however you like. Leave it empty and blocked visitors get a 403 page.

This runs on front-end requests only. The admin area, logged-in administrators, search engine bots, REST and ajax requests are all skipped. If the country cannot be determined, no restriction is applied.

## Page exclusions

Excluded pages skip the country redirect rules. Checkout and cart pages are the usual candidates.

![Page exclusion rules](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/page-exclusion-rules.png)

| Type | Example value | Matches |
| --- | --- | --- |
| Page URL Equals | `/checkout` | `/checkout` only, not `/checkout/cart` |
| Page URL Contains | `/blog` | `/blog` and `/blog/post-name` |
| Page Query Contains | `ref=google` | `/shop?ref=google` |

One match is enough to exclude a page. Trailing slashes are handled for you.

> [!TIP]
> On a WooCommerce site, exclude `/checkout`, `/cart` and `/my-account` before you switch on your first redirect rule. Use **Page URL Contains** for these so the endpoints underneath them, such as `/my-account/orders`, are covered by the same entry.

## Shortcodes

### Show a single value

`[ipgeo field]` prints one piece of information about the current visitor.

```bash
[ipgeo country]       United States
[ipgeo city]          New York
[ipgeo state]         California
[ipgeo country_code]  US
[ipgeo currency]      USD
[ipgeo calling_code]  +1
```

Available fields: `ip`, `city`, `state`, `country`, `country_code`, `zipcode`, `continent`, `latitude`, `longitude`, `currency`, `calling_code`, `languages`.

Paid plans add: `is_proxy`, `is_tor`, `is_anonymous`, `cloud_provider`.

> [!TIP]
> If you only need to change a banner, a price or a notice, reach for a shortcode instead of a redirect. The visitor stays on one URL, search engines index one version of the page, and nothing is broken if the location lookup fails.

### Show or hide content

`[ipgeo_if]` shows the content when the conditions match. `[ipgeo_if_not]` hides it when they match. Both accept the same attributes.

```bash
[ipgeo_if country_code="US,CA" logic="OR"]
Free shipping across the US and Canada.
[/ipgeo_if]

[ipgeo_if_not country="Germany"]
Hidden from visitors in Germany.
[/ipgeo_if_not]

[ipgeo_if country_code="US" state="California" logic="AND"]
Welcome, California visitors. Check out our local deals.
[/ipgeo_if]
```

Attributes: `country`, `country_code`, `state`, `city`, `continent`, `is_proxy`, `is_tor`, `is_cloud_provider`, `is_anonymous`.

Use `logic="AND"` (the default) to require every condition, or `logic="OR"` to accept any one of them. Within a single attribute, comma separated values are always treated as "any of these".

## Caching, CDNs and performance

API results are cached for 24 hours per IP address in WordPress transients, and IP rules are a local check with no lookup involved, so the impact on page load is minimal.

Uncached requests and logged-in visitors are handled normally. A page already saved as a static cache file may still be served, because that happens before this plugin runs. In your caching plugin, exclude any page that must always be checked. Blocked responses are marked as not cacheable.

> [!IMPORTANT]
> Full page caching and location shortcodes do not mix. If a cached page contains `[ipgeo ...]` or `[ipgeo_if ...]`, every visitor is served whatever the first visitor saw. Exclude those pages in your caching plugin, or keep the location-specific part in a section your cache leaves alone.

## Troubleshooting


### Nothing is being redirected

Work through these in order:

1. You are logged in as an administrator. Admins are never redirected, so use a private window.
2. Your API key is missing, or the selected plan type does not match your account.
3. The page is covered by a [page exclusion rule](#page-exclusions).

### The page keeps reloading

In 1.1.0 an empty redirect URL in Country Access Control caused this, so update to 1.2.0 first. If it continues, your redirect target is itself covered by the rule that sends people there. Point the rule at a page outside its own scope, or add the target to Page exclusions.

### Everyone is blocked after I saved an IP rule

Check that the IP address shown at the top of the IP section is really yours. If it is not, your site is behind a proxy or CDN and every visitor looks like the same IP address, so one rule catches all of them. Fix the detection mode under Advanced settings. To get back in meanwhile, add `define( 'IPGEO_DISABLE_IP_ACCESS', true );` to `wp-config.php`.

### My contact form or checkout stopped working

Turn off `admin-ajax.php` under **Also apply the login rule to**. Many forms, shop features and page builders use that file, so applying a login rule to it blocks ordinary visitors.

### Search engine crawlers are being blocked

Bots are excluded from country rules but not from IP rules, because any browser can claim to be a crawler. If an IP rule is catching a crawler you want, add its IP ranges to the safe list.

### A shortcode prints nothing

`[ipgeo ...]` returns an empty string when no location data is available: a missing key, an invalid key, or a field your plan does not include. `is_proxy`, `is_tor`, `is_anonymous` and `cloud_provider` need a paid plan.

## For developers

| Hook | Type | Use |
| --- | --- | --- |
| `ipgeo_ip_access_bypass` | filter | Skip the IP gate entirely for a request |
| `ipgeo_ip_access_blocked` | filter | Override the block decision |
| `ipgeo_ip_access_denied` | action | Fires before a request is denied, useful for logging |
| `IPGEO_DISABLE_IP_ACCESS` | constant | Turns off IP rules, for lockout recovery |

## Changelog

### 1.2.0

- Added IP access control: block or allow visitors by IP address, with separate rules for your website and your login page
- IP address lists accept single IP addresses, ranges such as `203.0.113.0/24`, wildcards such as `192.0.2.*`, and IPv6
- Added a safe list of IP addresses that are never blocked by any rule
- The settings screen now shows your own IP address and how it was detected
- The plugin refuses to save a login rule that would lock you out, with a `wp-config.php` recovery option
- Security: visitor IP addresses can no longer be faked using request headers
- Fixed an empty redirect URL in Country Access Control causing an endless redirect loop
- Fixed blocked visitors being redirected twice
- Fixed cached location data not being reused within the same page load

### 1.1.0

- Upgraded internal API from v2 to v3
- Simplified plan types to Developer and Paid only, with Standard, Advanced and Security auto-migrated to Paid
- Renamed the plugin

### 1.0.0

- Initial public release with country redirects, popup support, country allow and block rules, conditional shortcodes, bot detection and caching

## Frequently asked questions

<details>
<summary><strong>Do I need an API key to block IP addresses?</strong></summary>
No. Blocking by IP address is a local check. Only the country features need a key.
</details>

<details>
<summary><strong>The plugin shows the wrong IP address for me. Why?</strong></summary>
Your site is behind a CDN or proxy, which replaces the IP address your server sees. Open Advanced settings, choose the option that matches your setup, save, then check the IP address again.
</details>

<details>
<summary><strong>I locked myself out of my login page. How do I get back in?</strong></summary>
Add <code>define( 'IPGEO_DISABLE_IP_ACCESS', true );</code> to your <code>wp-config.php</code>, log in, fix the rule, then remove the line.
</details>

<details>
<summary><strong>Are logged-in administrators affected?</strong></summary>
They are never redirected, and by default they are never blocked by IP rules either. You can change the IP behaviour under Advanced settings.
</details>

<details>
<summary><strong>Does the plugin work with Cloudflare?</strong></summary>
Yes. Choose the Cloudflare option under Advanced settings. The plugin only trusts Cloudflare's headers when the request genuinely came from a Cloudflare server, so nobody can pretend to be at a different IP address.
</details>

<details>
<summary><strong>What happens if my API key is missing or wrong?</strong></summary>
Country redirects, country access control and the shortcodes stop working, because the plugin cannot determine a visitor's location. Visitors see your default content. IP rules keep working.
</details>

<details>
<summary><strong>Can I detect VPN, proxy or Tor users?</strong></summary>
Yes, on a paid plan. Use <code>is_proxy</code>, <code>is_tor</code>, <code>is_anonymous</code> or <code>cloud_provider</code> in either the display or the conditional shortcodes.
</details>

<details>
<summary><strong>Will geo redirects hurt my SEO?</strong></summary>
Known search engine crawlers are excluded from country redirects and country access control, so your pages are indexed as written. Use 301 only for redirects you intend to be permanent, and prefer conditional shortcodes over redirects when you only need to change a banner or a price.
</details>