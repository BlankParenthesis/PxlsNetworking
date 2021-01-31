Placement Statistics
====================
Implementing this extension provides summary statistics to users on placement actions they've taken.
Since this builds on user objects, implementing this extension requires implementing the [users extension](./users.md).

This extension adds the following fields to the Users object:
```typescript
{
	"stats": {
		[boardId: number | string | "totals"]: {
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
	};
}
```
Implementations may choose to provide information on as many boards as they choose.
This might mean they provide information on all previous boards, only provide information for the current board, or don't provide board specific data at all.

If the `boardId` is "totals", the entry represents the internal sum of all known statistics for the user.
Notably, the sum of all other entries do not need to sum to the total provided.