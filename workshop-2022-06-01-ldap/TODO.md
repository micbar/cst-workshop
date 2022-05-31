
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
- ` LDAPTLS_REQCERT=never` to disable ldap certificate checking for `ldapsearch`
- use `ldapsearch` (on the host) to explore the default LDAP tree (see also https://owncloud.dev/extensions/idm/configuration_hints/#access-via-ldap-command-line-tools, eg. `ldapsearch -x -H ldaps://127.0.0.1:9235 -x -D uid=admin,ou=users,o=libregraph-idm -W -b o=libregraph-idm objectclass=inetorgperson`)
- add some users/groups via the user management UI and use `ldapsearch` again
- teardown this setup

## Setup oCIS with an external (read-only) LDAP server

- an LDAP server is listening for LDAPS connections on `localhost` port 1636:
  - BaseDN: "dc=owncloud,dc=com"
  - Service User": "cn=ocis,dc=owncloud,dc=com"
  - Password for Service User": "servicepw"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
  - Use `entryUUID` as the id attribute for users and groups
- start ocis so that it is using the above LDAP server as a read-only LDAP server for all its services (including the builtin IDP). Use
  the following environment as a starting point:

``` bash
    # Restrict the services to not start idm
    export OCIS_RUN_EXTENSIONS=app-registry,app-provider,audit,auth-basic,auth-bearer,auth-machine,frontend,gateway,graph,graph-explorer,groups,idp,nats,notifications,ocdav,ocs,proxy,search,settings,sharing,storage-system,storage-publiclink,storage-shares,storage-users,store,thumbnails,users,web,webdav
```
- more configuration options can be found eg. here https://owncloud.dev/extensions/users/configuration/#environment-variables (stick to the `LDAP_*` settings, since these configure all extensions at once)

- if the above is working try to reconfigure oCIS so that it only allows users of the group `physics-lovers` to use oCIS
  Hint: The ldap-server maintains an attribute "memberOf" on the user objects to list a user's groups memberships

## Setup oCIS with user autoprovsioning (using an external LDAP server)
- another LDAP server is listening for LDAPS connections on `localhost` port 2636:
  - BaseDN: "dc=owncloud,dc=com"
  - Service User": "cn=admin,dc=owncloud,dc=com"
  - Password for Service User": "adminpassword"
  - Users BaseDN": "ou=users,dc=owncloud,dc=com"
  - Group BaseDN": "ou=groups,dc=owncloud,dc=com"
- use the prepopulated keycloak running at https://keycloak.group<x>.ldap-workshop.owncloud.works with the realm `oCIS` -> `export OCIS_OIDC_ISSUER=https://keycloak.group<x>.ldap-workshop.owncloud.works/auth/realms/oCIS`
- enable user autoprovisioning in the proxy (https://owncloud.dev/extensions/proxy/configuration/)
  - ```bash
    # Restrict the services to not start idm and idp
    export OCIS_RUN_EXTENSIONS=app-registry,app-provider,audit,auth-basic,auth-bearer,auth-machine,frontend,gateway,graph,graph-explorer,groups,nats,notifications,ocdav,ocs,proxy,search,settings,sharing,storage-system,storage-publiclink,storage-shares,storage-users,store,thumbnails,users,web,webdav
```
- login to the Keycloak admin UI ( https://keycloak.group<x>.ldap-workshop.owncloud.works, user: admin, password: admin) and add your own user
