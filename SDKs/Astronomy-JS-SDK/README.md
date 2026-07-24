# Astronomy API Client Side Plugin 

## Overview
Get sunrise, sunset, moonrise, moonset, moon phases with precise twilight period times in combination with location information. One can get the [astronomy information](https://ipgeolocation.io/astronomy-api.html) from any IP address, geo-coordinates, or street address right in your web application. It is recommended to use the request origin feature (available only in paid plans) to ensure your API key is not exposed publicly. You can add your request origin (your website domain) by logging into your account at [IPGeolocation.io](https://app.ipgeolocation.io/login).

## Table of Contents
1. [Overview](#overview) 
2. [Requirements](#requirements)
3. [How to Get Your API Key](#how-to-get-your-api-key)
4. [Installation](#installation)
5. [Plugin Configurations](#plugin-configurations)
6. [Code Examples](#code-examples)
   - [Get Astronomical Information for a specific location](#get-astronomical-information-for-a-specific-location)
   - [Get Astronomical Information for Location Coordinates](#get-astronomical-information-for-location-coordinates)
   - [Get Astronomical Information for an IP Address](#get-astronomical-information-for-an-ip-address)
   - [Get Astronomical Information for a Specific Date](#get-astronomical-information-for-a-specific-date)
   - [Get Astronomical Event Times in a Specific Time Zone](#get-astronomical-event-times-in-a-specific-time-zone)
   - [Get Astronomical Information for a Date Range](#get-astronomical-information-for-a-date-range)
7. [Error Handling](#error-handling)

## Requirements

- Internet Connection
- API Key from [IPGeolocation.io](https://ipgeolocation.io)

## How to Get Your API Key

1. **Sign up** here: [https://app.ipgeolocation.io/signup](https://app.ipgeolocation.io/signup)
2. **(optional)** Verify your email, if you signed up using email.
3. **Log in** to your account: [https://app.ipgeolocation.io/login](https://app.ipgeolocation.io/login)
4. After logging in, navigate to your **Dashboard** to find your API key: [https://app.ipgeolocation.io/dashboard](https://app.ipgeolocation.io/dashboard)

## Installation
To access this service, add the following Javascript call (usually within `head` block of your pages).
```html
<script language="JavaScript" src="https://static.ipgeolocation.io/astronomy-api-plugin.v3.0.0.js" type="text/javascript"></script>
```

> [!NOTE]
> This service will only work when embedded in web pages - no server-side calls will work.

## Plugin Configurations
To instantiate the AstronomyAPI, you can use the following configuration options:
- `apiKey (optional: string)`: API key used for request authentication. This field is mandatory unless you have added your request origin in the dashboard, in which case it becomes optional.
- `ipAddress (optional: string)`: Specify an IP address for which you want the astronomical information. If not provided, the client’s IP address is used by default.
- `location (optional: string)`: Specify the location for which you want the astronomical information.
- `lat (optional: number)`: Latitude of the location. Provide together with `long`.
- `long (optional: number)`: Longitude of the location. Provide together with `lat`.
- `elevation (optional: number)`: Elevation of the location in meters. The maximum supported value is 10,000 meters.
- `date (optional: string)`: Date for a single lookup in YYYY-MM-DD format. You can pass it with `ipAddress`, `location`, or `lat` and `long`.
- `timeZone (optional: string)`: IANA time zone, such as `Europe/London`, used to convert returned event times.
- `dateStart (optional: string)`: Start date for a time-series lookup in YYYY-MM-DD format.
- `dateEnd (optional: string)`: End date for a time-series lookup in YYYY-MM-DD format. The maximum time-series range is 90 days.
- `lang (optional: string)`: Language of the response. Default is en. Supported languages include: ru, de, ja, fr, cn, es, cs, it, fa, ko, pt, ar.
- `saveToSessionStorage (optional: boolean)`: Caches successful responses in session storage for the current browser tab. Cache entries are specific to the lookup parameters.

> [!TIP]
> For detailed documentation for the Astronomy API, please visit [https://ipgeolocation.io/astronomy-api.html#documentation-overview](https://ipgeolocation.io/astronomy-api.html#documentation-overview).

Use `getAstronomy()` for a single-date lookup. Use `getAstronomyTimeSeries()` with `dateStart` and `dateEnd` for a date range.

## Code Examples
### Get Astronomical Information for a specific location
```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            location: "New York, US"
        });

        const resp = await astronomyAPI.getAstronomy();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "location": {
    "location_string": "New York, US",
    "country_name": "United States",
    "state_prov": "New York",
    "city": "New York",
    "locality": "Clinton",
    "latitude": "40.76473",
    "longitude": "-74.00084",
    "elevation": "9"
  },
  "astronomy": {
    "date": "2024-11-04",
    "current_time": "08:12:44.883",
    "mid_night": "23:39",
    "night_end": "04:57",
    "morning": {
      "astronomical_twilight_begin": "04:57",
      "astronomical_twilight_end": "05:29",
      "nautical_twilight_begin": "05:29",
      "nautical_twilight_end": "06:01",
      "civil_twilight_begin": "06:01",
      "civil_twilight_end": "06:30",
      "blue_hour_begin": "05:51",
      "blue_hour_end": "06:13",
      "golden_hour_begin": "06:13",
      "golden_hour_end": "07:10"
    },
    "sunrise": "06:30",
    "sunset": "16:48",
    "evening": {
      "golden_hour_begin": "16:08",
      "golden_hour_end": "17:05",
      "blue_hour_begin": "17:05",
      "blue_hour_end": "17:27",
      "civil_twilight_begin": "16:48",
      "civil_twilight_end": "17:16",
      "nautical_twilight_begin": "17:16",
      "nautical_twilight_end": "17:49",
      "astronomical_twilight_begin": "17:49",
      "astronomical_twilight_end": "18:21"
    },
    "night_begin": "18:21",
    "sun_status": "-",
    "solar_noon": "11:39",
    "day_length": "10:18",
    "sun_altitude": 16.025928084156632,
    "sun_distance": 148361706.3935511,
    "sun_azimuth": 128.14060120476182,
    "moon_phase": "WAXING_CRESCENT",
    "moonrise": "09:48",
    "moonset": "18:29",
    "moon_status": "-",
    "moon_altitude": -14.923954777733469,
    "moon_distance": 396332.621613144,
    "moon_azimuth": 113.76283054867883,
    "moon_parallactic_angle": -51.452160801531605,
    "moon_illumination_percentage": "8.42",
    "moon_angle": 33.74321660247666
  }
}
```
If there is some error while fetching the response from the API, following response will be returned, so handle it accordingly:
```json
{
  "error_status": "error_status_code",
  "error_message": "error_message"
}
```

### Get Astronomical Information for Location Coordinates
```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            lat: -27.4748,
            long: 153.017,
        });

        const resp = await astronomyAPI.getAstronomy();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "location": {
    "latitude": "-27.47480",
    "longitude": "153.01700",
    "country_name": "Australia",
    "state_prov": "Queensland",
    "city": "South Brisbane",
    "locality": "South Brisbane",
    "elevation": ""
  },
  "astronomy": {
    "date": "2025-12-05",
    "current_time": "00:48:29.414",
    "mid_night": "23:38",
    "night_end": "03:13",
    "morning": {
      "astronomical_twilight_begin": "03:13",
      "astronomical_twilight_end": "03:47",
      "nautical_twilight_begin": "03:47",
      "nautical_twilight_end": "04:18",
      "civil_twilight_begin": "04:18",
      "civil_twilight_end": "04:44",
      "blue_hour_begin": "04:08",
      "blue_hour_end": "04:28",
      "golden_hour_begin": "04:28",
      "golden_hour_end": "05:18"
    },
    "sunrise": "04:44",
    "sunset": "18:32",
    "evening": {
      "golden_hour_begin": "17:58",
      "golden_hour_end": "18:48",
      "blue_hour_begin": "18:48",
      "blue_hour_end": "19:09",
      "civil_twilight_begin": "18:32",
      "civil_twilight_end": "18:58",
      "nautical_twilight_begin": "18:58",
      "nautical_twilight_end": "19:30",
      "astronomical_twilight_begin": "19:30",
      "astronomical_twilight_end": "20:03"
    },
    "night_begin": "20:03",
    "sun_status": "-",
    "solar_noon": "11:38",
    "day_length": "13:47",
    "sun_altitude": -37.39827596244138,
    "sun_distance": 147449905.5742523,
    "sun_azimuth": 159.44581819516998,
    "moon_phase": "FULL_MOON",
    "moonrise": "19:06",
    "moonset": "04:20",
    "moon_status": "-",
    "moon_altitude": 31.398400021514547,
    "moon_distance": 357494.20244737004,
    "moon_azimuth": 334.9275491792954,
    "moon_parallactic_angle": 155.16199342731318,
    "moon_illumination_percentage": "99.81",
    "moon_angle": 174.9439622483671
  }
}
```

### Get Astronomical Information for an IP Address
```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            ipAddress: "8.8.8.8"
        });

        const resp = await astronomyAPI.getAstronomy();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "ip": "8.8.8.8",
  "location": {
    "continent_code": "NA",
    "continent_name": "North America",
    "country_code2": "US",
    "country_code3": "USA",
    "country_name": "United States",
    "country_name_official": "United States of America",
    "is_eu": false,
    "state_prov": "California",
    "state_code": "US-CA",
    "district": "Santa Clara",
    "city": "Mountain View",
    "zipcode": "94043-1351",
    "latitude": "37.42240",
    "longitude": "-122.08421",
    "locality": "Charleston Terrace",
    "elevation": "3"
  },
  "astronomy": {
    "date": "2025-07-18",
    "current_time": "03:06:31.046",
    "mid_night": "01:14",
    "night_end": "04:14",
    "morning": {
      "astronomical_twilight_begin": "04:14",
      "astronomical_twilight_end": "04:54",
      "nautical_twilight_begin": "04:54",
      "nautical_twilight_end": "05:32",
      "civil_twilight_begin": "05:32",
      "civil_twilight_end": "06:01",
      "blue_hour_begin": "05:20",
      "blue_hour_end": "05:43",
      "golden_hour_begin": "05:43",
      "golden_hour_end": "06:39"
    },
    "sunrise": "06:01",
    "sunset": "20:27",
    "evening": {
      "golden_hour_begin": "19:49",
      "golden_hour_end": "20:45",
      "blue_hour_begin": "20:45",
      "blue_hour_end": "21:09",
      "civil_twilight_begin": "20:27",
      "civil_twilight_end": "20:56",
      "nautical_twilight_begin": "20:56",
      "nautical_twilight_end": "21:34",
      "astronomical_twilight_begin": "21:34",
      "astronomical_twilight_end": "22:14"
    },
    "night_begin": "22:14",
    "sun_status": "-",
    "solar_noon": "13:14",
    "day_length": "14:25",
    "sun_altitude": -25.97691496384962,
    "sun_distance": 152051279.32455352,
    "sun_azimuth": 29.157153500526988,
    "moon_phase": "LAST_QUARTER",
    "moonrise": "00:23",
    "moonset": "14:34",
    "moon_status": "-",
    "moon_altitude": 31.341933591493504,
    "moon_distance": 370142.7870275993,
    "moon_azimuth": 94.69006597321885,
    "moon_parallactic_angle": -55.06382574132873,
    "moon_illumination_percentage": "-45.47",
    "moon_angle": 275.1982227377768
  }
}
```

### Get Astronomical Information for a Specific Date
```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            location: "New York, US",
            date: "2024-11-04"
        });

        const resp = await astronomyAPI.getAstronomy();
        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "location": {
    "location_string": "New York, US",
    "country_name": "United States",
    "state_prov": "New York",
    "city": "New York",
    "locality": "Clinton",
    "latitude": "40.76473",
    "longitude": "-74.00084",
    "elevation": "9"
  },
  "astronomy": {
    "date": "2024-11-04",
    "current_time": "08:12:44.883",
    "mid_night": "23:39",
    "night_end": "04:57",
    "morning": {
      "astronomical_twilight_begin": "04:57",
      "astronomical_twilight_end": "05:29",
      "nautical_twilight_begin": "05:29",
      "nautical_twilight_end": "06:01",
      "civil_twilight_begin": "06:01",
      "civil_twilight_end": "06:30",
      "blue_hour_begin": "05:51",
      "blue_hour_end": "06:13",
      "golden_hour_begin": "06:13",
      "golden_hour_end": "07:10"
    },
    "sunrise": "06:30",
    "sunset": "16:48",
    "evening": {
      "golden_hour_begin": "16:08",
      "golden_hour_end": "17:05",
      "blue_hour_begin": "17:05",
      "blue_hour_end": "17:27",
      "civil_twilight_begin": "16:48",
      "civil_twilight_end": "17:16",
      "nautical_twilight_begin": "17:16",
      "nautical_twilight_end": "17:49",
      "astronomical_twilight_begin": "17:49",
      "astronomical_twilight_end": "18:21"
    },
    "night_begin": "18:21",
    "sun_status": "-",
    "solar_noon": "11:39",
    "day_length": "10:18",
    "sun_altitude": 16.025928084156632,
    "sun_distance": 148361706.3935511,
    "sun_azimuth": 128.14060120476182,
    "moon_phase": "WAXING_CRESCENT",
    "moonrise": "09:48",
    "moonset": "18:29",
    "moon_status": "-",
    "moon_altitude": -14.923954777733469,
    "moon_distance": 396332.621613144,
    "moon_azimuth": 113.76283054867883,
    "moon_parallactic_angle": -51.452160801531605,
    "moon_illumination_percentage": "8.42",
    "moon_angle": 33.74321660247666
  }
}
```

### Return Astronomical Event Times in a Specific Time Zone
The API performs the astronomical calculation for the resolved location and converts event times to the requested time zone. Converted events include a date because some events can fall on the previous or next calendar day.

```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            ipAddress: "115.186.118.130",
            date: "2026-03-07",
            timeZone: "Europe/London"
        });

        const resp = await astronomyAPI.getAstronomy();

        if (!resp.error_message) {
            console.log(resp.astronomy.time_zone);
            console.log(resp.astronomy.sunrise);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response (excerpt):
```json
{
  "location": {
    "country_name": "Pakistan",
    "state_prov": "Punjab",
    "city": "Faisalabad",
    "latitude": "31.45037",
    "longitude": "73.13496",
    "elevation": "190"
  },
  "astronomy": {
    "time_zone": "Europe/London",
    "date": "2026-03-07",
    "sunrise": "2026-03-07 01:25",
    "sunset": "2026-03-07 13:11",
    "solar_noon": "2026-03-07 07:18"
  }
}
```

### Get Astronomical Information for a Date Range
Use `getAstronomyTimeSeries()` to retrieve daily Sun and Moon events for up to 90 days. In a time-series response, `astronomy` is an array. Date-level entries do not include current-time position fields such as `current_time`, `sun_altitude`, or `moon_altitude`.

```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            lat: 40.76473,
            long: -74.00084,
            dateStart: "2025-06-16",
            dateEnd: "2025-06-18"
        });

        const resp = await astronomyAPI.getAstronomyTimeSeries();

        if (!resp.error_message) {
            resp.astronomy.forEach((day) => {
                console.log(day.date, day.sunrise, day.sunset);
            });
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response (excerpt):
```json
{
  "location": {
    "latitude": "40.76473",
    "longitude": "-74.00084",
    "country_name": "United States",
    "state_prov": "New York",
    "city": "New York",
    "locality": "Midtown West",
    "elevation": "9"
  },
  "astronomy": [
    {
      "date": "2025-06-16",
      "mid_night": "00:56",
      "night_end": "03:18",
      "morning": {
        "astronomical_twilight_begin": "03:18",
        "astronomical_twilight_end": "04:08",
        "nautical_twilight_begin": "04:08",
        "nautical_twilight_end": "04:50",
        "civil_twilight_begin": "04:50",
        "civil_twilight_end": "05:23",
        "blue_hour_begin": "04:37",
        "blue_hour_end": "05:04",
        "golden_hour_begin": "05:04",
        "golden_hour_end": "06:05"
      },
      "sunrise": "05:23",
      "sunset": "20:30",
      "evening": {
        "golden_hour_begin": "19:48",
        "golden_hour_end": "20:49",
        "blue_hour_begin": "20:49",
        "blue_hour_end": "21:16",
        "civil_twilight_begin": "20:30",
        "civil_twilight_end": "21:03",
        "nautical_twilight_begin": "21:03",
        "nautical_twilight_end": "21:45",
        "astronomical_twilight_begin": "21:45",
        "astronomical_twilight_end": "22:36"
      },
      "night_begin": "22:36",
      "sun_status": "-",
      "solar_noon": "12:56",
      "day_length": "15:06",
      "moon_phase": "WANING_GIBBOUS",
      "moonrise": "-:-",
      "moonset": "10:28",
      "moon_status": "-"
    }
  ]
}
```

## Error Handling
Inspect the error_status field in the response to detect any errors. A missing error_status field typically indicates a successful request, while its presence signals an issue. HTTP errors return the API status code. Network and response parsing failures return `error_status: 0`. For example, if you are trying to fetch data using invalid coordinates, you will get the following error response:

```html
<script>
    (async () => {
        const astronomyAPI = new AstronomyAPI({
            apiKey: "YOUR_API_KEY",
            lat: -127.4748,
            long: 53.017,
            date: "2024-11-04"
        });

        const resp = await astronomyAPI.getAstronomy();

        if (!resp.error_message) {
            console.log(resp);
        } else {
            console.log("Something went wrong while fetching data", resp);
        }
    })();
</script>
```
Sample Response:
```json
{
  "error_message": "'latitude' (-127.4748) or 'longitude' (53.017) is not valid. 'latitude' must be between -90.0 and +90.0 and 'longitude' must be between -180.0 and +180.0.",
  "error_status": 400
}
```

