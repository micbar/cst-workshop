## Explore the internal LDAP (libregraph-idm)
- start the oCIS docker image in a default configuration (but expose port 9325 to allow accessing the internal LDAP server from the host)
- use `ldapsearch` (on the host) to explore the default LDAP tree (#TODO add example here)
- add some users/groups via the usermanagement UI and use `ldapsearch` again
- teardown this setup

## Setup oCIS with an external (read-only) LDAP server
- LDAP server running as "ldap-server1" in network "ocis-net":
  - BaseDN: "dc=owncloud,dc=com"
  - Service User": "uid=ocis,ou=serviceusers,dc=owncloud,dc=com"
  - Password for Service User": "servicepw"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
- configure oCIS to use that LDAP server as a read-only LDAP server for all its services (including the builtin IDP)
- if the above is working try to reconfigure oCIS so that it only allows users of the group `physics-lovers` to use oOCIS
- teardown this setup

## Setup oCIS with user autoprovsioning (using an external LDAP server)
- start a new OpenLDAP server (with the owncloud Schema enabled)
  - #TODO: create another docker-compose file for this LDAP config
- start a prepopulated keycloak 
- Task: start oCIS using the prepopulated keycloak as the IDP, configure it autoprovsion accounts in the external LDAP
        server.

