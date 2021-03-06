# APInalytics

Analytics for APIs.

We built APInalytics because we wanted simple, cheap analytics for an API and we couldn't find a great solution.  Now we've built it we're looking at opening it up to the public.  If you're interested please let us know via our [launchrock page](http://apinalytics.launchrock.com/) - if you're _really_ interested just follow the instructions below and jump in.  If you have any comments please let us know via this project's [issues](https://github.com/apinalytics/apinalytics/issues).  If you want to hear how we get on [follow us on twitter](https://twitter.com/apinalytics).

Unlike other API analytics providers we don't host your API or require you to run your API queries through our proxies. You simply insert a little bit of server-side code to report events to us. The APInalytics API (very meta) lets you retrieve data about those events in all sorts of ways, so you can build your own dashboard - or use our example one.

APInalytics is free to use in this trial phase, although we do intend this to be a commercial project and to start charging something for it eventually. We hope to provide a cost-effective API solution, and there will likely be a free tier. Your data is yours, and if you decide the paid service (when we launch it) isn't for you, you'll be able to extract the data you've already stored for free. You can help shape our pricing and business by responding to our [survey](http://goo.gl/forms/hMVmYKCKeU).

## API
Apinalytics is analytics for APIs, and you access it via APIs.  There's an API for sending events and an API for querying the data.  To make things easier we've supplied client event sending API for [go](https://github.com/apinalytics/apinalytics_client) ([Sorry](https://twitter.com/iamdevloper/status/524976026313826304)) (for more languages and frameworks see [here](https://github.com/apinalytics/apinalytics/wiki/Sending-Events---clients-for-different-languages) ) and an [example html dashboard](https://github.com/apinalytics/apinalytics_dashboard) that you can run locally and edit.

### Identity and authentication
To get started go to http://apinalytics.tanktop.tv/u/ and login with your GitHub account.  This will generate an Application ID, a _write_ key and a _read_ key.  Include the Application ID in an X-Auth-User header in all API requests.  When you send events include the _write_ key in an X-Auth-Key header.  When querying events include the _read_ key in the X-Auth-Key header.

### Sending events
Send events by POSTing JSON to the following URL.  You'll need to set Content-Type to "application/json".

`POST http://apinalytics.tanktop.tv/1/event/`

The JSON must be an array of objects with the following fields.


| Field | Description |
|---|---|
| timestamp | Event UNIX epoch timestamp in seconds (seconds since 1st Jan 1970 UTC) |
| consumer_id | A string that identifies the consumer of your API |
| method | The method used for the API request (e.g. POST, GET, DELETE, etc) |
| url | The full URL for the API request with parameters |
| function | The name of the function that the request invoked.|
| response_us | Request response time in microseconds |
| status_code | Request response code as an integer, e.g. 200 for success |

In the future we may add the ability to include arbitrary additional fields.

### The Query API
Apinalytics include an API that allows you to query your event data as time series.  You can group and aggregate your data in various ways. Or you can cheat and use our [example html dashboard](https://github.com/apinalytics/apinalytics_dashboard).

The query URL is as follows.  Start and end times are in seconds since 1st January 1970 UTC (Unix epoch).

`GET http://apinalytics.tanktop.tv/1/timeseries/<start time>/<end time>/`

The available parameters are as follows.

| Param | Default | Description |
| ---    | ---      | ---          |
| value | "response_us"| The name of the value to aggregate.  This must be the name of a numeric field.  At the moment only "response_us" is valid |
| granularity | omitting granularity results in all data in the time period being aggregated in a single time bucket | The granularity of the aggregation. Can be specified in seconds, or can be "second", "minute", "hour", "day", "week" |
| aggregation | "count" | Aggregation function to apply.  Values include "count", "mean", "min", "max", "sum".  Also "percentile_X", where X is a number from 1 to 99 |
| group | omitting the group parameter results in a single group | Specify the name of the field to group results by. Possible values are "consumer_id", "method", "url", "function", "status_code" |
| include | filter the response.  E.g. include=method:GET reduces the result to just include GET methods.  Can be repeated.  |

The default aggregation is "count".  So, for example, you can easily count the number of GET, POST, DELETE, etc., requests that your service processes each minute.


#### Example requests
To count the total number of API requests in a period.

`GET /1/timeseries/1412605036/1412691436/`

To break that count down by hour.

`GET /1/timeseries/1412605036/1412691436/?granularity=hour`

To get the count for each of your api's consumers.

`GET /1/timeseries/1412605036/1412691436/?granularity=hour&group=consumer_id`

To get the 95th percentile response time in each hour for each consumer.

`GET /1/timeseries/1412605036/1412691436/?granularity=hour&group=consumer_id&aggregation=percentile_95`

To see the average response time each hour of each function

`GET /1/timeseries/1412605036/1412691436/?granularity=hour&group=function&aggregation=mean`

#### Response format
Query responses are in JSON.  Here is an example, with data grouped by function.

```json
{
    "groups": {
        "actorlist":[[1413378000,3]],
        "full_list":[[1413378000,3]]},
    "start":1413295200,
    "end":1413385200,
    "granularity":3600000
}
```
The fields are as follows

| field name  | description |
|---|---|
|groups | A dictionary of time series.  The keys are the items the data is grouped by.  For example if you use group=method, the dictionary keys will be "GET", "PUT", "POST", etc.  The values are the timeseries data, as <timestamp, value> pairs |
| start | Start of the time period the response represents. Time is in seconds since 1 Jan 1970 UTC (UNIX epoch) |
| end | End of the time period the response represents. |
| granularity | Time bucket size that the data is aggregated into.  In milliseconds |

If you do not specify group, then you will receive a single group with the key being your application ID. If you do not specify granularity then each time series will contain a single entry with the timestamp being the start of the period queried.

