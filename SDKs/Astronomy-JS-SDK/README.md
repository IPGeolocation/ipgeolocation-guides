# Astronomy API Client Side Plugin 

## Table of Contents
1. [Requirements](#requirements)
2. [How to Get Your API Key](#how-to-get-your-api-key)
3. [Installation](#installation)
4. [Plugin Configurations](#plugin-configurations)
5. [Code Examples](#code-examples)
   - [Get Astronomical Information for a specific location](#get-astronomical-information-for-a-specific-location)
   - [Get Astronomical Information for Location Coordinates](#get-astronomical-information-for-location-coordinates)
   - [Get Astronomical Information for an IP Address](#get-astronomical-information-for-an-ip-address)
   - [Get Astronomical Information for a Specific Date](#get-astronomical-information-for-a-specific-date)
6. [Error Handling](#error-handling)

## Requirements

- Python 3.9+
- API Key from [IPGeolocation.io](https://ipgeolocation.io)

## How to Get Your API Key

1. **Sign up** here: [https://app.ipgeolocation.io/signup](https://app.ipgeolocation.io/signup)
2. **(optional)** Verify your email, if you signed up using email.
3. **Log in** to your account: [https://app.ipgeolocation.io/login](https://app.ipgeolocation.io/login)
4. After logging in, navigate to your **Dashboard** to find your API key: [https://app.ipgeolocation.io/dashboard](https://app.ipgeolocation.io/dashboard)

## Installation
To access this service, add the following Javascript call (usually within `head` block of your pages).
```html
<script language="JavaScript" src="https://static.ipgeolocation.io/astronomy-api-plugin.js" type="text/javascript"></script>
```

> [!NOTE]
> This service will only work when embedded in web pages - no server-side calls will work.

## Plugin Configurations
To instantiate the AstronomyAPI, you can use the following configuration options:
- `apiKey (optional: string)`: API key used for request authentication. This field is mandatory unless you have added your request origin in the dashboard, in which case it becomes optional.
- `ipAddress (optional: string)`: Specify an IP address for which you want the astronomical information. If not provided, the client’s IP address is used by default.
- `location (optional: string)`: Specify the location for which you want the astronomical information.
- `Coordinates (optional: string)`: Specify the location coordinates for which you want the astronomical information.
- `date (optional: string)`: Specify the date for which you want the astronomical information. You can pass this along with other parameters (ipAddress, location, coordinates)
- `lang (optional: boolean)`: Language of the response. Default is en. Supported languages include: ru, de, ja, fr, cn, es, cs, it, fa, ko.
- `saveToSessionStorage (optional: boolean)`: Saves geolocation data to session storage for temporary use.

> [!TIP]
> For detailed documentation for the Astronomy API, please visit [https://ipgeolocation.io/astronomy-api.html#documentation-overview](https://ipgeolocation.io/astronomy-api.html#documentation-overview).

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
    "country": "United States",
    "state": "New York",
    "city": "New York",
    "locality": "Clinton",
    "latitude": 40.76473335,
    "longitude": -74.00083980660943
  },
  "date": "2024-11-04",
  "current_time": "08:13:22.978",
  "sunrise": "06:30",
  "sunset": "16:48",
  "sun_status": "-",
  "solar_noon": "11:39",
  "day_length": "10:18",
  "sun_altitude": 16.120293371563754,
  "sun_distance": 148361706.39355108,
  "sun_azimuth": 128.26575101712496,
  "moonrise": "09:49",
  "moonset": "18:29",
  "moon_status": "-",
  "moon_altitude": -14.81836182838389,
  "moon_distance": 396331.25698682835,
  "moon_azimuth": 113.85094239453929,
  "moon_parallactic_angle": -51.40390182984849,
  "moon_phase": "WAXING_CRESCENT",
  "moon_illumination_percentage": "8.43",
  "moon_angle": 33.74826084504092
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
            ip: "8.8.8.8"
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
            location: "New York, US"
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
    "date": "2026-01-01",
    "current_time": "10:21:09.529",
    "mid_night": "23:59",
    "night_end": "05:41",
    "morning": {
      "astronomical_twilight_begin": "05:41",
      "astronomical_twilight_end": "06:15",
      "nautical_twilight_begin": "06:15",
      "nautical_twilight_end": "06:49",
      "civil_twilight_begin": "06:49",
      "civil_twilight_end": "07:19",
      "blue_hour_begin": "06:37",
      "blue_hour_end": "07:01",
      "golden_hour_begin": "07:01",
      "golden_hour_end": "08:03"
    },
    "sunrise": "07:19",
    "sunset": "16:40",
    "evening": {
      "golden_hour_begin": "15:55",
      "golden_hour_end": "16:58",
      "blue_hour_begin": "16:58",
      "blue_hour_end": "17:21",
      "civil_twilight_begin": "16:40",
      "civil_twilight_end": "17:10",
      "nautical_twilight_begin": "17:10",
      "nautical_twilight_end": "17:44",
      "astronomical_twilight_begin": "17:44",
      "astronomical_twilight_end": "18:17"
    },
    "night_begin": "18:17",
    "sun_status": "-",
    "solar_noon": "11:59",
    "day_length": "09:20",
    "sun_altitude": 22.285602788421738,
    "sun_distance": 147102938.88036564,
    "sun_azimuth": 155.50096343869086,
    "moon_phase": "WAXING_GIBBOUS",
    "moonrise": "14:31",
    "moonset": "05:42",
    "moon_status": "-",
    "moon_altitude": -21.431610505859357,
    "moon_distance": 360835.8162008277,
    "moon_azimuth": 2.859017413960146,
    "moon_parallactic_angle": -2.4464836911657195,
    "moon_illumination_percentage": "95.35",
    "moon_angle": 155.1019848933602
  }
}
```

## Error Handling
Inspect the status field in the response to detect any errors. A missing status field typically indicates a successful request, while its presence signals an issue. For example, if you are trying to fetch the data for a location that does not exist, you will get the following error response:

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

