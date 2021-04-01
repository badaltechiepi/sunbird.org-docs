---
title: Build and Deploy
page_title: Build and Deploy
description: Build and Deploy
published: true
allowSearch: true
keywords: Upgrade, Sunbird 3.7.0, Core, elasticsearch
---

## Overview

This page details out the jobs required to be run as part of the upgrade from Sunbird release 3.6.0 to release 3.7.0. Use the following table to understand the jobs that need to be executed in order to successfully complete the upgrade. 

## Running the Builds 

**IMPORTANT**: 

1. Take a backup of all your databases by running backup jobs located under OpsAdministration → Core / KnowledgePlatform / DataPipeline → BackupJobs OR You can use VM Disk Snapshots from your cloud provider.

2. Once all the variables and Jenkins configurations are complete, you can start to build and deploy services.

3. Build and deploy the plugins. For details, refer to the  [Plugins](developer-docs/server-installation/plugins){:target="_blank"} page

4. After the plugins build and deploy, provide the blob url of the plugins zip file in the Player build. For details, refer to the Player build job on the [Core Services](developer-docs/server-installation/artifactupload-job/core-services){:target="_blank"} page.

5. Build all the services mentioned in the table below. Refer to [Current Release Tags and Jenkins Jobs Reference](developer-docs/server-installation/current_release_tags_n_jenkins_jobs/index.html){:target="_blank"}

The following is the list of jobs required to be built and deployed :

Order: Top down per column

| KP Build | KP Deploy         | DataPipeline Build | DatePipeline Deploy | Core Build and Deploy | Plugins Build and Deploy|
|----------|-------------------|--------------------|---------------------|-----------------------|-------------------------|
| Learning | CassandraDbUpdate | AnalyticsCore      | AnalyticsCore       | Cassandra             | ContentPlayer   |
| Yarn     | Neo4jDefinitionUpdate| CoreDataproducts| CoreDataproducts    | Content               ||
| FlinkJob | Learning          | EdDataProducts     | EdDataProducts      | Dial                  | |
|          | KafkaSetup        | FlinkPipelineJobs  | FlinkPipelineJobs   | Groups                | |
|          | Yarn              |                    |                     | Learner               | |
|          |  FlinkJob         |                    |                     | LMS                   |    |
|          |                   |      |                                   | KnowledgeMW           |    |
|          |                   |      |                                   | Player                |    |
|          |                   |      |                                   | Keycloak              |    |                                                                                                                                               
|          |                   |      |                                   | nginx-public-ingress  |    |
|          |                   |      |                                   | nginx-private-ingress  |    |

**Create License and update Channel default License set and Content License migration based on channel**

1.Run the following script on the content-service Pod to create License .
 
```
"Copy below code and save file name as create_license"
#!/bin/bash
echo ""License name: $1""
echo ""License descripition: $2""
echo ""License url: $3""
curl -X POST \
  http://localhost:9000/license/v3/create \
  -H 'Content-Type: application/json' \
  -d '{
    ""request"":{
        ""license"":{
            ""name"": ""'$1'"",
            ""description"": ""'$2'"",
            ""url"": ""'$3'""
        }
    }
}'

Run these commands
bash -x script CC\ BY-NC-SA\ 4.0 This\ license\ is\ Creative\ Commons\ Attribution-NonCommercial-ShareAlike https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode
bash -x script CC\ BY-NC\ 4.0 This\ license\ is\ Creative\ Commons\ Attribution-NonCommercial  https://creativecommons.org/licenses/by-nc/4.0/legalcode
bash -x script CC\ BY-SA\ 4.0 This\ license\ is\ Creative\ Commons\ Attribution-ShareAlike https://creativecommons.org/licenses/by-sa/4.0/legalcode
bash -x script CC\ BY\ 4.0 This\ is\ the\ standard\ license\ of\ any\ Youtube\ content https://creativecommons.org/licenses/by/4.0/legalcode
bash -x script Standard\ YouTube\ License This\ license\ is\ Creative\ Commons\ Attribution-NonCommercial-ShareAlike https://www.youtube.com/"        
```

2.Update all content with valid license .  

```
CREATE INDEX ON :domain(mimeType);
CREATE INDEX ON :domain(license);
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.license='Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)' SET n.license='CC BY-NC 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.license='Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)' SET n.license='CC BY-NC 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.license='Creative Commons Attribution-ShareAlike (CC BY-SA)' SET n.license='CC BY-SA 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.license='Creative Commons Attribution-ShareAlike (CC BY-SA)' SET n.license='CC BY-SA 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.ecml-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.html-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.android.package-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.content-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.content-collection' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.plugin-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/vnd.ekstep.h5p-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/epub' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='text/x-url' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/x-youtube' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/octet-stream' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/msword' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='application/pdf' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/jpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/jpg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/png' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/tiff' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/bmp' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/gif' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='image/svg+xml' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/avi' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/quicktime' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/3gpp' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/mp4' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/ogg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='video/webm' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/mp3' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/mp4' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/ogg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/webm' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/x-wav' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where n.mimeType='audio/wav' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.ecml-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.html-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.android.package-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.content-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.content-collection' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.plugin-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/vnd.ekstep.h5p-archive' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/epub' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='text/x-url' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/x-youtube' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/octet-stream' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/msword' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='application/pdf' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/jpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/jpg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/png' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/tiff' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/bmp' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/gif' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='image/svg+xml' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/avi' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/quicktime' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/3gpp' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/mp4' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/ogg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='video/webm' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/mp3' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/mp4' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/mpeg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/ogg' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/webm' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/x-wav' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where n.mimeType='audio/wav' AND not(n.license in ['CC BY-NC 4.0', 'CC BY-SA 4.0', 'Standard YouTube License']) SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.ecml-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.html-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.android.package-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.content-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.content-collection' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.plugin-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.h5p-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/epub' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='text/x-url' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/x-youtube' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/octet-stream' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/msword' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='application/pdf' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/jpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/jpg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/png' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/tiff' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/bmp' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/gif' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='image/svg+xml' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/avi' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/quicktime' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/3gpp' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/mp4' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/ogg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='video/webm' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/mp3' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/mp4' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/ogg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/webm' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/x-wav' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'Content'}) where not(exists(n.license)) AND n.mimeType='audio/wav' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.ecml-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.html-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.android.package-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.content-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.content-collection' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.plugin-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/vnd.ekstep.h5p-archive' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/epub' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='text/x-url' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/x-youtube' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/octet-stream' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/msword' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='application/pdf' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/jpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/jpg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/png' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/tiff' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/bmp' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/gif' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='image/svg+xml' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/avi' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/quicktime' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/3gpp' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/mp4' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/ogg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='video/webm' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/mp3' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/mp4' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/mpeg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/ogg' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/webm' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/x-wav' SET n.license='CC BY 4.0';
match (n:domain{IL_FUNC_OBJECT_TYPE:'ContentImage'}) where not(exists(n.license)) AND n.mimeType='audio/wav' SET n.license='CC BY 4.0';
match (n:domain{}) where n.IL_FUNC_OBJECT_TYPE in ['Content', 'ContentImage'] return n.license, count(n);
DROP INDEX ON :domain(mimeType);
DROP INDEX ON :domain(license);
```
Copy above code and save it as migration.txt in neo4j server in tha path /home/learning/neo4j-learning/neo4j-enterprise-3.3.0/bin

run the command to update content with valid license
```
cat migration.txt | ./cypher-shell
```

3.Run the above script from learning server to update the Channel-default license .  

```
Copy below code and save file name as channel_license.sh
#!/bin/bash
echo "Channel Id: $1"
echo "Default License: $2"
IFS=,
curl -X PATCH \
"http://localhost:8080/channel/v3/update/"$1 \
-H 'Content-Type: application/json' \
-d '{
   "request": {
      "channel": {
        "defaultLicense":"'$2'"
      }
    }
}'

Run the Command
./channel_license.sh <channle_Id> <license_name>

license_name can be choosen from the list which was created in the 1st Step.
```

4.Update content with channel specific default license, Run the below query in neo4j

```
match (n:domain{}) WHERE n.IL_FUNC_OBJECT_TYPE IN ["Content", "ContentImage"] AND n.channel="<channel id>" AND n.license<>"Standard YouTube License" SET n.license="<channel defaultLicense>";
```

**Clear Redis cache**

Run the below command in LP redis server

```
cd /home/learning/redis-stable/src
./redis-cli keys do_* | xargs ./redis-cli del
```
5.Kong 9-14 migration

```
export PG_DB="api_manager_kong14"

docker container rm kong-migration

docker run -it --name kong-migration \
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




