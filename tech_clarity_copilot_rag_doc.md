# SimpleTexting API Reference for Tech Clarity Copilot

## Introduction
Thousands of businesses rely on SimpleTexting to communicate with their audience via text message. With our API, developers can access many of our platform’s features and integrate them with other websites or applications. This document details the available SimpleTexting API functions and their parameters. For additional security, our API is by approval only. If you’d like access, sign up for a trial account and email support@simpletexting.net with details about your use case.

How it works

Our API is organized around REST. It uses standard HTTP response codes and authentication. Before you get started, there a few things to keep in mind:

- When using the POST request, you must specify that content-type is application/json.
- The format of responses for all requests is JSON, you can skip the Accept request header or set it to application/json.

## Authentication
All requests to the SimpleTexting API require **Bearer token authentication**.  
Requests without a valid token will **fail**.

Your API token can be found in your SimpleTexting **Account Settings**.

**Important:**  
Keep your bearer token secure. Do **not** store it in public code repositories (e.g., GitHub) or expose it in client-side code.

## How Authentication Works
SimpleTexting uses **Bearer Authentication** (token-based authentication).

A *bearer token* is a secure, randomly generated string issued by the server.  
The token must be included in every protected request:

- Authorization: Bearer <your_token_here>

"Bearer" simply means: **anyone who possesses this token is granted access**.

### Security Scheme
| Field | Value |
|-------|--------|
| **Scheme Type** | API Key |
| **Location** | Header |
| **Header Name** | `Authorization` |
| **Format** | `Bearer <token>` |

### Example cURL Request
```bash
curl -X GET "https://api-app2.simpletexting.com/v2/api/campaigns" \
  -H "Authorization: Bearer YOUR_API_KEY_HERE"
```

### Tags
`authentication` `security` `bearer` `api_key`

## Contacts
At a minimum, each contact in SimpleTexting must have a phone number. Contacts can also hold additional information including first name, last name, and email address. You can also create custom fields to store data specific to your website or app’s needs.

## Get a Contact
Retrieve a contact via their **unique ID** or **phone number**.  
> Phone number is the preferred parameter for this call.

Example use case:  
Fetch the contact with ID `507f1f77bcf86cd799439011`.  
You can use the **Get All Contacts** endpoint to retrieve contact IDs for all of your contacts.

### Example Request
```http
GET https://api-app2.simpletexting.com/v2/api/contacts/507f1f77bcf86cd799439011
```

### Authorization
* `api_key` required

### Path Parameters
| Name                | Type   | Required | Example                                   | Description                                                  |
| ------------------- | ------ | -------- | ----------------------------------------- | ------------------------------------------------------------ |
| `contactIdOrNumber` | string | yes      | `3051234567` / `507f1f77bcf86cd799439011` | Phone number (preferred) or contact ID in hexadecimal format |


### Responses
**200 Success** Fetched contact.

```json
{
  "contactId": "607f0558a7c898629dd47d7a",
  "contactPhone": "1234567890",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@simpletext.org",
  "birthday": "1985-05-15",
  "lists": [
    {}
  ],
  "customFields": {
    "zipcode": "12345"
  },
  "comment": "VIP client",
  "subscriptionStatus": "OPT_IN",
  "created": "2021-04-28T23:20:08.489Z",
  "updated": "2021-05-28T20:20:08.489Z",
  "updateSource": "IMPORTED_FROM_FILE"
}
```

### Response Fields
| Field                | Type                             | Description                      | Example                    |
| -------------------- | -------------------------------- | -------------------------------- | -------------------------- |
| `contactId`          | string                           | Contact ID (hexadecimal)         | `607f0558a7c898629dd47d7a` |
| `contactPhone`       | string                           | Contact phone number             | `1234567890`               |
| `firstName`          | string (≤100 chars)              | Contact's first name             | `John`                     |
| `lastName`           | string (≤100 chars)              | Contact's last name              | `Doe`                      |
| `email`              | string                           | Contact's email                  | `john.doe@simpletext.org`  |
| `birthday`           | string (date)                    | Birthday in yyyy-mm-dd           | `1985-05-15`               |
| `lists`              | array of objects (`ContactList`) | Lists the contact belongs to     | `[{}]`                     |
| `customFields`       | object                           | Key/value pairs of custom fields | `{ "zipcode": "12345" }`   |
| `comment`            | string (≤1000 chars)             | Notes about the contact          | `VIP client`               |
| `subscriptionStatus` | enum                             | Contact subscription status      | `OPT_IN`                   |
| `created`            | string (ISO 8601)                | Contact creation date            | `2021-04-28T23:20:08.489Z` |
| `updated`            | string (ISO 8601)                | Contact last updated             | `2021-05-28T20:20:08.489Z` |
| `updateSource`       | enum                             | How the contact was updated      | `IMPORTED_FROM_FILE`       |

### Endpoint
```
GET /api/contacts/{contactIdOrNumber}
```

### Tags
`simpletexting` `contacts` `get` `path-parameter`

## Get All Contacts
Retrieve **all contacts** for a given account.  
Results are **paginated**.

Example use case:  
Fetch all contacts created or updated since April 28, 2021.

### Example Request
```http
GET https://api-app2.simpletexting.com/v2/api/contacts?page=100&size=2&since=2021-04-28T23:20:08.489Z&direction=ASC
```

### Authorization
* `api_key` required

### Query Parameters
| Name        | Type                | Required | Default | Example                    | Description                               |
| ----------- | ------------------- | -------- | ------- | -------------------------- | ----------------------------------------- |
| `page`      | integer ≥ 0         | no       | `0`     | `250`                      | Zero-indexed page number to return.       |
| `size`      | integer ≤ 500       | no       | `50`    | `150`                      | Number of contacts per page.              |
| `since`     | datetime (ISO 8601) | no       | -       | `2021-04-28T23:20:08.489Z` | Return contacts updated since this date.  |
| `direction` | string              | no       | `desc`  | `ASC` / `DESC`             | Sort order of results by `updated` field. |

### Request Notes
* Pagination is applied via `page` and `size`.
* Dates must be in **ISO 8601** format.
* Default sort is descending by `updated` field.

### Responses
**200 Success** Returns a paginated list of contacts.

```json
{
  "content": [
    {}
  ],
  "totalPages": 0,
  "totalElements": 0
}
```

### Response Fields
| Field           | Type                         | Description                                                     |
| --------------- | ---------------------------- | --------------------------------------------------------------- |
| `content`       | array of objects (`Contact`) | Contacts returned in the current page. Count depends on `size`. |
| `totalPages`    | integer                      | Total number of pages based on element count and page size.     |
| `totalElements` | integer                      | Total number of contacts in the account.                        |

### Endpoint
```
GET /api/contacts
```

### Tags
`simpletexting` `contacts` `get` `pagination`

## Contact Lists
Create, update, and delete contact lists.

## Get All Lists
Retrieve **all contact lists** from your SimpleTexting account.  
Results are **paginated**.

Example use case:  
Fetch all lists for an account.

### Example Request
```http
GET https://api-app2.simpletexting.com/v2/api/contact-lists?page=100&size=2
```

### Authorization
* `api_key` required

### Query Parameters
| Name   | Type          | Required | Default | Example | Description                         |
| ------ | ------------- | -------- | ------- | ------- | ----------------------------------- |
| `page` | integer ≥ 0   | no       | `0`     | `15`    | Zero-indexed page number to return. |
| `size` | integer ≤ 500 | no       | `50`    | `50`    | Number of lists to return per page. |


### Request Notes
* Pagination is applied via `page` and `size`.
* Page numbering starts at zero (0).

### Responses
**200 Success** Returns all lists in the account.

```json
{
  "content": [
    {}
  ],
  "totalPages": 0,
  "totalElements": 0
}
```

### Response Fields
| Field           | Type                      | Description                                                  |
| --------------- | ------------------------- | ------------------------------------------------------------ |
| `content`       | array of objects (`List`) | Lists returned in the current page. Count depends on `size`. |
| `totalPages`    | integer                   | Total number of pages based on element count and page size.  |
| `totalElements` | integer                   | Total number of lists in the account.                        |

### Endpoint
```
GET /api/contact-lists
```

### Tags
`simpletexting` `lists` `get` `pagination`

## Add Contact To List
Add a contact to a specified contact list in your SimpleTexting account.

### Authorization
- `api_key` required

### Path Parameters
| Name | Type | Required | Example | Description |
|------|------|----------|---------|-------------|
| `listIdOrName` | string | yes | `507f191e810c19729de860ea` / `My First List` | List ID or name where the contact will be added. |

### Request Body
**Content-Type:** `application/json`

| Field | Type | Required | Example | Description |
|-------|------|----------|---------|-------------|
| `contactPhoneOrId` | string | yes | `3051234567` / `507f1f77bcf86cd799439011` | Contact phone number (preferred) or contact ID in hexadecimal format. |

### Example Request Payload
```json
{
  "contactPhoneOrId": "3051234567 / 507f1f77bcf86cd799439011"
}
```

### Responses
**200 Success** Successfully added contact to the list.

```json
{
  "contactPhoneOrId": "3051234567 / 507f1f77bcf86cd799439011"
}
```

### Endpoint
```
POST /api/contact-lists/{listIdOrName}/contacts
```

### Tags
`simpletexting` `lists` `add-contact` `post`

## Remove Contact From List
Remove a contact from a specified contact list in your SimpleTexting account.

### Authorization
- `api_key` required

### Path Parameters
| Name | Type | Required | Example | Description |
|------|------|----------|---------|-------------|
| `listIdOrName` | string | yes | `507f191e810c19729de860ea` / `My First List` | List ID or name from which the contact will be removed. |
| `contactPhoneOrId` | string | yes | `3051234567` / `507f1f77bcf86cd799439011` | Contact phone number (preferred) or contact ID in hexadecimal format. |

### Responses
**204 Success** Contact was successfully removed from the list.  
No response body is returned.

### Endpoint
```
DELETE /api/contact-lists/{listIdOrName}/contacts/{contactPhoneOrId}
```

### Tags
`simpletexting` `lists` `remove-contact` `delete`


# Make.com - Google Sheets Modules Reference for Tech Clarity Copilot
After connecting your Google Sheets account, you can use the following modules to build scenarios in Make.com.

## Module: Get a Range of Values
Retrieve values from a specific range in a Google Sheet.

### Connection
Create a connection to your Google Sheets account.

### Input Parameters
| Field                | Type    | Required | Example / Notes | Description |
| -------------------- | ------- | -------- | --------------- | ----------- |
| Search Method         | string  | yes      | `Search by path` / `Select from all` / `Enter manually` | Choose which spreadsheet to retrieve range values from. |
| Spreadsheet ID        | string  | yes      | `abc1234567` (from URL: `https://docs.google.com/spreadsheets/d/abc1234567/edit#gid=0`) | The ID of the spreadsheet to get values from. |
| Sheet Name            | string  | yes      | `Sheet1` | The sheet to get values from. |
| Range                 | string  | yes      | `A1:D25` | Range to retrieve values from. |
| Table contains headers| string  | no       | `A1:F1` | Row range of table headers. Defaults to first row if empty. |
| Value render option   | enum    | no       | `Formatted value` / `Unformatted value` / `Formula` | How values should be returned: calculated & formatted, calculated only, or formulas. |
| Date and render option| enum    | no       | `Serial number` / `Formatted string` | How date/time/duration fields are returned: numeric serial or formatted string. |

### Value Render Option Details
- **Formatted value:** Returns calculated and formatted values according to spreadsheet locale.  
- **Unformatted value:** Returns calculated values without formatting.  
- **Formula:** Returns cell formulas instead of calculated values.

### Date and Render Option Details
- **Serial number:** Date/time/duration as numbers (days since Dec 30, 1899, fractional portion = time).  
- **Formatted string:** Date/time/duration returned as string formatted according to spreadsheet locale.

### Tags
`make` `google_sheets` `get-range` `read`

## Module: Clear Values from a Range
Clears a specific range of values from a Google Sheet.

### Connection
Create a connection to your Google Sheets account.

### Input Parameters
| Field         | Type   | Required | Example / Notes | Description |
| ------------- | ------ | -------- | --------------- | ----------- |
| Search Method | string | yes      | `Search by path` / `Select from all` / `Enter manually` | Choose the spreadsheet and sheet to clear. |
| Spreadsheet ID| string | yes      | `abc1234567` (from URL: `https://docs.google.com/spreadsheets/d/abc1234567/edit#gid=0`) | Spreadsheet ID to clear values from. |
| Sheet Name    | string | yes      | `Sheet1` | Name of the sheet to clear. |
| Range         | string | yes      | `A1:D25` | Range to clear values from. |

### Notes
- Ensure the correct spreadsheet and sheet are selected.  
- Range values will be permanently cleared from the sheet.

### Tags
`make` `google_sheets` `clear-range` `write`
