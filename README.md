# CypherHound
## Get Inactive or Decommissioned Machines:

- Retrieves all computers where the last logon occurred before a specified year.
- Useful for querying computer objects that exist in the AD domain but have not logged on for a long time.
- This helps in identifying potentially inactive or decommissioned machines.

```cypher
MATCH (c:Computer)
WHERE c.lastLogon IS NOT NULL
AND datetime({epochMillis: toInteger(c.lastLogon * 1000)}).year < 2010
RETURN *
```

## Get All Domain Controllers:

- Retrieves all Domain Controllers

```cypher
MATCH (c:Computer)-[:MemberOf*1..]->(g:Group) WHERE g.objectid ENDS WITH '-516' RETURN c
```

## Get All  Unconstrained Delegation Computers That are Not Domain Controllers:

- Retrieves all computers that allow unconstrained delegation but are not domain controllers.

```cypher 
MATCH (c1:Computer)-[:MemberOf*1..]->(g:Group) WHERE g.objectsid ENDS WITH '-516' WITH COLLECT(c1.name) AS domainControllers
MATCH (c2:Computer {unconstraineddelegation:true}) WHERE NOT c2.name IN domainControllers RETURN c2
```
