Placement Statistics
====================
Implementing this extension provides summary statistics to users on placement actions they've taken.
Since this builds on user objects, implementing this extension requires implementing the [users extension](./users.md).

This defines a PlacementStatistics object with the following type:
```typescript
{
	"colors": {
		"total": {
			"placed": number;
			"undone": number;
		};
		[index: number]: {
			"placed": number;
			"undone": number;
		};
	};
};
```

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["placement_statistics"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=placement_statistics
### Server packets
#### StatsUpdate
The client user's placement statistics have changed.
```typescript
{
	"type": "stats-updated";
	"stats": PlacementStatistics;
}
```
### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `socket.stats`. |

--------------------------------------------------------------------------------

## /users/stats
### GET
Information on the stats placement statistics of all users.
#### Response
A Paginated List of the following type:
```typescript
{
	"user": User;
	"stats": {
		"totals": PlacementStatistics;
		[boardId: number | string]: PlacementStatistics;
	};
}
```
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `users.list`.       |
| 403 Forbidden | Missing permission `users.stats.list`. |

--------------------------------------------------------------------------------

## /users/{user_id}/stats
### GET
Information on the placement statistics of the specified user.
#### Response
```typescript
{
	"totals": PlacementStatistics;
	[boardId: number | string]: PlacementStatistics;
}
```
Implementations may provide information on as many or few boards as they choose.
This might mean they provide information on all previous boards, only provide information for the current board, or don't provide board specific data at all.

For the field "totals", the entry represents the internal sum of all known statistics for the user.
Notably, the sum of all other entries do not need to sum to the total provided.
#### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.stats.get` or users.current.stats.get. |
| 404 Not Found | No such User exists.                                             |