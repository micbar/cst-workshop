## Explore the internal LDAP (libregraph-idm)
- start the oCIS docker image in a default configuration (but expose port 9325 to allow accessing the internal LDAP server from the host)
- use `ldapsearch` (on the host) to explore the default LDAP tree (#TODO add example here)
- add some users/groups via the usermanagement UI and use `ldapsearch` again
- teardown this setup

## Setup oCIS with an external (read-only) LDAP server
- LDAP server is listening for LDAPS connections on `localhost` port 1636:
  - BaseDN: "dc=owncloud,dc=com"
  - Service User": "cn=ocis,dc=owncloud,dc=com"
  - Password for Service User": "servicepw"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
  - Use `entryUUID` as the id attribute for users and groups
- start ocis so that it is using the above LDAP server as a read-only LDAP server for all its services (including the builtin IDP). Use
  the following environment as a starting point:
```
    # Restrict the services to not start idm
    export OCIS_RUN_EXTENSIONS=app-registry,app-provider,audit,auth-basic,auth-bearer,auth-machine,frontend,gateway,graph,graph-explorer,groups,idp,nats,notifications,ocdav,ocs,proxy,search,settings,sharing,storage-system,storage-publiclink,storage-shares,storage-users,store,thumbnails,users,web,webdav
    
    # Service secrets
    export OCIS_JWT_SECRET="ahpo8shaDee9pah1"
    export STORAGE_TRANSFER_SECRET="uP3seiz5ba8fieho"
    export OCIS_SYSTEM_USER_API_KEY="paiRoh5ohf7wieTh"
    export OCIS_MACHINE_AUTH_API_KEY="oereeF7noix3vaeC"
    
    # User IDs for boot strapping
    export OCIS_SYSTEM_USER_ID="some-system-user-id-000-000000000000"
    export OCIS_ADMIN_USER_ID="68fdd60c-7451-103c-9a31-3557ca419b62"
```

- if the above is working try to reconfigure oCIS so that it only allows users of the group `physics-lovers` to use oCIS
  Hint: The ldap-server maintains an attribute "memberOf" on the user objects to list a user's groups memberships

## Setup oCIS with user autoprovsioning (using an external LDAP server)
- start a new OpenLDAP server (with the owncloud Schema enabled)
  - #TODO: create another docker-compose file for this LDAP config
- start a prepopulated keycloak 
- Task: start oCIS using the prepopulated keycloak as the IDP, configure it autoprovsion accounts in the external LDAP
        server.

