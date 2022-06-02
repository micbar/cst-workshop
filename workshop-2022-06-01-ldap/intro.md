## LDAP

- A standard Network Protocol that defines how to access distributed directory information services
- Governerd by the IETF. The latest version LDAPv3 is documented in RFC4511 (and quite a few more)
- LDAP is based on a simpler subset of the standards contained within the X.500 standard. 
- Implementations:
  - MicrosoftActive Directory
  - OpenLDAP
  - 389DS

## Data Model

### Directory Information Tree
![DIT](https://ldapwiki.com/attach/DIT/DIT.png)

### Entry
```
dn: uid=einstein,ou=users,dc=owncloud,dc=com
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
objectClass: posixAccount
objectClass: top
uid: einstein
givenName: Albert
sn: Einstein
cn: einstein
displayName: Albert Einstein
description: A German-born theoretical physicist who developed the theory
 of relativity, one of the two pillars of modern physics (alongside quantum
 mechanics).
mail: einstein@example.org
uidNumber: 424242
gidNumber: 424242
loginShell: /bin/bash
userPassword:: e1NTSEF9TXJEcXpFNGdKbXZxbVRVTGhvWEZ1VzJBbkV3NWFLK3J3WTIvbHc9PQ==
```

## Directory Operations
The standard defines a couple of protocol operations that standard compliant
server must implement:
- `Bind`: For authenticating the client
- `StartTLS`: For setting up TLS encryption (nowadays not recommended anymore
  for security reasons. Use `ldaps` instead.
- `Add`
- `Modify`
- `ModifyRDN` (rename)
- `Delete`
- `Search`
- `Compare`
- Exentions mechanism are defined to implement additional Operation (e.g. Change Password)

### Search in a bit more detail
Search is probably the most used operation. It has the following parameters

- `baseObject`: The name of the base object entry (or possibly the root) relative to which the search is to be performed.
- `scope`: What elements below the baseObject to search. This can be `base`, `one` or `subtree`.
- `filter`: Criteria to use in selecting elements within scope. For example, the filter 
   `(&(objectClass=person)(|(givenName=John)(mail=john*)))` will select "persons" (entries of the objectClass person),
   where the attribute `givenName` is `John` or the `mail` address starts with `john`.
- `attributes`: Which attributes to return in result entries.
- `sizeLimit`, `timeLimit`: Maximum number of entries to return, and maximum time to allow search to run.
  These values, however, cannot override any restrictions the server places on size limit and time limit.
- and a few others left out for simplicity

### `ldapsearch` command

``` bash
ldapsearch
    -x <--- use simple bind (username / password)
    -W <---- will cause ldapsearch to prompt for the password
    -H <ldap(s) URL of the server to connect to>
    -D <DN of the user to authenticated as>
    -b <search base DN>
    -s <search scope>
    <search filter>
    <space separated list of attributes to return>
```

# LDAP in oCIS
https://owncloud.dev/extensions/idm/

https://owncloud.dev/extensions/users/configuration/

https://owncloud.dev/extensions/groups/configuration/

https://owncloud.dev/extensions/auth-basic/configuration/

https://owncloud.dev/extensions/idp/

https://owncloud.dev/extensions/graph/

# Hands on
https://github.com/wkloucek/cst-workshop/blob/main/workshop-2022-06-01-ldap/TODO.md
