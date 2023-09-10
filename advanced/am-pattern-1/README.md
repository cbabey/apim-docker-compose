# Pattern 1: Docker Compose Resource for Standard HA Deployment of WSO2 API Manager 

This deployment consists of an API-M cluster with two nodes of the API-M runtime.

![GitHub Logo](https://apim.docs.wso2.com/en/4.0.0/assets/img/setup-and-install/active-active-apim-deployment.png)

## Prerequisites

 * Install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), [Docker](https://www.docker.com/get-docker) and [Docker Compose](https://docs.docker.com/compose/install/#install-compose)
   in order to run the steps provided in the following Quick Start guide. <br><br>
 * In order to use Docker images with WSO2 updates, you need an active WSO2 subscription.
   Otherwise, you can proceed with Docker images available at [DockerHub](https://hub.docker.com/u/wso2/), which are created using GA releases.<br><br>
 * If you wish to run the Docker Compose setup using Docker images built locally, build Docker images using Docker resources available from [here](../../dockerfiles/) and remove the `docker.wso2.com/` prefix from the `image` name in the `docker-compose.yml`. <br><br>

## Quick Start Guide

1. Clone this docker-compose Git repository.

   ```
   git clone https://github.com/cbabey/apim-docker-compose
   ```
2. Checkout to the 4.0.x branch 

   ```
   git checkout 4.0.x
   ```

3. Switch to `apim-docker-compose/advanced/am-pattern-1` folder.

   ```
   cd apim-docker-compose/advanced/am-pattern-1
   ```
4. Download the APIM-4.0.0 binary distribution from the wso2official website and copy it into the `apim-docker-compose``/advanced`/am-pattern-1/dockerfiles` directory. (If you need to try out the flow in a specific U2 update level, update the downloaded binary distribution into the required) 
   > This Docker Compose configuration is set up to create a custom Docker image by utilizing a binary that has been copied into the /dockerfiles directory. This approach ensures compatibility with various operating systems, including Mac M1, as the resulting Docker image will be tailored to the specific runtime environment. For guidance on configuring this setup to utilize WSO2 official Docker images, please refer to advanced configuration. 

5. Add a DNS record mapping the hostnames and the loopback IP or instance IP.

      The APIM services are made accessible externally through a configured Nginx server. By default, servlet ports are exposed using the hostname api.am.wso2.com, while passthrough ports are exposed via gw.am.wso2.com. If you are running this setup on your local machine, you can add a DNS mapping for api.am.wso2.com and gw.am.wso2.com to point to the loopback IP address (127.0.0.1) in your etc/hosts file. Alternatively, if you are not in a local environment, you can configure these hostnames to map to the appropriate instance IP address.

6. Execute the following Docker Compose command to start the deployment.

   ```
   docker-compose up --build
   ```

7. Access the WSO2 API Manager web UIs using the below URLs via a web browser.

   ```
   https://api.am.wso2.com/publisher
   https://api.am.wso2.com/devportal
   https://api.am.wso2.com/admin
   https://api.am.wso2.com/carbon
   ```
   Login to the web UIs using the following credentials.
   
   * Username: admin <br>
   * Password: admin

   Please note that API Gateway will be available on the following ports.
   ```
   https://gw.am.wso2.com
   http://gw.am.wso2.com
   ```

9. To see analytics data, log in to [Choreo Analytics](https://analytics.choreo.dev/).

## Advanced Configuration 

### Building APIM Custom Docker Image

This Docker Compose configuration is designed to be compatible with any operating system, including Macs with Silicon chips. To achieve this compatibility, a custom Docker image is first built using a binary distribution of the API Management. This ensures that the Docker image is built to be compatible with the specific operating system on which the Docker compose resources are run.

This custom docker images are built using the official dockerfiles and the following changes were made on top of the official docker files. 

- The official docker images are implemented to fetch the APIM binary distribution from the APIM official docker repository, it was updated to fetch from the local file system. 

```diff
# add the WSO2 product distribution to user's home directory
+COPY --chown=wso2carbon:wso2 wso2am-4.0.0.zip /
RUN \
-    wget -O ${WSO2_SERVER}.zip "${WSO2_SERVER_DIST_URL}" \
    unzip -d ${USER_HOME} ${WSO2_SERVER}.zip \
    && chown wso2carbon:wso2 -R ${WSO2_SERVER_HOME} \
    && mkdir ${USER_HOME}/wso2-tmp \
    && bash -c 'mkdir -p ${USER_HOME}/solr/{indexed-data,database}' \
    && chown wso2carbon:wso2 -R ${USER_HOME}/solr \
    && cp -r ${WSO2_SERVER_HOME}/repository/deployment/server/synapse-configs ${USER_HOME}/wso2-tmp \
    && cp -r ${WSO2_SERVER_HOME}/repository/deployment/server/executionplans ${USER_HOME}/wso2-tmp \
    && rm -f ${WSO2_SERVER}.zip

```
- Download the Mysql jar and add it to the docker image 

```diff
+ADD --chown=wso2carbon:wso2 https://repo1.maven.org/maven2/mysql/mysql-connector-java/${MYSQL_CONNECTOR_VERSION}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}.jar ${WSO2_SERVER_HOME}/repository/components/dropins/
```
- Add the keystore and trust store into the docker image

```diff 
+COPY --chown=wso2carbon:wso2 ./certificates/client-truststore.jks ${WSO2_SERVER_HOME}/repository/resources/security
+COPY --chown=wso2carbon:wso2 ./certificates/wso2carbon.jks ${WSO2_SERVER_HOME}/repository/resources/security
```

### Certificate Manage

In this docker-compose resources, A single self-sign certificate was generated by defining *.am.wso2.com as a while card entry and adding a localhost as a SAN entry. This certificate was used in the nginx load balancer configuration to generate the TLS keystore used for the APIM.



We create customer Docker images for both Nginx and APIM by including the necessary certificate and keystore files. The folder structure containing the Docker files and certificates is as follows:


```
└── dockerfiles
    ├── apim
    │   ├── Dockerfile
    │   ├── certificates
    │   │   ├── client-truststore.jks
    │   │   └── wso2carbon.jks
    │   ├── docker-entrypoint.sh
    │   └── wso2am-4.0.0.zip
    └── nginx
        ├── Dockerfile
        └── certificates
            ├── server.crt
            └── server.key
```

#### Generating Self-Signed Certificate

You can follow the below steps to generate the self-sign certificate required for the nginx server.

Step 1: Generate an RSA Key Pair
   ```
   openssl genrsa -aes256 -passout pass:gsahdg -out server.pass.key 4096
   ```
Step 2: Remove the Passphrase from the RSA Key
   ```
   openssl rsa -passin pass:gsahdg -in server.pass.key -out server.key
   ```
Step 3: Generate a Certificate Signing Request (CSR)
   ```
   openssl req -new -key server.key -out server.csr -subj "/C=US/ST=California/L=Los Angeles/O=Example Inc./OU=IT Department/CN=*.am.wso2.com"

   ```
Step 4: Generate a Self-Signed X.509 Certificate (the localhost was added as a SAN entry)
   ```
   openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt -extfile <(printf "subjectAltName=DNS:localhost")
   ```

#### Generate Keystore using 

You can follow the below steps to generate a keystore using the above generated Self-signed Certificate.

Step 1: Create a PKCS12 File
```
openssl pkcs12 -export -in server.crt -inkey server.key -name "wso2carbon" -out wso2carbon.pfx
```
Step 2: Convert the PKCS12 File to a Java Keystore (JKS)

```
keytool -importkeystore -srckeystore wso2carbon.pfx -srcstoretype pkcs12 -destkeystore wso2carbon.jks -deststoretype JKS
```

