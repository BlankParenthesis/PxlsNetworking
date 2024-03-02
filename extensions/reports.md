Reports
=======
Implementing this extension provides users with a means of notifying moderators of some perceived infringement and a way for moderators to view these notifications.

Reports are represented with Report objects which are described by the type below:
```typescript
{
	"artifacts": Array<{
		"reference": Reference<any>;
		"type": string;
		"timestamp"?: Timestamp;
	}>;
	"status": "OPENED" | "CLOSED";
	"reason": string;
}
```

The artifact type provided in this object should reference valid objects when the report is created.
Some potential examples:
- /boards/1/pixels/0
- /users/12
- /roles/2

These artifacts need not stay correct (due to deletion of an object) and clients fetching them should be aware of this.

The status and reason in the root of the object reflect the most recent entry in the history field.

If the [users extension](./users.md) is implemented, Reports may additionally specify the user who created them:
```typescript
{
	"reporter"?: Reference<User>;
}
```

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["reports"];
}
```

--------------------------------------------------------------------------------

### /events?subscribe[]={events_list}
#### Server packets
##### ReportCreated
Sent when a report is created.
Set `subscribe[]=reports` to receive this event.
```typescript
{
	"type": "report-created";
	"report": Reference<Report>;
}
```
##### ReportUpdated
Sent when a report is updated.
Set `subscribe[]=reports` to receive this event.
```typescript
{
	"type": "report-updated";
	"report": Reference<Report>;
}
```
##### ReportRemoved
Sent when a report is removed.
Set `subscribe[]=reports` to receive this event.
```typescript
{
	"type": "report-removed";
	"report": Url;
}
```
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `events.reports`. |

--------------------------------------------------------------------------------

### /reports
#### GET 
Lists all reports.
##### Response
A Paginated List of Report References.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `reports.list`. |

#### POST
Creates a Report.
##### Request
A Report object without history or status.
*NOTE: the reporter field from the [users extension](./users.md) should not be included here*
##### Response
A reference to the created Report object.
##### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.post`.                      |
| 422 Unprocessable Content | Invalid message.                            |
| 422 Unprocessable Content | No artifacts provided.                      |
| 422 Unprocessable Content | One or more provided artifacts are invalid. |

--------------------------------------------------------------------------------

If the [authentication extension](./authentication.md) is implemented, any created reports become owned by their poster.

### /reports/owned
#### GET
Lists all reports owned by the client's user.
##### Response
A Paginated List of Report References.
##### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.list` or `reports.owned.list`. |

--------------------------------------------------------------------------------

### {report_uri}
#### GET
Gets a report.
##### Response
The Report object.
##### Errors
| Response Code | Cause                                                    |
|---------------|----------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.get` or `reports.owned.get`. |

#### PATCH
Re-opens or closes the report.
##### Request
```typescript
{
	"status": "OPENED" | "CLOSED";
	"reason"?: string;
}
```
##### Response
The updated Report object.
##### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.patch` or `reports.owned.patch`. |

#### DELETE
Deletes the report.
##### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.delete` or `reports.owned.delete`. |

--------------------------------------------------------------------------------

### {report_uri}/history
#### GET
Lists all changes to a report.
##### Response
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
	"reporter"?: User;
}
```
##### Errors
| Response Code | Cause                                                                      |
|---------------|----------------------------------------------------------------------------|
| 403 Forbidden | Missing permission `reports.get` or `reports.owned.get`.                   |
| 403 Forbidden | Missing permission `reports.history.list` or `reports.owned.history.list`. |
