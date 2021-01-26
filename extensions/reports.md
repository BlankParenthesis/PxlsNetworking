Reports
=======
Implementing this extension provides users with a means of notifying moderators of some perceived infringement and a way for moderators to view these notifications.

Reports are represented with Report objects which are described by the type below:
```typescript
{
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

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission       | Purpose                                           |
|------------------|---------------------------------------------------|
| `reports.list`   | Allows GET requests to `/reports`.                |
| `reports.get`    | Allows GET requests to `/reports/{report_id}`.    |
| `reports.post`   | Allows POST requests to `/reports`.               |
| `reports.patch`  | Allows PATCH requests to `/reports/{report_id}`.  |
| `reports.delete` | Allows DELETE requests to `/reports/{report_id}`. |

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
A list of all reports.
#### Response
An array of Report References.
#### Errors
| Response Code | Cause                                                     |
|---------------|-----------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list reports. |

### POST
Create a new Report.
#### Request
A Report object without an ID, history, or status.
#### Response
A reference to the created Report object.
#### Errors
| Response Code | Cause                                                       |
|---------------|-------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to create reports. |
| 422 Forbidden | The provided message is invalid.                            |
| 422 Forbidden | No artifacts were provided.                                 |
| 422 Forbidden | One or more of the artifacts provided is invalid.           |

--------------------------------------------------------------------------------

## /reports/open
### GET
A list of all reports with a status of "OPENED".
#### Response
An array of Report References.
#### Errors
| Response Code | Cause                                                     |
|---------------|-----------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list reports. |

--------------------------------------------------------------------------------

## /reports/closed
### GET
A list of all reports with a status of "CLOSED".
#### Response
An array of Report References.
#### Errors
| Response Code | Cause                                                     |
|---------------|-----------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list reports. |

--------------------------------------------------------------------------------

## {report_uri}
### GET
#### Response
The Report object.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the report. |

### PATCH
Re-opens or closes the report.
#### Request
```typescript
{
	"status": "OPENED" | "CLOSED";
	"reason"?: string;
}
```
#### Response
The updated Report object.
#### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to update the report. |

### DELETE
Deletes the report.
#### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to delete the report. |