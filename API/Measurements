# Measurements

Once you have an access token (see Authentication). You can then get measurements from the API:

```http
GET /api/v1/measurements HTTP/1.1
Host: david.cmh.edu
Authorization: Bearer {ACCESS_TOKEN}
Accept: application/vnd.api+json
```

## Fetching Relationships

See the [json:api](http://jsonapi.org/format/#fetching-relationships) spec for details

## Filters

Since we use [JSONAPI .Net Core](https://github.com/Research-Institute/json-api-dotnet-core) you can 
use any of the supported filters to query the data. Some examples include:

```http
/api/v1/measurements?filter[patient-id]=1&filter[data-type-id]=1
/api/v1/measurements?filter[device-time]=gt:2016-1-1&filter[device-time]=lt:2017-1-1
```
