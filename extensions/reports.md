Reports
=======
Implementing this extension provides users with a means of notifying moderators of some perceived infringement and a way for moderators to view these notifications.

Reports are represented with Report objects which are described by the type below:
```typescript
{
	"id": number | string;
	"artifacts": Array<{
		"uri": string;
		"timestamp"?: Timestamp;
	}>;
	"status": "OPENED" | "CLOSED";
	"reason": string;
	"history": Array<{
		"status": "OPENED" | "CLOSED";
		"reason"?: string;
		"time": Timestamp;
	}>;
}
```

The artifact uris provided in this object should reference valid objects when the report is created.
Some potential examples:
- /board/pixels/0/0
- /users/12
- /roles/2

These artifacts need not stay correct (due to deletion of an object) and clients fetching them should be aware of this.

The status and reason in the root of the object reflect the most recent entry in the history field.

If the [users extension](./users.md) is implemented, Artifacts may additionally contain information on the user responsible for a history event:
```typescript
{
	"reporter": User;
	"history": Array<{
		"user"?: User;
	}>;
}
```

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["reports"];
}
```

--------------------------------------------------------------------------------

## /reports
### GET 
Information on all reports.
Returns a Paginated List of Report objects.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | The client lacks the permission `reports.list`. |

### POST
Create a new Report.
#### Request
A Report object without an ID, history, or status (or reporter if the [users extension](./users.md) is implemented).
#### Response
The created Report object.
#### Errors
| Response Code | Cause                                             |
|---------------|---------------------------------------------------|
| 403 Forbidden | The client lacks the permission `reports.post`    |
| 422 Forbidden | The provided message is invalid.                  |
| 422 Forbidden | No artifacts were provided.                       |
| 422 Forbidden | One or more of the artifacts provided is invalid. |

--------------------------------------------------------------------------------

## /reports/open
Information on all Reports with a status of "OPENED".
Returns a Paginated List of Report objects.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | The client lacks the permission `reports.list`. |
--------------------------------------------------------------------------------

## /reports/closed
Information on all Reports with a status of "CLOSED".
Returns a Paginated List of Report objects.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | The client lacks the permission `reports.list`. |

--------------------------------------------------------------------------------

## /reports/{report_id}
### GET
Information on a specified report.
Returns the Report object with the ID matching `report_id`.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | The client lacks the permission `reports.get`. |
| 404 Not Found | No report with the requested ID exists.        |

### PATCH
Re-opens or closes a report.
#### Request
```typescript
{
	"status": "OPENED" | "CLOSED";
	"reason"?: string;
}
```
#### Response
The modified Report.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 404 Not Found | No report with the requested ID exists.          |
| 403 Forbidden | The client lacks the permission `reports.patch`. |

### DELETE
Deletes a report.
#### Errors
| Response Code | Cause                                             |
|---------------|---------------------------------------------------|
| 404 Not Found | No report with the requested ID exists.           |
| 403 Forbidden | The client lacks the permission `reports.delete`. |