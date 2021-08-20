Cooldown Information
====================
Implementing this extension allows the exact formula used to calculate cooldown to be derived by clients and other third parties without internal knowledge of the server implementation.
It utilizes a series of types defined here to describe how the cooldown should be calculated:
```typescript
interface All {
	"type": "ALL";
	"operands": Condition[];
}

interface Any {
	"type": "ANY";
	"operands": Condition[];
}

interface Not {
	"type": "NOT";
	"condition": Condition;
}

interface Equal {
	"type": "EQUAL";
	"left": Condition;
	"right": Condition;
}

interface Greater {
	"type": "GREATER";
	"left": Condition;
	"right": Condition;
}

interface Less {
	"type": "LESS";
	"left": Condition;
	"right": Condition;
}

type Condition = All | Any | Not | Equal | Greater | Less | Variable;

interface Select {
	"type": "SELECT";
	"condition": Condition;
	"if": Formula;
	"else": Formula;
}

/**
 * Calculated as the list containing values beginning with @start and 
 * increasing by @step until the last value is equal to or exceeds @end.
 * If @end is less than @start then the list is empty.
 */
interface Range {
	"type": "RANGE";
	"start": Formula;
	"end": Formula;
	"step": Formula;
}

interface Sum {
	"type": "SUM";
	"operands": Formula[] | Range;
}

interface Product {
	"type": "PRODUCT";
	"operands": Formula[] | Range;
}

interface Power { 
	"type": "POWER";
	"base": Formula;
	"exponent": Formula;
}

interface Constant {
	"type": "CONSTANT";
	"name": string;
	"value": number;
}

interface Variable {
	"type": "VARIABLE";
	"name": string;
}

type Formula = Sum | Product | Power | Constant | Variable;
```

The value of a Variable is determined based on its name.
The following table provides variable names and where their value can be sourced from:

| Variable Name     | Value Source                                                                            |
|-------------------|-----------------------------------------------------------------------------------------| 
| activeUsers       | The count of online users.                                                              |
| isBackgroundPixel | True if the last pixel placed was not modified previously                               |
| stackTarget       | The cooldown will reflect how long it takes to accumulate this number of pixels from 0. |

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["cooldown_info"];
}
```

--------------------------------------------------------------------------------

## /boards/{board_id}/cooldown
### GET
Gets the cooldown formula of the board.
#### Response
```typescript
{
	"formula": Formula;
}
```
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `board.cooldown`. |

#### Example
This example shows the current pxls reference implementation formula.
```JSON
{
	"formula": {
		"type": "PRODUCT",
		"operands": [
			{
				"type": "SUM",
				"operands": [
					{
						"type": "PRODUCT",
						"operands": [
							{
								"type": "CONSTANT",
								"name": "steepness",
								"value": 2.5
							},
							{
								"type": "POWER",
								"base": {
									"type": "SUM",
									"operands": [
										{
											"type": "VARIABLE",
											"name": "activeUsers"
										},
										{
											"type": "CONSTANT",
											"name": "userOffset",
											"value": 11.96
										}
									]
								},
								"exponent": {
									"type": "CONSTANT",
									"name": "sqrt",
									"value": 0.5
								}
							}
						]
					},
					{
						"type": "CONSTANT",
						"name": "globalOffset",
						"value": 6.5
					}
				]
			},
			{
				"type": "CONSTANT",
				"name": "multiplier",
				"value": 1
			},
			{
				"type": "SELECT",
				"condition": {
					"type": "ANY",
					"operands": [
						{
							"type": "VARIABLE",
							"name": "isBackgroundPixel"
						},
						{
							"type": "GREATER",
							"left": {
								"type": "VARIABLE",
								"name": "stackTarget",
							},
							"right": {
								"type": "CONSTANT",
								"name": "one",
								"value": 1
							}
						}
					]
				},
				"if": {
					"type": "CONSTANT",
					"name": "one",
					"value": 1
				},
				"else": {
					"type": "CONSTANT",
					"name": "backgroundMultiplier",
					"value": 1.6
				}
			},
			{
				"type": "SELECT",
				"condition": {
					"type": "GREATER",
					"left": {
						"type": "VARIABLE",
						"name": "stackTarget",
					},
					"right": {
						"type": "CONSTANT",
						"name": "one",
						"value": 1
					}
				},
				"if": {
					"type": "PRODUCT",
					"operands": [
						{
							"type": "CONSTANT",
							"name": "stackMultiplier",
							"value": 3
						},
						{
							"type": "SUM",
							"operands": [
								{
									"type": "CONSTANT",
									"name": "one",
									"value": 1
								},
								{
									"type": "VARIABLE",
									"name": "stackTarget",
								},
								{
									"type": "SUM",
									"operands": {
										"type": "RANGE",
										"start": {
											"type": "CONSTANT",
											"name": "one",
											"value": 1
										},
										"end": {
											"type": "SUM",
											"operands": [
												{
													"type": "CONSTANT",
													"name": "minusOne",
													"value": -1
												},
												{
													"type": "VARIABLE",
													"name": "stackTarget",
												}
											]
										},
										"step": {
											"type": "CONSTANT",
											"name": "one",
											"value": 1
										}
									}
								}
							]
						}
					]
				},
				"else":  {
					"type": "CONSTANT",
					"name": "one",
					"value": 1
				}
			}
		]	
	}
}
```