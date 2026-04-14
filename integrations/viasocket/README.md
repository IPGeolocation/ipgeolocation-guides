# **IPGeolocation + viaSocket Integration**

## **Overview**

The [**viaSocket + IPGeolocation**](https://viasocket.com/integrations/ipgeolocation) **integration** allows you to use the full power of IPGeolocation.io APIs directly inside your viaSocket workflows \- **without writing any code**.You do not need to write code or handle API requests because everything can be automated through clear and simple actions.

With this integration, you can automate retrieval of valuable IP intelligence data such as:

* Geolocation details (country, city, coordinates)  
* Security & threat intelligence (VPN, proxy, TOR detection)  
* Network ownership and ASN details  
* Timezone and time conversion data  
* Astronomy data (sunrise, sunset, moon phase)  
* User-agent insights (browser, OS, device)

This helps you build smarter automations, improve personalization, and enhance security workflows.

---

## **Available Actions in [viaSocket](https://viasocket.com/integrations/ipgeolocation)**

The integration includes **12 actions**, grouped into 6 main categories:

| Category | Actions |
| ----- | ----- |
| Geolocation | Single IP Lookup, Bulk IP Lookup |
| IP Security | IP Security Lookup, Bulk IP Security Lookup |
| Network Intelligence | ASN Lookup, Abuse Contact Lookup |
| Time Services | Timezone Info, Time Conversion |
| Astronomy | Astronomy Details, Astronomy Time Series |
| User-Agent Parsing | Single Parsing, Bulk Parsing |

## **API Key & Connection Setup**

To connect IPGeolocation with viaSocket,you need a valid API Key. Follow these steps carefully:

### **Create or log in to your IPGeolocation.io account**

* Go to [IPGeolocation.io](https://ipgeolocation.io/).  
* If you don’t have an account, click [Sign Up](https://app.ipgeolocation.io/signup) and complete the registration.  
* If you already have an account, click [Sign in](https://app.ipgeolocation.io/login) and enter your credentials.

### **Obtain your API Key**

* After logging in, go to your [dashboard](https://app.ipgeolocation.io/dashboard).  
* Copy your API Key \- you’ll need it to connect viaSocket with IPGeolocation.io.

### **Open viaSocket and create a Workflow**

* Go to [viasocket.com](http://viasocket.com) and log in.  
* Get into the created Workspace.  
* Click **“Create new workflow”** from your dashboard.  
* You will see a canvas where you can add Trigger and actions (actions that perform tasks).

### **Add an IPGeolocation.io action**

* Click the action button to add an action.  
* Search for **IPGeolocation** in the module search bar.  
* Select any action event (e.g., Find Geolocation Information or Find IP Security).

![Search IPGeolocation](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/search-ipgeo.png)

### **Set up the connection**

* In the module settings, find the account field and click Sign in.  
* Paste the API Key you copied earlier into the required field.  
* Click Yes, Continue to IPGeolocation.io to authenticate the connection.

![Add Connection](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/add-connection.png)

### **Test the connection**

* viaSocket will attempt to connect using your API Key.  
* If successful, the popup will appear and you may proceed with the actions.  
* If there’s an error, double-check your API Key and make sure it’s active and valid for the plan you’re using.

![Test the Connection](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/added-successfully.png)

### **Use your connection**

* Once connected, this API Key is available for all IPGeolocation modules in your Workflows.  
* You can now use it to perform actions like Find Geolocation Information, Find User Agent, or Find Bulk IP Geolocation, depending on your API plan.

## **Modules**

Below is a structured reference of all modules in this integration.  
---

### **Geolocation**

**Get IP Geolocation**

Retrieves full geolocation data of a single IPv4/IPv6.

* Input: IP Address  
* Outputs: country\_name, city, latitude, longitude, zipcode and [many more](https://ipgeolocation.io/documentation/ip-location-api.html#location-json-object-reference)

**Get Bulk IP Geolocation**

Retrieves geolocation for up to 50,000 IPs per request.

* Input: Array of IPs  
* Outputs: Collection of geolocation objects

![IPGeolocation API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/ip-geolocation.png)

### **Security**

**IP Security Lookup**

Security insights such as proxy, VPN, TOR, Relay, threat score, cloud and proxy providers.

* Input: IP Address  
* Outputs: security.is\_proxy, security.is\_vpn, security.is\_relay, security.is\_tor, security.threat\_score, security.is\_cloud\_provider and [more](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference)

**Bulk IP Security Lookup**

Security assessment for multiple IPs.

* Input: Array of IPs  
* Outputs: Collection of [security data](https://ipgeolocation.io/documentation/ip-security-api.html#security-json-object-reference)

![Security API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/security.png)

### **Network Intelligence**

**Lookup ASN**

Provides a simple way to retrieve accurate information about an Autonomous System Number (ASN) and its associated IPv4 and IPv6 address ranges.

* Input: IP or ASN number  
* Outputs: asn.as\_number, asn.organization and [more](https://ipgeolocation.io/documentation/asn-api.html#reference-to-asn-api-response)

![ASN API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/asn.png)

**Lookup Abuse Contact Information**

Includes details such as the role, handle, organization name, kind (e.g., group or individual), and postal address. This information helps identify the entity responsible for handling abuse reports.

* Input: IP Address  
* Outputs: abuse.emails, abuse.kind and [more](https://ipgeolocation.io/documentation/ip-abuse-contact-api.html#reference-to-abuse-contact-api-response)

![Abuse Contact API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/abuse.png)

### **Time Services**

Free Timezone API and Time conversion API provides date and time related information such as current time, date in various formats, week, month, year, time in unix timestamp, UTC/GMT offset and day light saving time from timezone name, any IPv4 or IPv6 address or geolocation coordinates, IATA code, ICAO code, or UN/LOCODE.

**Get Timezone Info**

It can be consumed with the following input variations:

* For a Time Zone Name  
* For any Address (preferrably, city address)  
* For Location Coordinates (latitude & longitude)  
* For any IP address  
* For any IATA code  
* For any ICAO code  
* For any UN/LO Code  
* Outputs: time\_zone.name, time\_zone.current\_time, time\_zone.date\_time\_wti and [more](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-zone-api-response)

![Timezone API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/timezone.png)

**Time Conversion**

Converts a time from one of following options

* Convert Time using Time Zone Names  
* Convert Time using Location  
* Convert Time using Coordinates  
* Convert Time using IATA codes  
* Convert Time using ICAO codes  
* Convert Time using UN/LOCODEs

Output: Converted [date/time](https://ipgeolocation.io/documentation/timezone-api.html#reference-to-time-conversion-api-response)

![Time Conversion API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/time-conversion.png)

### **Astronomy**

Provides timings for sunrise, sunset, moonrise, moonset, sun azimuth, moon azimuth, sun altitude, moon altitude, sun distance from the earth and moon distance from the earth.

**Get Astronomy Details**

* Outputs: astronomy.sunrise, astronomy.sunset, astronomy.moon\_phase and [many more.](https://ipgeolocation.io/documentation/astronomy-api.html#reference-to-astronomy-api-response)

![Astronomy API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/astronomy.png)

**Astronomy Time Series Lookup**

* Outputs: Daily astronomy data for a defined date range.

![Astronomy TimeSeries API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/astronomy-timeseries.png)

### **User-Agent Parsing**

Provides detailed client system information, allowing for the detection of bots, crawlers, and potential attackers.

**Parse User Agent String**

* Input: User agent string  
* Outputs: provides name, device and operating system [information](https://ipgeolocation.io/documentation/user-agent-api.html#reference-to-user-agent-api-response).

![Single UserAgent Lookup API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/single-user-agent.png)

**Parse Bulk User Agent Strings**

* Input: Array of User agent strings  
* Outputs: Collection of parsed user agent objects

![Bulk UserAgent Lookup API](https://static.ipgeolocation.io/web-assets/images/integrations/viasocket/bulk-user-agent.png)
