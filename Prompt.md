
## Objective

Build a MuleSoft application that:

1. Accepts an HTTP GET request
2. Retrieves monitoring archive data from Anypoint APIs
3. Filters, aggregates, sorts, and downloads log files
4. Writes combined results to a local file

---

## 1. Input

### HTTP Listener

* **Method:** GET
* **Endpoint:** `/data`

### Request Payload

```json
{
  "orgId": "string",
  "envId": "string",
  "entityType": "string",
  "entityId": "string",
  "fileType": "string",
  "date": "YYYY-MM-DD"
}
```

---

## 2. Variable Initialization

### 2.1 Construct Base URL

```
baseURL = https://monitoring.anypoint.mulesoft.com/monitoring/archive/api/v1/organizations/{orgId}/environments/{envId}/{entityType}
```

* Replace placeholders using payload values

### 2.2 Extract Variables

* `entityId` ← payload.entityId
* `fileType` ← payload.fileType

### 2.3 Parse Date

Split `date` into:

* `year`
* `month`
* `day`

---

## 3. Authentication

### 3.1 Input (Runtime Arguments)

* `connectedAppClientId`
* `connectedAppClientSecret`

### 3.2 Token Request

**POST**
`https://anypoint.mulesoft.com/accounts/api/v2/oauth2/token`

#### Payload:

```json
{
  "client_id": "{connectedAppClientId}",
  "client_secret": "{connectedAppClientSecret}",
  "grant_type": "client_credentials"
}
```

### 3.3 Output

* Extract `access_token`
* Use as `Bearer Token` for all subsequent requests

---

## 4. Retrieve Resource IDs

### 4.1 API Call

**GET** `{baseURL}`

### 4.2 Response Handling

* Extract `resources[].id`

### 4.3 Filter Logic

* Keep only resources where `id` contains `entityId`

### 4.4 Output

Filtered list of `resourceId`s

---

## 5. Retrieve File Metadata per Resource

### 5.1 Loop Over Each `resourceId`

**GET**

```
{baseURL}/{resourceId}/{fileType}/{year}/{month}/{day}
```

### 5.2 Response Structure

Each response contains:

```json
{
  "resources": [
    {
      "id": "fileName",
      "time": "timestamp",
      "size": number
    }
  ]
}
```

---

## 6. Aggregate and Transform Data

### 6.1 Combine All Responses

Merge all `resources` arrays from all `resourceId`s

### 6.2 Transform Fields

For each record:

* `fileName` ← `id`
* `time` ← `time`
* `resourceId` ← current resourceId

### 6.3 Sort

* Sort all records by `time` (ascending)

### 6.4 Final Structure

```json
{
  "data": [
    {
      "fileName": "string",
      "time": "timestamp",
      "resourceId": "string"
    }
  ]
}
```

---

## 7. Download File Contents

### 7.1 Loop Over Sorted Records

For each record:

**GET**

```
{baseURL}/{resourceId}/{fileType}/{year}/{month}/{day}/{fileName}
```

---

## 8. Output Handling

### 8.1 File Write

* Append each API response to local file:

```
/Users/liang.dai/Downloads/flex_log.txt
```

* Ensure:

  * Append mode (not overwrite)
  * Preserve order from sorted list

---

## 9. Additional Requirements

* Use **Bearer Token authentication** for all API calls after token retrieval
* Ensure **error handling** for:

  * Failed authentication
  * Empty resource list
  * API failures per resource
* Ensure **idempotency** where possible
* Maintain **sequential execution order** (sorting must occur before downloads)

---

## Notes

* Filtering must be substring-based on `resourceId` using `entityId`
* Date must strictly follow `YYYY-MM-DD` format
* Sorting is critical before file download

