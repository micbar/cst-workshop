

work on these topics as a group:

- log into the Keycloak management UI
  - credentials: admin / cst-workshop
  - Group 1: https://keycloak.group-1.cst-workshop.owncloud.works
  - ...

- create a new Realm called "ocis"
    - Read: what is a realm ? (https://www.keycloak.org/docs/latest/server_admin/#configuring-realms)

- add a user for yourself to the "ocis" realm

- add a client for the OpenID Connect debugger: https://oidcdebugger.com/
    - choose any client id
    - client protocol is "openid-connect"
    - root url is "https://oidcdebugger.com/"

- have a look at the config options of your newly created client
  - don't change the default settings for now

- try the login workflow of your newly created client with https://oidcdebugger.com/
  - Authorize URI is your keycloak url + "/auth/realms/ocis/protocol/openid-connect/auth"
  - Client ID is the one you chose in Keycloak
  - Response Type: code

  - On the login screen: have a look at the URL bar / query parameters
  - HINT: if you try the login flow a second time, you won't be asked for a password anymore. This is because you already have an active session. You can log out by nafvigating to your keycloak url + "/auth/realms/ocis/account/" and clicking "sign out" in the top right corner. Alternatively navigate to the user as an keycloak admin and log out all sessions of the user in the "sessions" tab


- import following clients into the ocis:
  - ownCloud Web
  - Desktop Client
  - Android Client
  - from here: https://github.com/owncloud/ocis/tree/master/deployments/examples/ocis_keycloak/config/keycloak/clients

- log into oCIS / ownCloud Web by browsing to:
  - Group 1: https://ocis.group-1.cst-workshop.owncloud.works
  - ...
  - ...

  - see if your l


- list user accounts in ocis:
  - ssh onto the server
  - cd to /opt/.....
  - exec into the ocis container: `docker-compose exec ocis sh`
  - use accounts cli to list and inspect users (especially the user you originally created in Keycloak)


- dynamic client registration
  - allow dynamic client registraion
  - remove the desktop client oidc client (which you previously imported)
  - log in from a desktop client
  - inspect the newly created oidc client
