
- ssh onto your group's server
  - ssh commands:
    - group 1: `ssh admin@ocis.group1.ldap-workshop.owncloud.works`
    - group 2: `ssh admin@ocis.group2.ldap-workshop.owncloud.works`
    - group 3: `ssh admin@ocis.group3.ldap-workshop.owncloud.works`
    - group 4: `ssh admin@ocis.group4.ldap-workshop.owncloud.works`
    - group 5: `ssh admin@ocis.group5.ldap-workshop.owncloud.works`
    - group 6: `ssh admin@ocis.group6.ldap-workshop.owncloud.works`
    - group 7: `ssh admin@ocis.group7.ldap-workshop.owncloud.works`
    - group 8: `ssh admin@ocis.group8.ldap-workshop.owncloud.works`

## Explore the internal LDAP (libregraph-idm)

- start the oCIS as binary (daily edition from here https://download.owncloud.com/ocis/ocis/daily/, we need some unreleased bug fixes) in a default configuration, with following changes:
  - set `export OCIS_URL=https://ocis.group<x>.ldap-workshop.owncloud.works`, this is the domain, for that Traefik will forward requests to oCIS (adapt to your group)
  - set `export PROXY_TLS=false` since we run oCIS behind Traefik, which does the SSL termination for us
  - don't forget `ocis init` (we don't need insecure, since we have valid LetsEncrypt certificates through Traefik. If you want a custom admin password, set it with the `--admin-password` option)
  - enable demo users `export IDM_CREATE_DEMO_USERS=true`
  - check if you can access oCIS and log in using the admin user
- use `ldapsearch` (on the host) to explore the default LDAP tree:
  - see https://owncloud.dev/extensions/idm/configuration_hints/#access-via-ldap-command-line-tools for some hints on how to do that
  - set `export LDAPTLS_REQCERT=never` to disable ldap certificate checking for the `ldapsearch` util
  - examples:
    - to list all users:
      ``` bash
      ldapsearch -x -H ldaps://127.0.0.1:9235 -x -D uid=admin,ou=users,o=libregraph-idm -W -b o=libregraph-idm "objectclass=inetorgperson"
      ```
    - to list all groups:
      ``` bash
      ldapsearch -x -H ldaps://127.0.0.1:9235 -x -D uid=admin,ou=users,o=libregraph-idm -W -b o=libregraph-idm "objectclass=groupOfNames"
      ```
    - to search the user "einstein":
      ``` bash
      ldapsearch -x -H ldaps://127.0.0.1:9235 -x -D uid=admin,ou=users,o=libregraph-idm -W -b o=libregraph-idm "(&(objectclass=inetorgperson)(uid=einstein))"
      ```
- add some users/groups via the oCIS user management UI, use `ldapsearch` again to lookup a new user

## Setup oCIS with an external (read-only) LDAP server

- teardown the previous setup (`rm -r ~/.ocis`) and reconnect via ssh to your assigned server to start with a clean environment
- an LDAP server configured for this setup is already running and listening for LDAPS connections on `localhost` port 1636, the basic configuration parameters are:
  - BaseDN: "dc=owncloud,dc=com"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
  - Service User": "cn=ocis,dc=owncloud,dc=com"
  - Password for Service User": "servicepw"
  - Use `entryUUID` as the id attribute for users and groups (Note: `entryUUID` is an operation Attribute, i.e. only return in search results if explicitly requested)
  - The LDAP server is provsioned with the oCIS demo users as listed here: https://owncloud.dev/ocis/getting-started/demo-users/
- To explore the LDAP data use `ldapsearch` again (e.g. to list only users that are a member of the group `sailing-lovers`:
  ``` bash
  export LDAPTLS_REQCERT=never
  ldapsearch -x -H ldaps://127.0.0.1:1636 -x -D cn=ocis,dc=owncloud,dc=com -W -b dc=owncloud,dc=com "(&(objectclass=inetOrgPerson)(memberOf=cn=sailing-lovers,ou=groups,dc=owncloud,dc=com))"
  ```
- start ocis so that it is using the above LDAP server as a read-only LDAP server for all its services (including the builtin IDP).
  - don't forget `ocis init` (we don't need insecure, since we have valid LetsEncrypt certificates through Traefik. If you want a custom admin password, set it with the `--admin-password` option)
  - Use the following environment as a starting point:
      ``` bash
      export PROXY_TLS=false
      export OCIS_URL=https://ocis.group<x>.ldap-workshop.owncloud.works
      # Restrict the services to not start idm
      export OCIS_RUN_EXTENSIONS=app-registry,app-provider,audit,auth-basic,auth-bearer,auth-machine,frontend,gateway,graph,graph-explorer,groups,idp,nats,notifications,ocdav,ocs,proxy,search,settings,sharing,storage-system,storage-publiclink,storage-shares,storage-users,store,thumbnails,users,web,webdav
      # Set the various LDAP_* env vars to match the above LDAP server's settings
      ```
  - documentation for the LDAP related configuration options can be found eg. here:
    https://owncloud.dev/extensions/users/configuration/#environment-variables (stick to the `LDAP_*` environment variables, since these configure all extensions at once)
- if the above is working try to reconfigure oCIS so that it only allows users of the group `physics-lovers` to use oCIS
  Hint: The ldap-server maintains an attribute "memberOf" on the user objects to list a user's groups memberships and oCIS allows to define custom filters for users.
  - to verify the setup:
    - login as user `einstein` should work
    - login as user `moss` should not work

## Setup oCIS with user autoprovsioning (using an external LDAP server)

- teardown the previous setup (`rm -r ~/.ocis`) and reconnect via ssh to your assigned server to start with a clean environment
- another LDAP server is listening for LDAPS connections on `localhost` port 2636:
  - BaseDN: "dc=owncloud,dc=com"
  - Service User": "cn=admin,dc=owncloud,dc=com"
  - Password for Service User": "adminpassword"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
  - Apart from the above service user the LDAP server has no user or groups provisioned
- there is also a prepopulated keycloak running at `https://keycloak.group<x>.ldap-workshop.owncloud.works` with the realm `oCIS` for this scenario
  - this keycloak instance has the oCIS demo users provisioned (https://owncloud.dev/ocis/getting-started/demo-users/)
- start ocis so that it is using the keycloak server as its IDP and uses the LDAP server to automatically provsion users when they first login into oCIS
  - Use the following environment as a starting point:
      ``` bash
      export OCIS_OIDC_ISSUER="https://keycloak.group<x>.ldap-workshop.owncloud.works/auth/realms/oCIS"
      export PROXY_TLS=false
      export OCIS_URL="https://ocis.group<x>.ldap-workshop.owncloud.works"
      # Restrict the services to not start idm and idp
      export OCIS_RUN_EXTENSIONS=app-registry,app-provider,audit,auth-basic,auth-bearer,auth-machine,frontend,gateway,graph,graph-explorer,groups,nats,notifications,ocdav,ocs,proxy,search,settings,sharing,storage-system,storage-publiclink,storage-shares,storage-users,store,thumbnails,users,web,webdav

      # enable user autoprovisioning in the proxy (see https://owncloud.dev/extensions/proxy/configuration/)

      # Set the various LDAP_* env vars to match the above LDAP server's settings
      # export LDAP_INSECURE=true
      # export LDAP_URI=...
      # ...
- login to the Keycloak admin UI ( `https://keycloak.group<x>.ldap-workshop.owncloud.works`, user: admin, password: admin) and add your own user
- login to oCIS with that user
- upload or create a file/directory
- try to share that file/directory with another user (it will not work yet as no other users are listed)
- login into oCIS with one of the demo users
- try again to share a file (you should now at least be able to share files with the other user that already logged in)
- use `ldapsearch` again to list the users
