# Apinalytics

Analytics for APIs


## API
Apinalytics is analytics for APIs, and you access it via APIs.  There's an API for sending events and for querying the data.  To make things easier we've supplied client event sending API for go (more languages and frameworks to follow) and an example html dashboard that you can run locally and edit.

### Identity and authentication
To get started go to http://apinalytics.tanktop.tv/u/ and login with your github account.  This will generate an Application ID, a _write_ key and a _read_ key.  Include the Application ID in an X-Auth-User header in all API requests.  When you send events include the _write_ key in an X-Auth-Key header.  When querying events include the _read_ key in the X-Auth-Key header.

### Sending events
Send events by POSTing JSON to the following URL.  You'll need to set Content-Type to "application/json".

`POST /1/events/`

The JSON must be an array of objects with the following fields

| Field | Description |
|-- | --|
| timestamp | Event UNIX epoch timestamp in seconds (seconds since 1st Jan 1970 UTC) |
| consumer_id | A string that identifies the consumer of your API |
| method | The method used for the API request (e.g. POST, GET, DELETE, etc) |
| url | The full URL for the API request with parameters |
| function | The name of the function that the request invoked.|
| response_us | Request response time in microseconds |
| status_code | Request response code as an integer, e.g. 200 for success |

In the future we may add the ability to include arbitrary additional fields.
