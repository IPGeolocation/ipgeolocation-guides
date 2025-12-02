# **IP Geolocation Shopify Integration**

## **1\. OverView**

It is a powerful, free Shopify app that delivers a seamless international shopping experience for your customers. Whether you manage multiple markets within one Shopify store or operate separate stores, IP Geolocation automatically detects your visitors' locations and guides them to the right storefront, in their preferred language and with the correct experience.

### **Who Is This App For?**

IP Geolocation shopify app is designed for Shopify merchants expanding internationally. If you're selling to customers across different countries, regions, or languages, this app ensures every visitor lands on the right version of your store.

Whether you're a small business testing new markets or an established brand with global operations, IP Geolocation provides the tools you need to create a localized shopping experience that feels native to each customer.

### **Key Features**

**Automatic Country Detection**: The app identifies where your visitors are browsing from and shows them the most relevant store or market for their location.

**Smart Popup with Consent**: A customizable popup asks visitors if they'd like to switch to their local store, respecting their choice and remembering it for future visits.

**Multi-Store Connection**: Link up to three separate Shopify stores for free, allowing visitors to move seamlessly between them without realizing they've switched domains.

**Unlimited Custom Translations**: Create fully translated popup text in any language. Choose between language-only translations or country-specific translations for even more precision.

**Manual Market Selector**: Give customers control with a selector that lets them manually switch regions or languages anytime while browsing.

**Advanced Redirect Rules**: Control exactly when and where the popup appears based on visitor type, country, and URL patterns.

**SEO-Friendly Redirects**: Built with search engines in mind to ensure each localized storefront is properly indexed which helps in improving SEO.

**UTM Tracking Support**: Preserve marketing attribution across redirects for accurate campaign tracking.

### **How It Works**

When a visitor lands on your store, IP Geolocation detects their country using their IP address. If they're browsing from a location where you have a different market or connected store available, the app displays a popup asking if they'd like to switch.

If the visitor accepts, they're redirected to the appropriate storefront. The app remembers their choice, so returning visitors are automatically taken to their preferred store without seeing the popup again (unless you configure it otherwise).

Visitors can also use the market selector at any time to manually switch between available regions or languages, giving them full control over their shopping experience.

### **What Makes IP Geolocation Different**

While other redirecting apps charge for store connections and translations, IP Geolocation offers these features completely free. You can connect up to three stores and create unlimited translations at no cost.

Our app includes both language-only translations and country-plus-language translations for free, features that are typically locked behind premium tiers in other apps. The popup is simple, functional, and fully customizable to match your brand's design while delivering an excellent user experience.

### **Ready to Get Started?**

Head to the Getting Started guide to install IP Geolocation and configure your first market settings. Within minutes, you'll be delivering a personalized experience to every international visitor.

## **2\. Getting Started**

This guide will walk you through installing IP Geolocation and completing your initial setup. The entire process takes just a few minutes.

### **Installation**

**Step 1: Install from Shopify App Store**

Visit the Shopify App Store, search for "IP Geolocation," and click Install. You'll be redirected to your Shopify admin to approve the app's permissions.

**Step 2: Access the App Dashboard**

Once installed, open the app from your Shopify admin under Apps \> IP Geolocation. You'll land on the home page, which displays your setup progress and key stats.

### **Initial Setup Checklist**

When you first open the app, you'll see a setup progress bar at the top of your dashboard. This tracks three essential setup tasks:

**Connect Stores**: If you operate multiple Shopify stores, connect them together so visitors can switch between them seamlessly. If you only use one store with Shopify Markets, you can skip this step.

**Add Translations**: Create custom popup text in different languages to match your visitors' preferences. This is optional, if you don't add translations, the app will use the default text you configure in the Geo Redirect Popup settings in App embeds.

**Configure Rules**: Set up rules to control when and where the geolocation popup appears. This determines which visitors see the popup and under what conditions.

As you complete each task, the progress bar fills up, giving you a clear view of your setup status.

### **Understanding Your Markets**

IP Geolocation automatically detects the markets you've configured in Shopify. These markets represent the different countries or regions where you sell, along with their associated languages and currencies.

On your dashboard home page, you'll see a list of your current markets. If any markets are missing, perhaps because the automatic sync didn't trigger, you can use the manual Sync button to refresh the list.

### **Configuring the Popup and Selector**

The popup and market selector are configured through Shopify's app embeds settings, not within the IP Geolocation dashboard.

**Step 1: Go to Theme Settings**

In your Shopify admin, navigate to Online Store \> Themes. Click Customize on your active theme.

**Step 2: Enable App Embeds**

In the theme editor, click on the App embeds section in the left sidebar. Find Geo Redirect Popup and Geo Market Selector and toggle them on.

**Step 3: Customize Popup Text**

Once enabled, you'll see customization options for the popup, including the title, body text, and button labels. You can also insert dynamic variables like {country}, {city}, {currency\_code}, and {currency\_symbol} to personalize the message for each visitor.

**Step 4: Configure the Market Selector**

The market selector settings are also in the app embeds section. Customize its appearance and behavior.

**Step 5: Save Your Changes**

Click Save in the top right corner of the theme editor to publish your changes.

## **3\. Understanding Your Dashboard**

The IP Geolocation dashboard is your command center for monitoring visitor activity, tracking performance, and managing your market settings. This page explains each section of the dashboard and what the metrics mean.

![Homepage screenshot](https://static.ipgeolocation.io/web-assets/images/integrations/shopify/homepage.png)

### **Setup Progress Bar**

At the top of your dashboard, you'll see a progress bar that tracks three key setup tasks: connecting stores, adding translations, and configuring rules.

This bar fills as you complete each task, giving you a quick visual indicator of your setup status. Once all three tasks are done, the bar will be fully filled, and your app will be ready to deliver a complete geolocation experience.

### **Key Metrics Overview**

Below the progress bar, the dashboard displays four key metrics that help you understand how visitors are interacting with your store:

**Geolocation Lookups**: This shows the total number of times the app has detected a visitor's country using their IP address. Every time someone lands on your store, the app performs a lookup to determine their location. This metric tells you how much traffic you're receiving.

**Successful Redirects**: This counts how many times visitors have been redirected to a different market or connected store. It also includes a Reset option for successful redirects, which resets it if needed.

**Connected Stores**: This displays the total number of Shopify stores you've linked together using the Connect Stores feature. If you only use one store with Shopify Markets, this will show zero.

**Translations Created**: This shows how many custom translations you've added to personalize the popup for different languages or countries.

These metrics give you a snapshot of your app's activity and help you gauge how well your geolocation strategy is performing.

### **Current Markets**

Below the key metrics, you'll see a list of the markets you've configured in Shopify. Each market represents a country or region where you sell, along with its associated languages.

The app automatically syncs these markets from your Shopify settings using webhooks. When you add,  remove or edit a market in Shopify, the app should update automatically.

### **Manual Market Sync**

If you notice that a market is missing from your dashboard, perhaps because the webhook didn't trigger, you can use the manual Sync button to refresh the list.

Click the Sync button, and the app will pull the latest market data from Shopify. This ensures your dashboard always reflects your current market configuration.

### **Monitoring Performance**

Check your dashboard regularly to monitor how visitors are interacting with your geolocation settings. If you notice low redirect numbers despite high lookup counts, it might indicate that visitors are declining the popup or that your rules need adjustment.

Use the metrics to inform decisions about your popup text, translation strategy, and rule configuration. The data on your dashboard is your best guide for optimizing the visitor experience.

## **4\. Configuring Redirect Rules**

The Rules page gives you precise control over when and where the geolocation popup appears. You can target specific countries, customize behavior for first-time and returning visitors, and exclude the popup from certain URLs.

![Redirect Rules 1st screenshot](https://static.ipgeolocation.io/web-assets/images/integrations/shopify/redirect-rules.png)

### **Visitor Targeting**

The first section of the Rules page determines which visitors see the popup based on their country.

**Show to All Visitors**: The popup will appear for everyone, regardless of location. This is the simplest option and ensures no visitor is missed.

**Show to Specific Countries Only**: The popup will only appear for visitors from the countries you select. This is useful if you want to target specific markets or test the feature in certain regions before rolling it out globally.

**Hide from Specific Countries**: The popup will appear for all visitors except those from the countries you select. Use this option if you want to exclude certain regions, such as your home market where redirects aren't needed.

Choose the option that best matches your business model and market strategy.

### **First-Time Visitor Behavior**

This setting controls what happens when someone visits your store for the very first time.

**Always Show Popup**: The popup will appear every time a first-time visitor lands on your store, regardless of whether they're on the correct market or not. This ensures maximum visibility and gives every visitor the option to choose their preferred store.

**Show Only on Wrong Storefront**: The popup will only appear if the visitor is browsing a market that doesn't match their detected country. For example, if a visitor from France lands on your US store, they'll see the popup. But if they're already on the French store, the popup won't appear. This reduces interruptions for visitors who are already on the right storefront.

Select the behavior that balances visibility with user experience. If you want to maximize conversions by ensuring every visitor sees the popup, choose "Always Show." If you prefer a more subtle approach that only intervenes when needed, choose "Show Only on Wrong Storefront."

### **Returning Visitor Behavior**

This setting determines how the app handles visitors who have already made a choice in a previous session.

**Always Redirect to Saved Preference**: Returning visitors will be automatically redirected to the store they chose during their last visit, without seeing the popup again. This creates a seamless experience for repeat customers who have already indicated their preference.

**Redirect Only if Location Matches**: The app will check if the visitor's current location still matches their saved preference. If it does, they'll be redirected automatically. If their location has changed (for example, they're traveling), the popup will appear again, giving them a chance to update their choice.

Most merchants choose "Always Redirect to Saved Preference" for a smoother returning visitor experience. However, if your customers frequently travel or use VPNs, "Redirect Only if Location Matches" may be more appropriate.

### **URL Exclusion Rules**

Sometimes you don't want the popup to appear on certain pages, such as checkout, policy pages, or specific landing pages. The URL exclusion section lets you prevent the popup from showing on specific URLs.

**How to Add Exclusions**

Enter the URL path you want to exclude in the input field. You can use three types of patterns:

**Exact Path Match**: Excludes a specific page exactly as written. Example: `/pages/about` will exclude only the About page.

**Single-Level Wildcard**: Use an asterisk (`*`) to match any single segment of a URL. Example: `/collections/*` will exclude the collections page and its subpages.

**Multi-Level Wildcard**: Use a double asterisk (`**`) to match any number of URL segments. Example: `/blogs/**` will exclude all the subpages of blogs not including blogs in it

## **5\. Managing Translations**

Custom translations allow you to display the popup in your visitors' native languages, creating a more welcoming and localized experience. You can create translations for specific languages, specific countries, or both.

![Translations screenshot](https://static.ipgeolocation.io/web-assets/images/integrations/shopify/translations.png)

### **Why Use Custom Translations?**

By default, the popup displays the text you configure in the Geo Redirect Popup settings. This text is the same for all visitors, regardless of their language or country.

Custom translations override this default text and show different content based on the visitor's detected language or country. This is especially valuable if you serve customers in multiple languages or want to tailor messaging for specific regions.

### **Enabling Custom Translations**

On the Translations page, you'll see a toggle to enable or disable the custom translations feature.

When enabled, the app will check for matching translations when a visitor lands on your store. If a match is found, the translated text is displayed. If no match is found, the fallback text is used which you can also configure.

When disabled, the app will always use the default text, ignoring any translations you've created.

### **Translation Strategy**

Before creating translations, you need to choose a translation strategy. This determines how the app matches visitors to translations.

**Language Only**: The app matches translations based solely on the visitor's detected language. For example, if you create a translation for French, it will be shown to all French-speaking visitors, regardless of which country they're in.

**Country and Language**: The app matches translations based on both the visitor's country and language. For example, you can create separate translations for French speakers in France versus French speakers in Canada, allowing you to tailor messaging for each region.

**Note**: if you change the strategy then all the previous translations would be deleted.

### **Creating a Translation**

To create a new translation, click the create button on the Translations page.

**Step 1: Select Country (if using Country and Language strategy)**

If you've chosen the "Country and Language" strategy, you'll also need to select a country. The app will only show this translation to visitors from that specific country who speak the selected language.

**Step 2: Select Language**

Choose the language for this translation from the dropdown menu. The app will show this translation to visitors whose detected language matches your selection.

**Step 3: Enter Translated Text**

Enter the translated versions of your popup title, body text, and button labels. You can use the same dynamic variables ({country}, {city}, {currency\_code}, {currency\_symbol}) in your translations.

**Step 4: Save the Translation**

Click Save to add the translation to your library.

### **Using Dynamic Variables in Translations**

Dynamic variables work the same way in translations as they do in the Geo Redirect Popup text. The app automatically inserts location-specific information when the popup is displayed.

For example, a French translation might include:

"Bienvenue \! Il semble que vous visitiez depuis {country}. Souhaitez-vous acheter en {currency\_code} ?"

When shown to a visitor from France, this would display:

"Bienvenue \! Il semble que vous visitiez depuis la France. Souhaitez-vous acheter en EUR ?"

### **Editing and Deleting Translations**

The Translations page lists all the translations you've created. You can edit or delete any translation at any time.

To edit a translation, click the Edit button next to it, make your changes, and save.

To delete a translation, select the translations you want to delete then click the Delete button. The app will immediately stop using that translation and fall back to the default text for visitors who would have matched it.

### **Fallback Text Behavior**

If a visitor's language or country doesn't match any of your custom translations, the app will display the fallback text you will configure in the translation page.

This fallback ensures that every visitor sees a popup, even if you haven't created a translation for their specific language or country.

## **6\. Connecting Multiple Stores**

If you operate separate Shopify stores for different countries or regions, the Connect Stores feature links them together into one seamless experience. Visitors can switch between connected stores without realizing they've moved to a different domain. After the connection is made, all features, rules, translations, and popup behavior are managed entirely from your group owner’s dashboard, keeping configuration simple and centralized.

![Connet-Store screenshot](https://static.ipgeolocation.io/web-assets/images/integrations/shopify/connect-store.png)

### **What is Store Connection?**

Store connection allows you to group multiple Shopify stores so they function as one unified storefront from the visitor's perspective. When a visitor switches markets using the popup or selector, they're redirected to the appropriate connected store.

For example, you might have separate stores for the US, UK, and Australia. By connecting these stores, a visitor from Australia who initially lands on your US store can be redirected to your Australian store, where they'll see local pricing, inventory, and content.

### **Why Use Store Connection?**

Some merchants prefer to operate separate Shopify stores for each country or region instead of using Shopify Markets within one store. Reasons for this approach include:

**Greater Control**: Separate stores allow for complete customization of products, pricing, and content for each region.

**Different Business Entities**: If you have separate legal entities or partnerships in different countries, separate stores may be necessary.

**Legacy Setup**: You may have already established separate stores before Shopify Markets was available.

Store connection bridges these separate stores, ensuring visitors can move between them smoothly while maintaining a consistent brand experience.

### **Understanding Connection IDs**

Every store using IP Geolocation has a unique Connection ID. This ID is used to link stores together.

You can find your store's Connection ID on the Connect Stores page in the IP Geolocation dashboard. It's a unique string of characters that identifies your store within the app.

### **How to Connect Stores**

Connecting stores requires coordination between the owners or administrators of each store. Here's how it works:

**Step 1: Share Your Connection ID**

On the Connect Stores page, copy your store's Connection ID and share it with the owner or administrator of the store you want to connect to. This could be another person in your organization or a partner managing a regional store.

**Step 2: Enter the Other Store's Connection ID**

Once you receive the Connection ID from the other store, enter it in the "Connect a Store" field on your Connect Stores page and click Connect.

**Step 3: Handle Market Conflicts (Important)**

If the two stores have overlapping markets (e.g., both target “Canada”),the app will ask which store’s market setup should be used.This avoids incorrect redirects and ensures clean market logic.

**Future Conflicts Are Also Detected Automatically**

Even after store connection:

* If a merchant later adds a new market in Shopify.

* And that market already exists in the connected group.

* Our app will notify the merchant that a conflict must be resolved.

The merchant will be instructed to visit the Connect Stores page to fix the conflict and choose which store should own that region.

This ensures consistent behaviour and eliminates accidental misrouting in multi-store setups.

**Step 4: Confirm the Connection**

The app will verify the Connection ID and link the two stores together. Both stores will now appear in each other's "Connected Stores" list.

**Step 5: Repeat for Additional Stores**

If you want to connect more than two stores, repeat the process. Each store enters the Connection IDs of the others, and the app groups them all together.

You can connect up to three stores for free.

### **Managing Connected Stores**

The "Connected Stores" section on the Connect Stores page lists all stores currently linked to yours. Each entry includes:

**Store Name**: The name of the connected store.

**Country**: The primary country or market the store serves.

**Owner**: Whether the store is owned by you or another user.

From this section, you can also disconnect a store if needed.

### **Disconnecting a Store**

If you no longer want a store to be connected, click the Disconnect button next to it in the Connected Stores list.

Once disconnected, that store will no longer appear in your popup or selector, and visitors won't be redirected to it. The disconnected store will return to its own separate group.

Disconnecting a store doesn't delete any data or settings; it simply removes the link between the stores.

### **How Visitors Experience Connected Stores**

From the visitor's perspective, connected stores appear as one seamless experience. When they click the popup or selector and choose a different market, they're redirected to the appropriate store.

The app preserves their choice, so if they return later, they'll automatically land on their preferred store. The transition is smooth and fast, and most visitors won't even realize they've switched domains.

### **Important Considerations**

**Shared Markets**: When stores are connected, they share market data. Ensure that each store is configured with the correct markets in Shopify so the app can properly detect and redirect visitors.

**Consistent Branding**: For the best visitor experience, ensure that all connected stores have consistent branding, design, and navigation. Differences between stores can confuse visitors and reduce trust.

**Coordination**: Connecting stores requires coordination between store owners or administrators. Make sure all parties understand how the connection works and agree to maintain the link.

Store connection is a powerful feature that extends the capabilities of IP Geolocation beyond single-store setups, enabling complex international operations while maintaining a simple, unified experience for visitors.

## **7\. Geo Redirect Popup**

The geolocation popup is the primary way visitors interact with your app. Customizing its appearance and content ensures it matches your brand and delivers a clear, compelling message.

### **Accessing Popup Settings**

The popup is configured through Shopify's app embeds settings, not within the IP Geolocation dashboard.

**Step 1: Open Theme Editor**

In your Shopify admin, go to Online Store \> Themes and click Customize on your active theme.

**Step 2: Open App Embeds**

In the theme editor, click the App embeds section in the left sidebar. Find Geo Redirect Popup and toggle it on if it is not already enabled.

**Step 3: Customize Popup Content**

Once enabled, you'll see customization options for the popup, including fields for the title, body text, button labels, and more.

### **Popup Content Fields**

It several text fields you can customize:

**Title**: The main heading visitors see when the popup appears. Keep it short and clear, such as "Welcome\!" or "Choose Your Store."

**Fallback Title**: A backup title that appears for countries where shipping is unavailable.

**Subtitle**: A secondary line of text displayed under the title, commonly used to guide the visitor’s next step.

**Country Label & Language Label**: These define the text shown above the country and language selectors.

**Helper Text**: An optional description that appears below the selectors. It can include dynamic variables to give visitors personalized information about their location and currency.

**Button Text**: The label for the call-to-action button, such as “Shop now.”

Customize each field to match your brand's tone and messaging style.

### **Using Dynamic Variables**

Dynamic variables let you personalize the popup content based on each visitor's location. The app supports the following variables:

**{country}**: Inserts the visitor's detected country name (e.g., "France," "Canada").

**{city}**: Inserts the visitor's detected city name (e.g., "Paris," "Toronto").

**{currency\_code}**: Inserts the currency code for the visitor's market (e.g., "EUR," "CAD").

**{currency\_symbol}**: Inserts the currency symbol for the visitor's market (e.g., "€," "$").

**Example Usage**

"It looks like you're visiting from {country}. Would you like to shop in {currency\_code}?"

"Welcome\! We noticed you're in {city}. Switch to your local store for prices in {currency\_symbol}."

Dynamic variables make the popup feel more relevant and personalized, which can increase the likelihood that visitors will accept the redirect.

### **Design Customization**

In addition to text, you can customize the popup's visual appearance to match your store's design. Options typically include:

**Test mode**: Force-show the popup inside the theme editor for styling.

**Show close button**: Enable or disable the “X” button.

**Display country flag**: Show a localized flag icon in the popup.

**Button and selector style**: Choose between square or rounded shapes.

**Selector type**: Choose outlined, underlined or without lines styles.

**Button alignment**: Position the CTA button left, center, or right.

**Font sizes**: Customize title and text sizes in pixels.

**Background overlay**: Select the overlay color behind the popup.

**Images**: Add desktop and mobile background images and choose their alignment.

**Color controls**: Customize CTA text, interactive elements, selector background, and general text color.

These options allow you to create a popup that blends seamlessly with your store’s theme while providing a clear and intuitive experience for visitors.

## **8\. Geo Market Selector**

The market selector gives visitors control over their shopping experience by allowing them to manually switch between available markets or connected stores at any time.

### **What is the Selector?**

The selector is a widget that appears on your store while visitors are browsing. It displays a list of all available markets or connected stores, showing the countries and languages you support.

When a visitor clicks the selector and chooses a different market, they're redirected to that storefront, just as if they had accepted the geolocation popup.

### **Why Use the Selector?**

While the geolocation popup handles automatic detection and initial redirection, the selector empowers visitors to change their choice at any point during their session.

This is especially useful for:

**Visitors Who Declined the Popup**: If someone initially chose to stay on the current store but later wants to switch, the selector provides an easy way to do so.

**Travelers or VPN Users**: Visitors whose detected location doesn't match their actual preference can manually select the correct store.

**Returning Customers Exploring Other Markets**: Customers who are curious about products, pricing, or availability in other regions can browse different markets without triggering the popup again.

The selector ensures that visitors always have control, reducing frustration and improving the overall shopping experience.

### **Selector Types**

The app offers two types of selectors:

**1\. Classic Selector**

A compact selector that appears directly inside your header, footer, or any element you specify.  
 You decide where it appears using the Selector Position (CSS Selector) field.

**2\. Modal Selector**

A button that opens a modal popup when clicked.  
 Visitors select their region inside the modal.

Each type has its own customization options inside the app embed settings.

### **Configuring the Selector**

Like the popup, the selector is configured through Shopify's app embeds settings.

**Step 1: Open Theme Editor**

In your Shopify admin, go to Online Store \> Themes and click Customize on your active theme.

**Step 2: Open App Embeds**

In the theme editor, click the App embeds section in the left sidebar. Find the Geo Market Selector settings and enable it

**Step 3: Classic Selector Settings (Placement)**

If you choose Classic, you must provide the **Selector Position**. This field controls exactly where the selector appears on your storefront.

**How Placement Works**

The app will look for a specific CSS class or ID on your page and automatically insert the selector inside that element.	

**How to Use It**

1. Identify the element where you want the selector to appear  
    (for example, your header, top bar, footer, or a custom container).

2. Check the class or ID of that element in your theme.

   * Classes look like: `.header__icons`, `.site-footer__item`, `.top-bar`

   * IDs look like: `#shopify-section-header`, `#Footer`, `#announcement-bar`

Enter the class name or ID name into the Selector Position field

**Additional Classic Settings**

You can customize:

* Selector type (Outlined, Filled, etc.)

* Whether to show languages

* Flag display

* Button styles (rounded / square)

**Step 4: Modal Selector Settings**

If you select Modal, the selector becomes a clickable button that opens a modal popup.

You can customize:

* Modal title

* Hover color

* Flag display

* Corners (square/rounded)

* Icon and text styling

* Selector alignment (Left, Center, Right)

The modal appears as an overlay, so no CSS selector is required.

**Step 5: Customize Styles**

Under **Styles**, adjust the visual appearance:

* Background color

* Text color

* Interactive element color

* Rounded or square style

* Flags (on/off)

Adjust the selector's design to match your store's theme. Options may include colors, fonts, icon style, and size.

**Step 6: Save Your Changes**

Click Save to publish the selector on your live store.

### **How Visitors Use the Selector**

When a visitor clicks the selector, they see a list of all available markets or connected stores. Each entry typically includes the country name, flag, and supported languages.

The visitor clicks on their desired market, and the app redirects them to that storefront. Their choice is saved, so if they return to your store later, they'll automatically land on their selected market (depending on your returning visitor rules).

## **9\. Troubleshooting**

Even with careful setup, you may occasionally encounter issues with IP Geolocation shopif app. This section covers common problems and how to resolve them.

### **Popup Not Appearing**

If the popup isn't showing to visitors, check the following:

**App Embed Not Enabled**: Go to Online Store \> Themes \> Customize, open the App embeds section, and ensure Geo Redirect Popup is toggled on. Save your changes.

**Rules Configuration**: Check your Rules page to ensure the popup is configured to show for the visitor's country. If you've set it to "Show to Specific Countries Only," verify that the visitor's country is included. Also check URL exclusion rules to ensure the current page isn't excluded.

**Returning Visitor Redirect**: If the visitor has previously made a choice, they may be automatically redirected without seeing the popup. Test with a new private browsing window or clear cookies to simulate a first-time visitor.

**Browser Issues**: Some browser extensions or privacy settings may block the popup. Test in a different browser or incognito mode.

**JavaScript Errors**: Open your browser's developer console (F12) and check for JavaScript errors. If you see errors related to IP Geolocation, contact [support](https://ipgeolocation.io/contact.html) with the details.

### **Markets Not Syncing**

If your markets aren't appearing on the dashboard or seem outdated, try the following:

**Manual Sync**: Use the manual Sync button on the dashboard home page to refresh your market data from Shopify.

**Webhook Issues**: The app relies on webhooks to automatically sync market changes. If webhooks aren't working, the app may not receive updates. Contact Shopify [support](https://ipgeolocation.io/contact.html) to verify that webhooks are functioning correctly for your store.

**Shopify Markets Configuration**: Ensure that your markets are properly configured in Shopify. Go to Settings \> Markets in your Shopify admin and verify that all countries, languages, and currencies are set up correctly.

If manual sync doesn't resolve the issue, contact IP Geolocation [support](https://ipgeolocation.io/contact.html) for further assistance.

### **Redirects Not Working**

If visitors are seeing the popup but not being redirected when they accept, check the following:

**Connected Stores**: If you're using the store connection feature, ensure that all stores are properly connected. Check the Connected Stores list on the Connect Stores page to verify the links.

**Browser Restrictions**: Some browsers may block redirects if they appear to be automatic or suspicious. Test in different browsers to rule out browser-specific issues.

### **Translation Issues**

If custom translations aren't appearing correctly, check the following:

**Translations Enabled**: Go to the Translations page and ensure the "Enable Custom Translations" toggle is turned on.

**Translation Strategy**: Verify that your translation strategy (Language Only vs Country and Language) matches your setup.

**Language Detection**: The app detects the visitor's language based on their browser settings. If a visitor has their browser set to a different language than expected, they may see a different translation or the fallback text.

**Fallback Text**: If no translation matches the visitor's language or country, the app uses the fallback text you provided in the translations.

**Dynamic Variables**: If dynamic variables like {country} or {currency\_code} aren't populating correctly, verify that the visitor's location is being detected properly. Check your dashboard to see if geolocation lookups are being recorded.

### **Store Connection Problems**

If you're having trouble connecting stores or if connected stores aren't functioning correctly, try the following:

**Correct Connection ID**: Double-check that you're using the correct Connection ID. A single character error will prevent the connection from working.

**App Installed on Both Stores**: The IP Geolocation app must be installed and active on both stores you're trying to connect. If one store doesn't have the app installed, the connection will fail.

**Store Limit**: You can connect up to three stores for free. If you've reached this limit, you'll need to disconnect an existing store before adding a new one.

**Synchronization Delay**: After connecting stores, it may take a few moments for the connection to fully synchronize. Wait a minute or two and refresh the page.

**Disconnection Issues**: If you can't disconnect a store, ensure you have the proper permissions in the IP Geolocation dashboard. Only store owners can manage connections.

### **Common Errors and Solutions**

**"Market Not Found"**: This error indicates that the app couldn't find a matching market for the visitor's country. Ensure your markets are properly configured in Shopify and synced in the app.

**"Redirect Failed"**: This error suggests a problem with the redirect logic. Check your connected stores, market configuration, and browser settings. Contact [support](https://ipgeolocation.io/contact.html) if the issue persists.

**"Translation Not Found"**: This warning means the app couldn't find a matching translation and is using the fallback text instead. This isn't necessarily an error, it's expected behavior when no translation matches. Create additional translations if you want to cover more languages or countries.

### **Getting Additional Help**

If you've tried the troubleshooting steps above and are still experiencing issues, contact IP Geolocation [support](https://ipgeolocation.io/contact.html) through the Help Center in the app dashboard. Provide as much detail as possible, including:

What you were trying to do when the issue occurred.

Any error messages you saw.

Screenshots or screen recordings showing the problem.

Your store URL and the approximate time the issue occurred.

The [support](https://ipgeolocation.io/contact.html) team will investigate and help you resolve the issue as quickly as possible.

## **10\. FAQ**

This section answers the most common questions about IP Geolocation.

### **General Questions**

**Is IP Geolocation shopify app really free?**

Yes, it is completely free. You can connect up to three stores, create unlimited translations, and use all features at no cost. There are no hidden fees or premium tiers.

**Do I need a specific Shopify plan to use IP Geolocation?**

No, IP Geolocation works with any Shopify plan. There are no technical requirements or restrictions based on your plan level.

**Does IP Geolocation work with Shopify Markets?**

Yes, IP Geolocation is fully compatible with Shopify Markets. If you manage multiple markets within one store, the app will detect them automatically and redirect visitors to the correct market.

**Can I use IP Geolocation with separate Shopify stores?**

Yes, the Connect Stores feature allows you to link up to three separate Shopify stores together. Visitors can switch between connected stores seamlessly.

**How accurate is the country detection?**

IP Geolocation uses industry-standard IP geolocation databases, which are typically accurate to the country level for 99% of visitors.

**What happens if a visitor uses a VPN?**

If a visitor uses a VPN, the app will detect the VPN's location, not the visitor's actual location. This may result in the app showing the wrong market. However, visitors can always use the manual selector to choose the correct store.

### **Setup and Configuration**

**How long does setup take?**

Most merchants complete the initial setup in 5-10 minutes. This includes enabling the app embeds, configuring basic rules, and testing the popup.

**Do I need to add translations?**

No, translations are optional. If you don't add custom translations, the app will use the default text you configure in the app embeds settings for all visitors.

**What are dynamic variables?**

Dynamic variables are placeholders like {country} or {currency\_code} that the app automatically replaces with the visitor's location-specific information. For example, {country} might display "France" for a visitor from France.

**Can I exclude the popup from specific pages?**

Yes, the Rules page includes URL exclusion settings where you can prevent the popup from appearing on specific pages using exact paths or wildcard patterns.

### **Privacy and Compliance**

**Is IP Geolocation GDPR compliant?**

Yes, IP Geolocation is designed with privacy in mind. The app detects the visitor's country using their IP address but does not store or share personal data. The app asks for visitor consent before redirecting, which aligns with GDPR principles.

**Does the app store visitor data?**

The app stores the visitor's market choice in the visitors browser so it can remember their preference on future visits. It does not contain personal information, only the selected market or store.

**Do I need to update my privacy policy?**

It's a good practice to mention in your privacy policy that your store uses IP geolocation to detect visitor locations and offer a localized shopping experience. Consult with a legal professional if you're unsure about your specific requirements.

### **Performance and Technical**

**Will IP Geolocation slow down my store?**

No, IP Geolocation is designed to be lightweight and fast. The app loads asynchronously, meaning it doesn't block other content on your store. Most visitors won't notice any performance impact.

**How does the app handle redirects for SEO?**

The app uses client-side redirects that preserve your SEO structure. Search engines can still crawl and index each localized version of your store correctly. The app does not interfere with hreflang tags or other SEO signals.

**Can I test the app before publishing it to my live store?**

Yes, you can enable the app in your theme's customization settings and test it in preview mode before publishing. You can also use private browsing or a VPN to simulate visitors from different countries.

**Does the app work on mobile devices?**

Yes, It is fully responsive and works on all devices, including smartphones and tablets. Both the popup and the selector are optimized for mobile screens.

### **Translations and Languages**

**How many translations can I create?**

You can create unlimited translations at no cost. There are no limits on the number of languages or countries you can support.

**What's the difference between language-only and country-language translations?**

Language-only translations match visitors based solely on their browser language. Country-language translations match visitors based on both their country and language, allowing you to create separate text for French speakers in France versus French speakers in Canada, for example.

**What happens if no translation matches the visitor?**

If no custom translation matches the visitor's language or country, the app displays the fallback text you configured in the translations page. This ensures every visitor sees a popup, even if you haven't created a translation for their specific case.

### **Store Connection**

**How many stores can I connect to?**

You can connect up to three stores for free using the Connect Stores feature.

**Do all connected stores need to be owned by the same person?**

Yes, connected stores cannot be owned by different stores.

**What happens if I disconnect a store?**

When you disconnect a store, the markets assosiated to that store will be removed from your popup and selector. Visitors will no longer be redirected to that store. The disconnected store returns to its own separate group. No data is deleted, the connection is simply removed.


