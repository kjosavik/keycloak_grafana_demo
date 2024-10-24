### Grafana and Keycloak Docker Compose Example

This repository demonstrates how to integrate Grafana with Keycloak using OAuth 2.0 for authentication. The setup includes Grafana, Keycloak, and Postgres containers orchestrated with Docker Compose. The purpose of this demonstration is to show a non-production-ready environment where Grafana utilizes groups and roles available on the ID token sent from Keycloak, as well as data from the user info endpoint.

```
This setup is not suitable for production. It's meant for demonstration purposes and does not include security hardening.
```

## Prerequisites

Before you get started, make sure you have the following installed:

 - Docker
## Quick Start

### Clone the Repository
```
git clone https://github.com/kjosavik/keycloak_grafana_demo.git
```

### Start the Containers
Run the following command to spin up the containers:

```
cd keycloak_grafana_demo
docker-compose up
```

This will start three services:

 - Grafana (listening on port 3000)
 - Keycloak (listening on port 3002)
 - Postgres (used by Keycloak to store data)

### Access the Services
Keycloak takes some time to start up, especially the first time since it imports a realm.
Once the containers are running, you can access the following services:

 - Grafana: http://localhost:3000
 - Keycloak Admin Console: http://localhost:3002/admin 


### Logging Into Grafana
To log into Grafana click on the "Sign in with Keycloak" button and use credentials from the table bellow. 

| Username                  | Email                     | Group       | Grafana Role | Password   |
|:--------------------------|:--------------------------|:------------|:-------------|:-----------|
| grafana_admin@example.com | grafana_admin@example.com |             | Grafana Admin| password   |
| admin@alfa.example.com    | admin@alfa.example.com    | Alfa        | Admin        | password   |
| editor@alfa.example.com   | editor@alfa.example.com   | Alfa        | Editor       | password   |
| viewer@alfa.example.com   | viewer@alfa.example.com   | Alfa        | Viewer       | password   |
| admin@bravo.example.com   | admin@bravo.example.com   | Bravo       | Admin        | password   |
| viewer@bravo.example.com  | viewer@bravo.example.com  | Bravo       | Viewer       | password   |

Keycloak will provide groups and roles to Grafana as part of the id_token and the `/userinfo` endpoint.

Roles and groups are persisted for a user as a Realm Role and a group in Keycloak. Roles and groups are then mapped to the `id_token` and  the endpoint `http://localhost:3002/realms/testrealm/protocol/openid-connect/userinfo`. The configuration of the mappers can be found [in Keycloak's admin panel.](http://localhost:3002/admin/master/console/#/testrealm/clients/acf588b2-221e-4317-82ea-459d0501cdcf/clientScopes/dedicated) 


Grafana's mapping is configured as environment variable. You can find them in the `docker-compose.yml`. Groups are mapped directly. Roles are mapped using [JMESPath syntax](https://jmespath.org/examples.html).

Mapping found in docker-compose.yml:
``` yaml
- GF_AUTH_GENERIC_OAUTH_ROLE_ATTRIBUTE_PATH: |
    contains(roles[*], 'grafana_grafana_admin') && 'GrafanaAdmin' ||
    contains(roles[*], 'grafana_admin') && 'Admin' ||
    contains(roles[*], 'grafana_editor') && 'Editor' ||
    'Viewer'
- GF_AUTH_GENERIC_OAUTH_GROUPS_ATTRIBUTE_PATH: groups
```

Role mapping overview:
| Keycloak Realm Role       | Grafana Role              |
|:--------------------------|:--------------------------|
| grafana_grafana_admin     | GrafanaAdmin              |
| grafana_admin             | Admin                     |
| grafana_editor            | Editor                    |
| grafana_viewer            | Viewer                    |


### Keycloak Configuration
By default, Keycloak is running with an admin user:

 - Username: admin
 - Password: admin

You can log into the Keycloak Admin Console using these credentials. Keycloak comes with a pre-configured Realm that is imported on startup.


