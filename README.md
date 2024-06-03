# CypherHound

![image](https://github.com/gokupwn/cypherHound/assets/76757267/c47581f7-8823-4481-86fd-089064e69cde)


## Get Inactive or Decommissioned Machines:

- Retrieves all computers where the last logon occurred before a specified year.
- Useful for querying computer objects that exist in the AD domain but have not logged on for a long time.
- This helps in identifying potentially inactive or decommissioned machines.

```cypher
MATCH (c:Computer)
WHERE c.lastlogon IS NOT NULL
AND datetime({epochMillis: toInteger(c.lastlogon * 1000)}).year < 2010
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

## Query for Users with Password Last Set in Year X:

- Identify user accounts where the password was last set in the year 2009. It is specifically useful for auditing security practices related to password management policies and identifying potentially dormant or neglected accounts in the Active Directory environment.

```cypher 
MATCH (u:User)
WHERE u.pwdlastset IS NOT NULL
AND datetime({epochMillis: toInteger(u.pwdlastset * 1000)}).year = $YEAR_X
RETURN *

// EXAMPLE
MATCH (u:User)
WHERE u.pwdlastset IS NOT NULL
AND datetime({epochMillis: toInteger(u.pwdlastset * 1000)}).year = 2009
RETURN *
```

## Query to identify AS-REP Roastable users with passwords set 10 or more years ago and passwords that never expire:

- Retrieves all users who are susceptible to AS-REP Roasting, have passwords that never expire, and whose passwords were set 10 or more years ago.

```cypher
MATCH (u:User {dontreqpreauth: true})
WHERE u.pwdlastset IS NOT NULL 
AND u.pwdneverexpires = true
AND datetime({epochMillis: toInteger(u.pwdlastset * 1000)}).year <= (datetime().year - 10)
RETURN *
```

## Query to get all Kerberoastable users:

```cypher
MATCH (u:User {hasspn:true})
RETURN *
```

## Query to identify Kerberoastable users with passwords set 10 or more years ago and passwords that never expire:

- Retrieves all users who are susceptible to Kerberoasting, have passwords that never expire, and whose passwords were set 10 or more years ago.

```cypher
MATCH (u:User {hasspn:true})
WHERE u.pwdlastset IS NOT NULL 
AND u.pwdneverexpires = true
AND datetime({epochMillis: toInteger(u.pwdlastset * 1000)}).year <= (datetime().year - 10)
RETURN *
```
