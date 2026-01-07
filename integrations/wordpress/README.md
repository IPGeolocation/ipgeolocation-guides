# **IPGeolocation.io – Geo Redirects & Content Control Plugin**

## Introduction

Imagine your website could automatically greet visitors in their language, show them relevant local offers, or redirect them to the perfect regional page—all based on where they're browsing from. That's exactly what the [IPGeolocation.io – Geo Redirects & Content Control Plugin](https://wordpress.org/plugins/ipgeolocation-geo-redirects-content-control/) does!

This smart WordPress plugin uses visitors' IP addresses to customize their experience on your site. Whether you're running an international store, a global blog, or a local service, this plugin helps you speak directly to each visitor in a way that feels personal and relevant to their location.

### **Why This Matters for Your Website**

- **Higher Conversions**: People are 70% more likely to buy when they see content in their language and currency
- **Better User Experience**: Visitors get information that's actually useful for their location
- **Smart Content Control**: Show special offers, local events, or regional notices only to the right audience
- **Improved Security**: Block unwanted traffic from specific regions

## **What Can You Do With This Plugin?**

### **Smart Redirects (Guide Visitors to the Right Place)**

- Send visitors from **France** straight to your French-language website
- Redirect **Canadian** customers to your Canadian prices (in CAD dollars!)
- Create special offers for visitors from specific **cities or states**
- Show a friendly popup before redirecting: "Taking you to our UK store. OK?"

*Perfect for:* International businesses, regional service providers, multi-language websites

### **Location-Based Content Control (Show/Hide Content)**

- Display a "Free Shipping in the USA" banner **only to US visitors**
- Show different contact information for **European vs Asian customers**
- Hide certain products in countries where they're not available
- Welcome visitors by their city name: "Welcome, visitors from London!"

*Perfect for:* Global blogs, membership sites, regional service announcements

### **Geographic Access Control (Who Can See Your Site)**

- Block traffic from countries with too much spam or unwanted attention
- Create VIP access for visitors from specific regions
- Allow only local visitors to see special local offers

*Perfect for:* Digital product sellers, local businesses, content creators

### **Visitor Information Display (Personalize Their Experience)**

- Show visitors their own location information
- Display currency, or language based on their location
- Detect if someone is using a VPN or proxy connection

*Perfect for:* Community sites, local directories, personalized services

## **How to Get Started (Step-by-Step)**

### **Step 1: Get Your Free API Key**

The plugin needs a "key" to work—think of it like a password that lets your WordPress site talk to our location service.

1. Go to [**ipgeolocation.io**](https://ipgeolocation.io/)
2. Click **Sign up** and create your account or **Login** if you have account already
3. **Copy your API key** from your account dashboard

### **Step 2: Install & Activate the Plugin**

1. In your WordPress dashboard, go to **Plugins → Add New**
2. Search for "**IPGeolocation – Geo Redirects & Content Control**"

![Add Plugin and Geo Redirects](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/add-plugin.png)

1. Click "**Install Now**" then "**Activate**"
2. Go to the plugin settings and **paste your API key** and **select the API plan type** you are on.

![API Configuration](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/api-configs.png)

Imagine your website could automatically greet visitors in their language, show them relevant local offers, or redirect them to the perfect regional page—all based on where they're browsing from. That's exactly what the [IPGeolocation.io](http://ipgeolocation.io/) – Geo Redirects & Content Control Plugin does!**Important**

- Please make sure to select the correct plan type. Choosing the wrong option may cause the plugin to not work properly.

## Geo Redirection Rules

A Geo Redirect Rule allows you to redirect visitors based on their country. When a visitor’s location matches the rule, the redirect is applied according to the settings you configure.

![Geo Redirection Rules](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/redirect-rules.png)

**Country Code**

- Enter a 2-letter ISO country code (for example: **US, GB or CA**)
- The rule will only apply to visitors from this country

**Apply To**

- **Entire Site**: Applies the redirect across all pages on your website
- **Specific Page**: Applies the redirect only when a selected page is visited
- **URL Pattern Matching**: Applies the redirect to URLs matching a pattern (for example **/shop/*** or **/products/***) ****

**Redirect URL**

- The destination URL where matching visitors will be redirected
- Supports both relative (**/example**) and absolute URLs

**Redirect Type**

- **301 (Permanent)**: Use when the redirect is permanent and should be indexed by search engines
- **302 (Temporary)**: Use for testing, campaigns, or temporary redirects

**Redirect Method**

- **Automatic**: Visitors are redirected immediately without any interaction
- **Popup**: Visitors see a confirmation popup and choose whether to continue

If **Popup** is selected, you can customize the popup appearance:

- Popup message text
- Text and background colors
- Yes and No button labels
- Button background and text colors

A live preview is shown so you can see changes instantly.

![Popup Appearance](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/popup-appearance.png)

**Popup behavior**

- Clicking **Yes** redirects the visitor to the target URL
- Clicking **No** keeps the visitor on the current page

**Notes:**

- Popup redirects are user-friendly and GDPR-friendly
- Automatic redirects are best for country-specific pages

## Country Access Rules

Country Access Control lets you **allow or block access to your site based on the visitor’s country**. This logic runs automatically on the frontend and does not affect admins, search engine bots.

The system automatically skips:

- WordPress admin area
- Logged-in administrators
- Search engine bots

This ensures normal site operations and SEO safety.

The visitor’s country is detected using IP-based geolocation.

If the country cannot be detected, no restriction is applied.

**Mode**

- **Block**: Selected countries are blocked
- **Allow**: Only selected countries are allowed; all others are blocked

![Country Access Control](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/country-access-control.png)

**Countries**

- A list of 2-letter ISO country codes (for example: **US**, **GB**, **CA**)
- Country codes are case-insensitive

**Redirect URL**

- The redirect URL must be relative (You may design the custom block page where the blocked users can be redirected to).

**Behavior summary**

- If **Block mode** is enabled and the visitor’s country matches the list → access is blocked
- If **Allow mode** is enabled and the visitor’s country does NOT match the list → access is blocked
- Allowed visitors continue browsing normally

**Notes**

- Runs early and only on frontend requests
- Uses strict country matching
- Safe for caching and SEO
- Fully automatic once configured

## Page Exclusion Rules

Excluded pages let you **skip geo rules on specific URLs**.

If a page matches an exclusion rule, country redirection rules will not run on that page.

The plugin checks the current page URL, including:

- The page path (for example **/checkout**)
- The page URL contains (for example **/blog)**
- URL parameters (for example **ref=google**)

![Page Exclusion Rules](https://static.ipgeolocation.io/web-assets/images/integrations/wordpress/page-exclusion-rules.png)

Each exclusion rule has a **type** and a **value**.

If any rule matches, the page is excluded.

**Page URL Equals**

- Matches only one exact page
- Best for excluding a single URL

**Example**

- Value: **/checkout**
- Matches: **/checkout**
- Not: **/checkout/cart**

**Page URL Contains**

- Matches a section or group of pages
- Best for folders like blogs or shops

**Example**

- Value: **/blog**
- Matches: **/blog** and **/blog/post-name**

**Page Query Contains**

- Matches URL parameters
- Useful for tracking or campaign URLs

**Example**

- Value: **ref=google**
- Matches: **/shop?ref=google**

**Notes**

- Only one match is needed to exclude the page
- Trailing slashes are handled automatically
- Works with normal URLs and query strings

## **Shortcodes & Dynamic Content Display**

### **Display Visitor Location Information**

Use these simple shortcodes anywhere in your posts, pages, or widgets to show location-based information:

text

```
[ipgeo country]       <!-- Shows country name (e.g., United States) -->
[ipgeo city]          <!-- Shows city name (e.g., New York) -->
[ipgeo state]         <!-- Shows state/province (e.g., California) -->
[ipgeo country_code]  <!-- Shows 2-letter code (e.g., US, GB, DE) -->
[ipgeo continent]     <!-- Shows continent (e.g., North America) -->
[ipgeo currency]      <!-- Shows local currency (e.g., USD, EUR) -->
[ipgeo calling_code]  <!-- Shows country dialing code (e.g., +1) -->
[ipgeo languages]     <!-- Shows primary languages (e.g., English) -->
[ipgeo latitude]      <!-- Shows latitude coordinates -->
[ipgeo longitude]     <!-- Shows longitude coordinates -->
```

**Advanced fields** (available on Advanced/Security plans)

```
[ipgeo is_proxy]      <!-- Shows if visitor uses proxy (true/false) -->
[ipgeo is_tor]        <!-- Shows if visitor uses Tor browser -->
[ipgeo is_anonymous]  <!-- Shows if connection is anonymous -->
[ipgeo cloud_provider]<!-- Shows if from AWS, Google Cloud, etc. -->
```

### **Conditional Content Shortcodes**

Show or hide content based on visitor location:

**Show content to specific visitors**

```
[ipgeo_if country_code="US,CA" logic="OR"]
This content is only visible to visitors from the United States or Canada.
[/ipgeo_if]
```

**Hide content from specific visitors**

```
[ipgeo_if_not country="Germany"]
This content is hidden from visitors in Germany.
[/ipgeo_if_not]
```

**Multiple conditions with AND logic**

```
[ipgeo_if country_code="US" state="California" logic="AND"]
Welcome, California visitors! Check out our local deals.
[/ipgeo_if]
```

**Available attributes for conditional content**

- `country` (country name)
- `country_code` (2-letter code)
- `state` (state/province)
- `city` (city name)
- `continent` (continent name)
- `is_proxy` (yes/no)
- `is_tor` (yes/no)
- `is_cloud_provider` (yes/no)
- `is_anonymous` (yes/no)
- `proxy_type` (type of proxy if detected e.g., vpn)

**Logic operators**

- `logic="AND"` - ALL conditions must match (default)
- `logic="OR"` - ANY condition can match

### **Practical Shortcode Examples**

**E-commerce Personalization**

```
[ipgeo_if country_code="US"]
<p class="local-offer">🇺🇸 Free shipping across the USA!</p>
[/ipgeo_if]

[ipgeo_if country_code="DE"]
<p class="local-offer">🇩🇪 Kostenloser Versand in Deutschland!</p>
[/ipgeo_if]
```

**Local Business Targeting**

```
[ipgeo_if city="New York"]
<div class="nyc-banner">
    <h3>New York City Customers!</h3>
    <p>Same-day delivery available in NYC</p>
    <button>Book NYC Delivery</button>
</div>
[/ipgeo_if]
```

**Currency Display**

```
<div class="price">
    Price: [ipgeo currency] 49.99
</div>
```

**Welcome Message with Location**

```
<h2>Welcome, visitor from [ipgeo city]!</h2>
<p>We're glad to have you here from [ipgeo country].</p>
```

## Frequently Asked Questions

<details>
<summary><strong>Does this plugin redirect logged-in administrators?</strong></summary>

No. Administrators are automatically excluded from all redirects and access rules.

</details>

<details>
<summary><strong>Are bots and search engines redirected?</strong></summary>

No. Known bots and search engine crawlers (e.g., Google, Bing, Facebook, Twitter) are excluded from redirects so your SEO indexing remains unaffected.

</details>

<details>
<summary><strong>Does the plugin cache API responses?</strong></summary>

Yes. IPGeolocation stores geolocation data per IP in WordPress transients for <strong>24 hours</strong>, improving performance and reducing API calls.

</details>

<details>
<summary><strong>Does the plugin work with Cloudflare?</strong></summary>

Yes. The plugin supports the <code>CF-Connecting-IP</code> header, allowing it to detect visitor IPs even when Cloudflare is enabled.

</details>

<details>
<summary><strong>Will this slow down my website?</strong></summary>

No. API calls are cached, and geolocation rules are processed efficiently so page load impact is minimal under normal conditions.

</details>

<details>
<summary><strong>What happens if the IPGeolocation API key is missing or incorrect?</strong></summary>

Without a valid API key, the plugin’s core features (redirects, content rules, and access control) <strong>will not function</strong> because it cannot determine visitor location. Users might see default content or no redirection until the key is configured.

</details>

<details>
<summary><strong>Can I temporarily bypass redirection for testing or users?</strong></summary>

Yes. You can add query parameters for control:

- <code>?geo_bypass=1</code> — bypasses redirects for 30 days  
- <code>?geo_reset=1</code> — resets the bypass state and allows redirects again

</details>

<details>
<summary><strong>How can I verify that redirects or content rules are working?</strong></summary>

To test rules:

- Use a <strong>VPN or proxy</strong> to simulate different countries.
- Try <code>?geo_bypass=1</code> to skip redirects temporarily.
- Use an incognito browser session to avoid cached rules.

</details>

<details>
<summary><strong>Is there a way to detect VPN, proxy, or Tor users?</strong></summary>

Yes — with the <strong>Advanced or Security API plan</strong>, you can detect if a visitor is using a proxy, VPN, or Tor. These details can also be used in conditional shortcodes to show or hide relevant content.

</details>

<details>
<summary><strong>Can I show different prices, banners, or contact details based on location?</strong></summary>

Absolutely. Use the conditional content shortcodes (e.g., <code>[ipgeo_if country_code="US"]</code>) to display localized banners, contact information, prices, or other content tailored to the visitor’s country or region.

</details>

