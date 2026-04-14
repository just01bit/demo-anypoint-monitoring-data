# demo-anypoint-monitoring-data

## Overview
Up to 14th April 2026, for managed flex gateway, the log forwardinng to external is not available yet. This is on MuleSoft roadmap.
As a workaround, for MuleSoft Integration Advanced or Titanium customers, you can leverage the platform API - [Anypoint Monitoring Archive API](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-23abe036327a.anypoint-platform/anypoint-monitoring-archive-api/) to extract metrics data like logs from Anypoinnt Monitoring.

This PoC shows the steps to call the various endpoints of the platform API to retrieve logs for a specific managed flex gateway instance.

## Workflow

The application starts from a HTTP Listener, endpoint is /data, method: GET

The input payload is like below:
```json
{
    "orgId": "{YOUR ANYPOINT ORG ID}",
    "envId": "{YOUR ENV ID}",
    "entityType": "flex",
    "entityId": "{YOUR FLEX GATEWAY ENTITY ID}",
    "fileType": "logs",
    "date": "2026-03-30"
}
```

- Form the base URL as: https://monitoring.anypoint.mulesoft.com/monitoring/archive/api/v1/organizations/{orgId}/environments/{envId}/{entityType} and save it to variable: "baseURL"
- Replace "orgId", "envId", "entityType" with "orgId", "envId", "entityType" from the input payload.
- Save "entityId" and "fileType" into variables: "entityId", "fileType".
- For "date", break it to "year", "month" and "day" into variables: "year", "month" and "day".

1. When running the app, the anypoint platform connected app client id and secret are provided via mule runtime args: connectedAppClientId and connectedAppClientSecret
1.1 Firstly, the app makes a POST call to https://anypoint.mulesoft.com/accounts/api/v2/oauth2/token by passing the connected app's client id and secret to the payload to get the access token.
1.2 The payload to the token endpoint is:
```jsonn
{
    "client_id": "{connectedAppClientId}",
    "client_secret": "{connectedAppClientSecret}",
    "grant_type": "client_credentials"
}
```
1.3 In the response, the access token is from "access_token"
1.4 For all the following APIs call, we will use the same bearer token authentication method and pass the "access_token"

2. The app calls the GET method to the saved variable "baseURL" to get the list of entity identifiers
2.1 The response payload is like below:
```json
{
    "resources": [
        {
            "id": "00b7cbfc-f68f-4d0a-b443-298d7cbceffc_demo-managed-flex-79d5bd78b-d84r4.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "id": "00b7cbfc-f68f-4d0a-b443-298d7cbceffc_demo-managed-flex-79d5bd78b-qnqsw.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "id": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "id": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        }
    ]
}
```
2.2 Use a Transformation to filter the payload by the saved variable "entityId" to get the relevant id list. For instance, we pass "entityId" as "7886bc31-5215-457e-b343-51bb5548f25d", so after this step, we should get the resources id list as below:
```json
{
    "resources": [
        {
            "id": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "id": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        }
    ]
}
```
3. Loop resource id from the above, and call the GET method to: {baseURL}/{resourceId}/{fileType}/{year}/{month}/{day} to get the list of data files for that resource id and the specified date.
3.1 As from the previous step 2, it returns 2 resources id, therefore, we loop each id and get its relevant response like below:
3.1.1 For resource id - "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5":
```json
{
    "resources": [
        {
            "id": "001796ff-fc6a-44be-bd77-94df9eacffb0.2026-03-30T11.43.part70.txt",
            "time": "2026-03-30T11:53:07Z",
            "size": 1028
        },
        {
            "id": "003c49ae-7473-4802-ab73-d7246c4e59ab.2026-03-30T04.01.part24.txt",
            "time": "2026-03-30T04:11:10Z",
            "size": 1478
        },
        {
            "id": "007b279f-a7e4-42ac-ab29-7c4c9905c39d.2026-03-30T07.41.part46.txt",
            "time": "2026-03-30T07:51:57Z",
            "size": 1178
        }
    ]
}
```
3.1.2 For resource id - "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
```json
{
    "resources": [
        {
            "id": "001ff8a0-ca35-407f-a980-de003e1ca063.2026-03-30T02.10.part13.txt",
            "time": "2026-03-30T02:20:56Z",
            "size": 2121
        },
        {
            "id": "005ebdb5-1e63-4766-a2b2-24347508ce05.2026-03-30T16.03.part96.txt",
            "time": "2026-03-30T16:13:46Z",
            "size": 1525
        },
        {
            "id": "010d8913-fab8-4142-8641-814da70b7973.2026-03-30T10.13.part61.txt",
            "time": "2026-03-30T10:23:06Z",
            "size": 1330
        }
    ]
}
```
3.2 As we can see, in the response, each data has "id" and "time", let's combine all records from all resources and sorted them by "time". 
3.2.1 "id" mapps to "fileName", "time" to "time". 
3.2.2 For each combined record, we also append its "resourceId", as we need this for the next API call.
3.2.3 So the result looks like:
```json
{
    "data": [
        {
            "fileName": "001ff8a0-ca35-407f-a980-de003e1ca063.2026-03-30T02.10.part13.txt",
            "time": "2026-03-30T02:20:56Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "fileName": "003c49ae-7473-4802-ab73-d7246c4e59ab.2026-03-30T04.01.part24.txt",
            "time": "2026-03-30T04:11:10Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "fileName": "007b279f-a7e4-42ac-ab29-7c4c9905c39d.2026-03-30T07.41.part46.txt",
            "time": "2026-03-30T07:51:57Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "fileName": "010d8913-fab8-4142-8641-814da70b7973.2026-03-30T10.13.part61.txt",
            "time": "2026-03-30T10:23:06Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "fileName": "001796ff-fc6a-44be-bd77-94df9eacffb0.2026-03-30T11.43.part70.txt",
            "time": "2026-03-30T11:53:07Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-qlsr8.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        },
        {
            "fileName": "005ebdb5-1e63-4766-a2b2-24347508ce05.2026-03-30T16.03.part96.txt",
            "time": "2026-03-30T16:13:46Z",
            "resourceId": "7886bc31-5215-457e-b343-51bb5548f25d_demo-flex-2025-6b58dc7c4-6lp97.b567e4ea-9236-49ee-b7b2-dc4d5da65ed5"
        }                                
    ]
}
``` 
4 Loop each record from the result of step 3, and call GET method to the endpoint: {baseURL}/{resourceId}/{fileType}/{year}/{month}/{day}/{fileName}
4.1 append the response into the local file: /Downloads/flex_log.txt

## Considerations
1 Decoupled Processing Using Anypoint MQ

In this PoC, the log file download is not handled within the same flow. Instead, Anypoint MQ is used to decouple the process and improve reliability.

- The combined records generated in step 3.2.3 are published to a FIFO queue.
- A separate flow consumes messages from this queue and performs the log file download independently.

2 Handling Platform API Rate Limits

The Platform API enforces rate limits (e.g., 60 requests per minute). Depending on the volume of log files to extract, you may encounter errors such as **“429 – Too Many Requests.”**

To mitigate this:

- Implement a retry mechanism (e.g., retry scope with backoff strategy).
- Create an API instance for the base URL and apply traffic management policies such as spike control to regulate request throughput.
