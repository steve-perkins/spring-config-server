spring-config-server
====================
This is one of three Git repositories, which work together to demonstrate using 
[Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/) and [Vault](https://www.vaultproject.io) for 
configuration management:

* https://github.com/steve-perkins/spring-config-properties - Contains:
  * Properties files for global default properties that are available to all applications 
    (i.e. `application.propeties`), 
  * per-application files with properties that are common to that application in every environment 
    (e.g. `sampleapp.properties`), and 
  * environment-specific properties for each application (e.g. `sampleapp-staging.properties`).
* https://github.com/steve-perkins/spring-config-server - An instance of Spring Cloud Config server, configured 
  to serve up *non-secret* properties from the Git repo above, and *secret* properties from Vault.
* https://github.com/steve-perkins/spring-config-sample-app - A sample web application which uses the Spring Cloud 
  Config client to retrieve its config properties from the server above.
  
This demo shows the use case of having two types of config properties:

1. **non-secret** values, which can and should be maintainable by developer teams (e.g. JDBC URL's).
2. **secret** values, which should only be viewable or maintainable by people with specialized access (e.g. 
   usernames and passwords)
   
The non-secret values are stored as-is in the `spring-config-properties` repo.  The *secret* values are manually 
written to Vault.  At runtime, Spring Cloud Config retrieves properties from both sources, giving Vault the higher 
precedence whenever the same property is found in both.

Setup
=====
* Download [Vault](https://www.vaultproject.io/downloads.html).  It requires no installation, just copy the 
  executable to some location locally.
* In a command-line shell, startup Vault like this:

```
vault server -dev
```

* In another shell, execute the following commands:

```
export VAULT_ADDR=http://127.0.0.1:8200 (Linux / OS X)
or
set VAULT_ADDR=http://127.0.0.1:8200 (Windows)
```
... and then...
```
vault auth-enable userpass
vault policy-write writers writers.hcl
vault write auth/userpass/users/vault_user password=vault_pass policies=writers

vault write secret/sampleapp,dev jdbcUsername=devuser jdbcPassword=devpass
vault write secret/sampleapp,staging jdbcUsername=staginguser jdbcPassword=stagingpass
vault write secret/sampleapp,prod jdbcUsername=scott jdbcPassword=tiger
```

The `writers.hcl` file is located in the root of this reps.  This policy grants our test user access to a path branch 
in Vault's tree-like structure.  In real life, you might want to create separate policies for the various environments 
and/or applications (at *some* level of granularity), so that you can control access and prevent one application from 
being able to read another one's secrets.

* This repo is a Gradle-based project, containing a [Spring Boot](https://projects.spring.io/spring-boot/)-based 
   app with a `bootRun` Gradle task for launching it.
   
```
gradlew bootRun 
```

* Proceed to the steps described in the 
[spring-config-sample-app](https://github.com/steve-perkins/spring-config-sample-app) project README.
