Spring Boot REST Service Protected Using Keycloak Authorization Services
===================================================

Level: Beginner
Technologies: Spring Boot
Summary: Spring Boot REST Service Protected Using Keycloak Authorization Services
Target Product: Keycloak

What is it?
-----------

This quickstart demonstrates how to protect a Spring Boot REST service using Keycloak Authorization Services.

It tries to focus on the authorization features provided by Keycloak Authorization Services, where resources are
protected by a set of permissions and policies defined in Keycloak and access to these resources are enforced by a policy enforcer(PEP)
that intercepts every single request sent to the application to check whether or not access should be granted.

System Requirements
-------------------

To compile and run this quickstart you will need:

* JDK 21
* Apache Maven 3.9.6
* Spring Boot 3.2.2
* Keycloak 23.0.5

Starting and Configuring the Keycloak Server
-------------------

To start a Keycloak Server you can use OpenJDK on Bare Metal, Docker, Openshift or any other option described in [Keycloak Getting Started guides]https://www.keycloak.org/guides#getting-started. For example when using Docker just run the following command in the root directory of this quickstart:

```shell
docker run --name keycloak \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  --network=host \
  quay.io/keycloak/keycloak:{KC_VERSION} \
  start-dev \
  --http-port=8180
```

where `KC_VERSION` should be set to 23.0.5 or higher.

You should be able to access your Keycloak Server at http://localhost:8180.

Log in as the admin user to access the Keycloak Administration Console. Username should be `admin` and password `admin`.

Import the [realm configuration file](config/realm-import.json) to create a new realm called `quickstart`.
For more details, see the Keycloak documentation about how to [create a new realm](https://www.keycloak.org/docs/latest/server_admin/index.html#_create-realm).

Build and Run the Quickstart
-------------------------------

If your server is up and running, perform the following steps to start the application:

1. Open a terminal and navigate to the root directory of this quickstart.

2. The following shows the command to run the application:

   ````
   mvn spring-boot:run

   ````

Access the Quickstart
---------------------

There are 2 endpoints exposed by the service:

* http://localhost:8080/ - can be invoked by any authenticated user
* http://localhost:8080/protected/premium - can be invoked by users with the `user_premium` role

To invoke the protected endpoints using a bearer token, your client needs to obtain an OAuth2 access token from a Keycloak server.
In this example, we are going to obtain tokens using the resource owner password grant type so that the client can act on behalf of any user available from
the realm.

You should be able to obtain tokens for any of these users:

| Username | Password | Roles        |
|----------|----------|--------------|
| jdoe     | jdoe     | user_premium |
| alice    | alice    | user         |

To obtain the bearer token, run for instance the following command when on Linux (please make sure to have `curl` and `jq` packages available in your linux distribution):

```shell
export access_token=$(\
curl -X POST http://localhost:8180/realms/quickstart/protocol/openid-connect/token \
-H 'content-type: application/x-www-form-urlencoded' \
-d 'client_id=authz-servlet&client_secret=secret' \
-d 'username=jdoe&password=jdoe&grant_type=password' | jq --raw-output '.access_token' \
)
```

You can use the same command to obtain tokens on behalf of user `alice`, just make sure to change both `username` and `password` request parameters.

After running the command above, you can now access the `http://localhost:8080/protected/premium` endpoint
because the user `jdoe` has the `user_premium` role.

```shell
curl http://localhost:8080/protected/premium \
  -H "Authorization: Bearer "$access_token
```

As a result, you will see the following response from the service:

```
Hello, jdoe!
```

References
--------------------

* [Spring OAuth 2.0 Resource Server JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html)
* [Keycloak Authorization Services](https://www.keycloak.org/docs/latest/authorization_services/)
* [Keycloak Documentation](https://www.keycloak.org/documentation)