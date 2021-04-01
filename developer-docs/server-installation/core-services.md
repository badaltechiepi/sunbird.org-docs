---
title: Core Services
page_title: Core Services
description: Explains how to create schemas for Core service objects
keywords: core services, services
allowSearch: true
---

## Overview
This page explains the jobs to be run to bring up the Core services. 

## Prerequisites

1.Create a container in the Azure blob and make it public to publish content. The container name must be the same as the variable **sunbird_content_azure_storage_container** specified in the **common.yml** file present in the core inventory.

2.Switch to the `Build` folder and run all jobs. Set the value for **github_release_tag** according to the details mentioned in this page - [Current Release Tags and Jenkins Jobs Reference](developer-docs/server-installation/current_release_tags_n_jenkins_jobs){:target="_blank"}

## Running the Jobs
    
> Note: Adhere to the following sequence when running the jobs.

### DevOps Administration

| Operation Name | Function | 
|--------------------|-----------| 
| Bootstrap | Creates Deployer User | 

### Builds

| Build Name | Function (Builds) | CURRENT RELEASE TAG |
|--------------------|-----------| ------------------- 
| Adminutils | Adminutils docker image | release-3.7.0_RC1
| API MANAGER | API manager docker image | release-3.7.0_RC14
| API MANAGER Echo | API manager echo docker image | master
| Badger | Badger docker image |
| Cassandra | Jar for migration| release-3.7.0_RC1 |
| Lms | LMS service docker image | release-3.7.0_RC3
| Dial    | Generate the artifacts for dial service | release-3.7.0_RC2
| Content | Content service docker image | release-3.7.0_RC1
| Search  | search service docker image |release-3.7.0_RC1
| KnowledgeMW | Knowledge-mw service docker image | release-3.7.0_RC2
| Learner | Learner service docker image | release-3.7.0_RC12
| Player | Player service docker image. |  release-3.7.0_RC38
| Cert | Cert service docker image | release-1.1.1
| EncService | Envservice docker image | release-1.1.1
| Proxy | Proxy docker image | release-3.7.0_RC15
| Telemetry | Telemetry docker image | release-2.1.0
| Keycloak | Keycloak zip file | release-3.6.0_RC1|
| Analytics | Analytics service docker image |  release-3.4.0
| Taxonomy  | Taxonomy service docker image | ​release-3.7.0_RC1

### Artifacts  
Ensure that all Artifacts are uploaded

### Provision

| Operation Name | Function | Description | CURRENT RELEASE TAG |
|--------------------|-----------|---------| ------------------- |
| (Deploy) ApplicationES( Kubernetes folder) | Provisions Elasticsearch 6.2.3 version used by core services | From the Deploy Folder, **Deploy ApplicationES** provisions for the Elasticsearch and creates indices necessary for Sunbird Core| release-3.7.0_RC15
| ESMapping (Under OpsAdministarion. Provide the value as *all* for job parameter indices_name) | Creates Elasticsearch indexes | Creates the specific index with its mapping or all indices which will be used by our sunbird app | release-3.7.0_RC15
| Postgres | Provisions for Postgres | Provisions the Postgres instance | release-3.7.0_RC15
| PostgresDbUpdate | Creates the databases, assigns roles and creates users | Creates a schema with DB's, Tables | release-3.7.0_RC15 **Perform Kong migration once db update is done, steps are mentioned in the kong migration section**
| EsReindexing (Under OpsAdministario, parameters should be esHost: es ip, oldIndex: certreg, newIndex: certv2, aliasName: certs, indexFileLocation: sunbird-es-utils/src/main/resources/reindexing/indices/certreg_indices.json, mappingFileLocation: sunbird-es-utils/src/main/resources/reindexing/mappings/certreg_mappings.json) | --- | --- | release-3.7.0_RC1

### Deploy

| Operation Name | Function | Description |  CURRENT RELEASE TAG |
|------|------|---------------------------------| --------------|
|BootstrapMinimal |creating the namespace for kubernetes orchestration and also Bootstrap minimal create docker registry secrets and nginx ssls also | Creates namespace for the core service | release-3.7.0_RC15 
|nginx-private-ingress |xxxx | xxxx | release-3.7.0_RC4
| API Manager | Deploys the API manager Kong and API manager Echo | Manages consumers and APIs of sunbird |  ​release-3.7.0_RC14
| OnboardAPIS | Onboards all API's to Sunbird | onboards sunbird API's |  ​release-3.7.0_RC15
| OnboardConsumers | Onboards sunbird consumers |onboards sunbird consumers |  ​release-3.7.0_RC15
Update **core_vault_sunbird_api_auth_token**, **core_vault_kong__test_jwt** and **core_vault_sunbird_ekstep_api_key** with the **jwt token** from the Jenkins output of **api-management-test-user** if you are using the Knowledge Platform and Data Pipeline along with core| Generates user specific key |  Onboards new consumer to Sunbird and generates the consumer specific API key | 
| (Provision) Cassandra |Provisions Cassandra and create keyspaces required for Sunbird Core | Provisions Cassandra and creates keyspaces and performs migration | release-3.7.0_RC15|
| Cassandra | Performs keyspace schema migration | Performs migration if required. Deploy this thrice by choosing different zip files using the build_number parameter. Ensure that you get a success message for the Cassandra migration on the Jenkins console output. Do not rely only on the red or green status indicator on Jenkins job | release-3.7.0_RC4
|(Provision) Keycloak | Install dependencies for keycloak | Provisions Keycloak by installing prerequisites like Java and environment variables | release-3.7.0_RC15
| Keycloak | Deploys Keycloak service to VM | Centralized user management for sunbird | release-3.7.0_RC4
| KeycloakRealm(Core folder) |  User management -Creates sunbird realm | Creates a Sunbird Realm. After the Sunbird realm is created, configure Keycloak by using the steps mentioned in the **Keycloak Configuration** section | release-3.7.0_RC15
|Adminutil | Deploys the Adminutil container | Creates tokens for the sunbird devices | ​release-3.7.0_RC15
| Player |  UI for sunbird  | Deploys portal UI | release-3.7.0_RC5
| Learner | Deploys the Learner Service |  Handles user management and helps to search org | release-3.7.0_RC4 **before deploying learner service Create root org by using the steps mentioned in the Create Org section below**
| Content | Deploys content service | Helps to create content | release-3.7.0_RC4
| Search              | Deploys the search service artefacts and starts the search service | |release-3.7.0_RC15|
| Dial              | Installs the dependency to run the dial service | |release-3.7.0_RC4
| KnowledgeMW | Deploys knowledgemw service | Deploys the knowledgemw service | release-3.7.0_RC4
| Lms | Deploys LMS Service | Provides the APIs for LMS functionality of Sunbird| release-3.7.0_RC4
| EncService | Deploys enc service | Encrypts and decrypt the keys to generate certificate | release-3.7.0_RC15
| Cert | Deploys cert services | Issues certificates | release-3.1.0
| Telemetry | Aggregates and send telemetry data to kafka | Telemetry management service| release-3.7.0_RC15
| Analytics | Deploys Analytics service | Deploys Analytics service | release-3.7.0_RC15
| Taxonomy | Deploys Taxonomy service | Deploys Taxonomy service | release-3.7.0_RC15 after deploying taxonomy service create master category and run definition scripts by using the steps mentioned in **Create master category** section
|nginx-public-ingress | Deploys Proxy service | Deploys Nginx | release-3.7.0_RC4 
|BootstrapMinimal |creating the namespace for kubernetes orchestration  and also Bootstrap minimal create docker registry secrets and nginx ssls also| Creates namespace for the core service | release-3.7.0_RC15 

### Keycloak Configuration 

> Note: From release-2.0.0 keycloak admin portal is disabled from public internet. 
You must tunnel the port in to the local machine via ssh tunnelling.  
`ssh -L 8080:localhost:8080 ops@~keycloak-ip-address~`  
You can access keycloak via `localhost:8080`

| Step | Action  |
|------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | Login to Keycloak using the user name **admin** and password as given in the private **secrets.yml** file. Or, login to keycloak using &#60;localhost:8080&#62;/auth |
| 2 | Navigate to Sunbird Realm > Realm Settings > Keys. Click `Public Key`. Copy the key value that you see and update the variable core_vault_sso_public_key |
| 3 | Creating keycloak federation [Deployment Steps for Keycloak User Federation](developer-docs/server-installation/keycloak_user_federation) |
| 4 | Creating keycloak lms client [Creation Steps for Keycloak lms client](developer-docs/server-installation/keycloak_lms_configuration) |
| 5 | Create Keycloak Password policy: navigate to Authentication > Password Policy > from add policy drop down select `Hashing Iterations, Minimum Length, Special Characters, Uppercase Characters, Lowercase Characters`
| 6 | Get the value for variable adminutil_refresh_token_public_key_kid [Steps to get the value for variable](developer-docs/server-installation/adminutil_refresh_token) |


**Note:**
If the Cassandra migration fails, run the query manually to set the corresponding version for the failed migration to True

**Example:**

`SELECT * from sunbird.cassandra_migration_version;`

Check the rows for which the value in the success column is False. The following is an example -

`1.80 |   180685665 |   cassandra |              4 |         null | 2019-12-01 13:58:52.401000+0000 |            136 | V1.80_cassandra.cql |   False |  CQL |           80`

Run the update query for each row separately 

`update sunbird.cassandra_migration_version set success=True where version ='1.80';`

Verify that all the values in the success column are True and rerun the Jenkins job again with same zip file and tag

Once this succeeds, use the second zip file and tag to deploy again

The current migration version is 1.88. The output of the Jenkins job should be as follows -

`Migrating keyspace Sunbird to version 1.88 - Cassandra
Successfully applied 3 migrations to keyspace sunbird (execution time 00:20.547s).
Migration Completed at ==1571996508540`


### Create Root org

```
   # Create root org
      # cqlsh
      INSERT INTO sunbird.organisation (id,channel,orgname,hashtagid,isrootorg) values ('1234567890','default', 'default','1234567890',true);
      
      # from Elastic search
      # Sync to es
      curl -X PUT "localhost:9200/org/_doc/1234567890?pretty" -H 'Content-Type: application/json' -d'
      {
        "id": "1234567890",
        "channel": "default",
        "orgName": "default",
        "hashTagId": "1234567890",
        "isRootOrg": true
      }'
      
      #cqlsh
      # Insert in to system settings
      INSERT INTO sunbird.system_settings (id,field,value) values ('custodianOrgId','custodianOrgId','1234567890');
      INSERT INTO sunbird.system_settings (id,field,value) values ('custodianOrgChannel','custodianOrgChannel','default');

```

### Knong migration

```
   set or export postgres variables
   export PG_HOST="postgres ip"         
   export PG_USER="postgres"                    
   export PGPASSWORD="pgpassword"   #  copy value from core_vault_postgres_password variable       
   export PG_DB="kong db name"   # by default db name will be api_manager_{{env}}, verify in postgres database for knong db name              
   
   docker run --name kong-migration \
   -e ""KONG_DATABASE=postgres"" \
   -e ""KONG_PG_DATABASE=$PG_DB"" \
   -e ""KONG_PG_HOST=$PG_HOST"" \
   -e ""KONG_PG_PASSWORD=$PGPASSWORD"" \
   -e ""KONG_PG_USER=$PG_USER"" \
   -e ""KONG_PROXY_ACCESS_LOG=/dev/stdout"" \
   -e ""KONG_ADMIN_ACCESS_LOG=/dev/stdout"" \
   -e ""KONG_PROXY_ERROR_LOG=/dev/stderr"" \
   -e ""KONG_ADMIN_ERROR_LOG=/dev/stderr"" \
   -e ""KONG_ADMIN_LISTEN=0.0.0.0:8001"" \
   -p 8000:8000 \
   -p 8001:8001 \
   kong:0.14.1 kong migrations up --vv

```

### Create master category

1.login to taxonomy service container and execute the curl commands to create master category. curl commands are availabe in the file [master_category_create](https://github.com/project-sunbird/knowledge-platform/blob/release-3.4.0/definition-scripts/master_category_create) 

2.Replace the host to http://localhost:9000 while running

3.Run other defintion scripts from taxonomy service [definition-scripts](https://github.com/project-sunbird/knowledge-platform/tree/release-3.4.0/definition-scripts) Since master_category_create is already ran do not run again.
