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
}
```

The artifact uris provided in this object should reference valid objects when the report is created.
Some potential examples:
- /board/pixels/0/0
- /users/12
- /roles/2

These artifacts need not stay correct (due to deletion of an object) and clients fetching them should be aware of this.

The status and reason in the root of the object reflect the most recent entry in the history field.

If the [users extension](./users.md) is implemented, Reports may additionally specify the user who created them:
```typescript
{
	"reporter"?: User;
}
```

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["reports"];
}
```

--------------------------------------------------------------------------------

## /socket?extensions[]=reports
### Server packets
#### ReportCreated
There is a new report.
```typescript
{
	"type": "report-created";
	"report": Report;
}
```
#### ReportUpdated
A report has been updated.
```typescript
{
	"type": "report-updated";
	"report": Report;
}
```
#### ReportRemoved
A report has been removed.
```typescript
{
	"type": "report-removed";
	"id": number | string;
}
```
### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `socket.reports`. |

--------------------------------------------------------------------------------

## /reports
### GET 
Lists all reports.
#### Response
A Paginated List of Report References.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `reports.list`. |

### POST
Creates a Report.
#### Request
A Report object without an ID, history, or status (or reporter if the [users extension](./users.md) is implemented).
#### Response
A reference to the created Report object.
#### Errors
| Response Code | Cause                                       |
|---------------|---------------------------------------------|
| 403 Forbidden | Missing permission `reports.post`.          |
| 422 Forbidden | Invalid message.                            |
| 422 Forbidden | No artifacts provided.                      |
| 422 Forbidden | One or more provided artifacts are invalid. |

--------------------------------------------------------------------------------

If the [authentication extension](./authentication.md) is implemented, any created reports become owned by their poster.

## /reports/owned
### GET
Lists all reports owned by the client's user.
#### Response
A Paginated List of Report References.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.list` or `reports.owned.list`. |

--------------------------------------------------------------------------------

## {report_uri}
### GET
Gets a report.
#### Response
The Report object.
#### Errors
| Response Code | Cause                                                    |
|---------------|----------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.get` or `reports.owned.get`. |

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
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.patch` or `reports.owned.patch`. |

### DELETE
Deletes the report.
#### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.delete` or `reports.owned.delete`. |

--------------------------------------------------------------------------------

## {report_uri}/history
### GET
Lists all changes to a report.
#### Response
A Paginated List of the following type:
```typescript
{
	"status": "OPENED" | "CLOSED";
	"reason"?: string;
	"time": Timestamp;
}
```
If the [users extension](./users.md) is implemented, history entries may additionally contain information on the user responsible for a history event:
```typescript
{
	"user"?: User;
}
```
#### Errors
| Response Code | Cause                                                                      |
|---------------|----------------------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.get` or `reports.owned.get`.                   |
| 403 Forbidden | Missing permission `reports.history.list` or `reports.owned.history.list`. |
