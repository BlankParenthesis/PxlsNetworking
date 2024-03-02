Factions Cosmetics
==================
Implementing this extension adds new fields to factions to add more identity to each.
As such, implementing this extension requires implementing the [factions extension](./factions.md).

Faction objects are updated by this extension to contain these fields:
```typescript
{
	"tag"?: string;
	"color"?: number;
	"icon"?: string;
	"banner"?: string;
}
```
`tag` is a short string which serves as an abbreviation of the faction's full identifier.

`color` is a 8-bit rgb color (encoded as a single number) associated with the faction generally.

`icon` is the uri of a small 1:1 aspect ratio image representing the faction.

`banner` is the uri of a large image to be used as a header or background associated with the faction.

Faction members gain an additional field indicating that the faction may be displayed next to the user:
```typescript
{
	"displays_faction": boolean;
}
```

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["factions_cosmetics"];
}
```