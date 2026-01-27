* [Overview](#overview)
* [Functionality](#functionality)
* [Recommended System Requirements](#recommended-system-requirements)
* [Running Docker Image](#running-docker-image)
* [Configuring Docker Image](#configuring-docker-image)
    - [Storing Data](#storing-data)
    - [Running UNIVAULTOFFICE Document Server on Different Port](#running-univaultoffice-document-server-on-different-port)
    - [Running UNIVAULTOFFICE Document Server using HTTPS](#running-univaultoffice-document-server-using-https)
        + [Generation of Self Signed Certificates](#generation-of-self-signed-certificates)
        + [Strengthening the Server Security](#strengthening-the-server-security)
        + [Installation of the SSL Certificates](#installation-of-the-ssl-certificates)
        + [Available Configuration Parameters](#available-configuration-parameters)
* [Installing UNIVAULTOFFICE Document Server integrated with Community and Mail Servers](#installing-univaultoffice-document-server-integrated-with-community-and-mail-servers)
* [UNIVAULTOFFICE Document Server ipv6 setup](#univaultoffice-document-server-ipv6-setup)
* [Issues](#issues)
    - [Docker Issues](#docker-issues)
    - [Document Server usage Issues](#document-server-usage-issues)
* [Project Information](#project-information)
* [User Feedback and Support](#user-feedback-and-support)

## Overview

UNIVAULTOFFICE Docs (Document Server) is an open-source office suite that comprises all the tools you need to work with documents, spreadsheets, presentations, PDFs, and PDF forms. The suite supports office files of all popular formats (DOCX, ODT, XLSX, ODS, CSV, PPTX, ODP, etc.) and enables collaborative editing in real time.

Starting from version 6.0, Document Server is distributed as UNIVAULTOFFICE Docs. It has [three editions](https://github.com/UnivaultOffice/DocumentServer#univaultoffice-docs-editions). With this image, you will install the free Community version. 

UNIVAULTOFFICE Docs can be used as a part of [UNIVAULTOFFICE DocSpace](https://www.univaultoffice.github.io/docspace.aspx) and UNIVAULTOFFICE Workspace, or with [third-party sync&share solutions](https://www.univaultoffice.github.io/all-connectors.aspx) (e.g. Odoo, Moodle, Nextcloud, ownCloud, Seafile, etc.) to enable collaborative editing within their interface.

***Important*** Please update `docker-engine` to latest version (`20.10.21` as of writing this doc) before using it. We use `ubuntu:24.04` as base image and it older versions of docker have compatibility problems with it

## Functionality ##

Take advantage of the powerful editors included in UNIVAULTOFFICE Docs:

* [UNIVAULTOFFICE Document Editor](https://www.univaultoffice.github.io/document-editor.aspx)
* [UNIVAULTOFFICE Spreadsheet Editor](https://www.univaultoffice.github.io/spreadsheet-editor.aspx)
* [UNIVAULTOFFICE Presentation Editor](https://www.univaultoffice.github.io/presentation-editor.aspx)
* [UNIVAULTOFFICE Form Creator](https://www.univaultoffice.github.io/form-creator.aspx)
* [UNIVAULTOFFICE PDF Editor](https://www.univaultoffice.github.io/pdf-editor.aspx)
* [UNIVAULTOFFICE Diagram Viewer](https://www.univaultoffice.github.io/diagram-viewer.aspx) 

The editors empower you to create, edit, save, and export text docs, sheets, presentations, PDFs, create and fill out PDF forms, open diagrams, all while offering additional advanced features such as:

* Collaborative editing (review & track changes, comments, chat)
* [AI-powered assistants](https://www.univaultoffice.github.io/ai-assistants.aspx) 
* Spell-checking 
* Scalable UI options (including dark mode)
* [Security tools & services](https://www.univaultoffice.github.io/security.aspx)

UNIVAULTOFFICE Docs offer support for plugins allowing you to add specific features to the editors that are not directly related to the OOXML format. For more details, see [our API](https://api.univaultoffice.github.io/docs/plugin-and-macros/get-started/overview/) or visit the [plugins repo](https://github.com/UnivaultOffice/univaultoffice.github.io). Would like to explore the existing plugins? Open the [Marketplace](https://www.univaultoffice.github.io/app-directory).

## Recommended System Requirements

* **RAM**: 4 GB or more
* **CPU**: dual-core 2 GHz or higher
* **Swap**: at least 2 GB
* **HDD**: at least 2 GB of free space
* **Distribution**: 64-bit Red Hat, CentOS or other compatible distributive with kernel version 3.8 or later, 64-bit Debian, Ubuntu or other compatible distributive with kernel version 3.8 or later
* **Docker**: version 1.9.0 or later

## Running Docker Image

    sudo docker run -i -t -d -p 80:80 univaultoffice/documentserver

Use this command if you wish to install UNIVAULTOFFICE Document Server separately. To install UNIVAULTOFFICE Document Server integrated with Community and Mail Servers, refer to the corresponding instructions below.

## Configuring Docker Image

### Storing Data

All the data are stored in the specially-designated directories, **data volumes**, at the following location:
* **/var/log/univaultoffice** for UNIVAULTOFFICE Document Server logs
* **/var/www/univaultoffice/Data** for certificates
* **/var/lib/univaultoffice** for file cache
* **/var/lib/postgresql** for database

To get access to your data from outside the container, you need to mount the volumes. It can be done by specifying the '-v' option in the docker run command.

    sudo docker run -i -t -d -p 80:80 \
        -v /app/univaultoffice/DocumentServer/logs:/var/log/univaultoffice  \
        -v /app/univaultoffice/DocumentServer/data:/var/www/univaultoffice/Data  \
        -v /app/univaultoffice/DocumentServer/lib:/var/lib/univaultoffice \
        -v /app/univaultoffice/DocumentServer/rabbitmq:/var/lib/rabbitmq \
        -v /app/univaultoffice/DocumentServer/redis:/var/lib/redis \
        -v /app/univaultoffice/DocumentServer/db:/var/lib/postgresql  univaultoffice/documentserver

Normally, you do not need to store container data because the container's operation does not depend on its state. Saving data will be useful:
* For easy access to container data, such as logs
* To remove the limit on the size of the data inside the container
* When using services launched outside the container such as PostgreSQL, Redis, RabbitMQ

### Running UNIVAULTOFFICE Document Server on Different Port

To change the port, use the -p command. E.g.: to make your portal accessible via port 8080 execute the following command:

    sudo docker run -i -t -d -p 8080:80 univaultoffice/documentserver

### Running UNIVAULTOFFICE Document Server using HTTPS

        sudo docker run -i -t -d -p 443:443 \
        -v /app/univaultoffice/DocumentServer/data:/var/www/univaultoffice/Data  univaultoffice/documentserver

Access to the UNIVAULTOFFICE application can be secured using SSL so as to prevent unauthorized access. While a CA certified SSL certificate allows for verification of trust via the CA, a self-signed certificate can also provide an equal level of trust verification as long as each client takes some additional steps to verify the identity of your website. Below the instructions on achieving this are provided.

To secure the application via SSL basically two things are needed:

- **Private key (.key)**
- **SSL certificate (.crt)**

So you need to create and install the following files:

        /app/univaultoffice/DocumentServer/data/certs/tls.key
        /app/univaultoffice/DocumentServer/data/certs/tls.crt

When using CA certified certificates (e.g. [Let's Encrypt](https://letsencrypt.org)), these files are provided to you by the CA. If you are using self-signed certificates you need to generate these files [yourself](#generation-of-self-signed-certificates).

#### Using the automatically generated Let's Encrypt SSL Certificates

        sudo docker run -i -t -d -p 80:80 -p 443:443 \
        -e LETS_ENCRYPT_DOMAIN=your_domain -e LETS_ENCRYPT_MAIL=your_mail  univaultoffice/documentserver

If you want to get and extend Let's Encrypt SSL Certificates automatically just set LETS_ENCRYPT_DOMAIN and LETS_ENCRYPT_MAIL variables.

#### Generation of Self Signed Certificates

Generation of self-signed SSL certificates involves a simple 3 step procedure.

**STEP 1**: Create the server private key

```bash
openssl genrsa -out tls.key 2048
```

**STEP 2**: Create the certificate signing request (CSR)

```bash
openssl req -new -key tls.key -out tls.csr
```

**STEP 3**: Sign the certificate using the private key and CSR

```bash
openssl x509 -req -days 365 -in tls.csr -signkey tls.key -out tls.crt
```

You have now generated an SSL certificate that's valid for 365 days.

#### Strengthening the server security

This section provides you with instructions to [strengthen your server security](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html).
To achieve this you need to generate stronger DHE parameters.

```bash
openssl dhparam -out dhparam.pem 2048
```

#### Installation of the SSL Certificates

Out of the four files generated above, you need to install the `tls.key`, `tls.crt` and `dhparam.pem` files at the UNIVAULTOFFICE server. The CSR file is not needed, but do make sure you safely backup the file (in case you ever need it again).

The default path that the UNIVAULTOFFICE application is configured to look for the SSL certificates is at `/var/www/univaultoffice/Data/certs`, this can however be changed using the `SSL_KEY_PATH`, `SSL_CERTIFICATE_PATH` and `SSL_DHPARAM_PATH` configuration options.

The `/var/www/univaultoffice/Data/` path is the path of the data store, which means that you have to create a folder named certs inside `/app/univaultoffice/DocumentServer/data/` and copy the files into it and as a measure of security you will update the permission on the `tls.key` file to only be readable by the owner.

```bash
mkdir -p /app/univaultoffice/DocumentServer/data/certs
cp tls.key /app/univaultoffice/DocumentServer/data/certs/
cp tls.crt /app/univaultoffice/DocumentServer/data/certs/
cp dhparam.pem /app/univaultoffice/DocumentServer/data/certs/
chmod 400 /app/univaultoffice/DocumentServer/data/certs/tls.key
```

You are now just one step away from having our application secured.

#### Available Configuration Parameters

*Please refer the docker run command options for the `--env-file` flag where you can specify all required environment variables in a single file. This will save you from writing a potentially long docker run command.*

Below is the complete list of parameters that can be set using environment variables.

- **UNIVAULTOFFICE_HTTPS_HSTS_ENABLED**: Advanced configuration option for turning off the HSTS configuration. Applicable only when SSL is in use. Defaults to `true`.
- **UNIVAULTOFFICE_HTTPS_HSTS_MAXAGE**: Advanced configuration option for setting the HSTS max-age in the UNIVAULTOFFICE nginx vHost configuration. Applicable only when SSL is in use. Defaults to `31536000`.
- **SSL_CERTIFICATE_PATH**: The path to the SSL certificate to use. Defaults to `/var/www/univaultoffice/Data/certs/tls.crt`.
- **SSL_KEY_PATH**: The path to the SSL certificate's private key. Defaults to `/var/www/univaultoffice/Data/certs/tls.key`.
- **SSL_DHPARAM_PATH**: The path to the Diffie-Hellman parameter. Defaults to `/var/www/univaultoffice/Data/certs/dhparam.pem`.
- **SSL_VERIFY_CLIENT**: Enable verification of client certificates using the `CA_CERTIFICATES_PATH` file. Defaults to `false`
- **NODE_EXTRA_CA_CERTS**: The [NODE_EXTRA_CA_CERTS](https://nodejs.org/api/cli.html#node_extra_ca_certsfile "Node.js documentation") to extend CAs with the extra certificates for Node.js. Defaults to `/var/www/univaultoffice/Data/certs/extra-ca-certs.pem`.
- **DB_TYPE**: The database type. Supported values are `postgres`, `mariadb`, `mysql`, `mssql` or `oracle`. Defaults to `postgres`.
- **DB_HOST**: The IP address or the name of the host where the database server is running.
- **DB_PORT**: The database server port number.
- **DB_NAME**: The name of a database to use. Should be existing on container startup.
- **DB_USER**: The new user name with superuser permissions for the database account.
- **DB_PWD**: The password set for the database account.
- **DB_SCHEMA**: Database schema name (optional).  
  - **PostgreSQL** — schema for [search_path](https://www.postgresql.org/docs/current/ddl-schemas.html#DDL-SCHEMAS-PATH), default `public`.  
  - **MSSQL** — schema to set as [DEFAULT_SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-user-transact-sql?view=sql-server-ver17#default_schema---schema_name--null-), default `dbo`.  
- **AMQP_URI**: The [AMQP URI](https://www.rabbitmq.com/uri-spec.html "RabbitMQ URI Specification") to connect to message broker server.
- **AMQP_TYPE**: The message broker type. Supported values are `rabbitmq` or `activemq`. Defaults to `rabbitmq`.
- **REDIS_SERVER_HOST**: The IP address or the name of the host where the Redis server is running.
- **REDIS_SERVER_PORT**:  The Redis server port number.
- **REDIS_SERVER_USER**: The Redis server username. The username is not set by default.
- **REDIS_SERVER_PASS**: The Redis server password. The password is not set by default.
- **REDIS_SERVER_DB**: The Redis database index number to select. Defaults to `0`.  
- **NGINX_WORKER_PROCESSES**: Defines the number of nginx worker processes.
- **NGINX_WORKER_CONNECTIONS**: Sets the maximum number of simultaneous connections that can be opened by a nginx worker process.
- **NGINX_ACCESS_LOG**: Defines whether access logging is enabled. Defaults to `false`.
- **SECURE_LINK_SECRET**: Defines secret for the nginx config directive [secure_link_md5](https://nginx.org/en/docs/http/ngx_http_secure_link_module.html#secure_link_md5). Defaults to `random string`.
- **JWT_ENABLED**: Specifies the enabling the JSON Web Token validation by the UNIVAULTOFFICE Document Server. Defaults to `true`.
- **JWT_SECRET**: Defines the secret key to validate the JSON Web Token in the request to the UNIVAULTOFFICE Document Server. Defaults to random value.
- **JWT_HEADER**: Defines the http header that will be used to send the JSON Web Token. Defaults to `Authorization`.
- **JWT_IN_BODY**: Specifies the enabling the token validation in the request body to the UNIVAULTOFFICE Document Server. Defaults to `false`.
- **WOPI_ENABLED**: Specifies the enabling the wopi handlers. Defaults to `false`.
- **ALLOW_META_IP_ADDRESS**: Defines if it is allowed to connect meta IP address or not. Defaults to `false`.
- **ALLOW_PRIVATE_IP_ADDRESS**: Defines if it is allowed to connect private IP address or not. Defaults to `false`.
- **USE_UNAUTHORIZED_STORAGE**: Set to `true` if using self-signed certificates for your storage server e.g. Nextcloud. Defaults to `false`
- **GENERATE_FONTS**: When 'true' regenerates fonts list and the fonts thumbnails etc. at each start. Defaults to `true`
- **METRICS_ENABLED**: Specifies the enabling StatsD for UNIVAULTOFFICE Document Server. Defaults to `false`.
- **METRICS_HOST**: Defines StatsD listening host. Defaults to `localhost`.
- **METRICS_PORT**: Defines StatsD listening port. Defaults to `8125`.
- **METRICS_PREFIX**: Defines StatsD metrics prefix for backend services. Defaults to `ds.`.
- **LETS_ENCRYPT_DOMAIN**: Defines the domain for Let's Encrypt certificate.
- **LETS_ENCRYPT_MAIL**: Defines the domain administrator mail address for Let's Encrypt certificate.
- **PLUGINS_ENABLED**: Defines whether to enable default plugins. Defaults to `true`.

## Installing UNIVAULTOFFICE Document Server using Docker Compose

You can also install UNIVAULTOFFICE Document Server using [docker-compose](https://docs.docker.com/compose/install "docker-compose"). 

First you need to clone this [GitHub repository](https://github.com/UnivaultOffice/Docker-DocumentServer/):

```bash
git clone https://github.com/UnivaultOffice/Docker-DocumentServer
```

After that switch to the repository folder:

```bash
cd Docker-DocumentServer
```

After that, assuming you have docker-compose installed, execute the following command:

```bash
docker-compose up -d
```

## Installing UNIVAULTOFFICE Document Server as a part of UNIVAULTOFFICE Workspace

UNIVAULTOFFICE Document Server is a part of UNIVAULTOFFICE Workspace that comprises also Community Server, Mail Server, and Control Panel. To install them, follow these easy steps:

**STEP 1**: Create the `univaultoffice` network.

```bash
docker network create --driver bridge univaultoffice
```
Then launch containers on it using the 'docker run --net univaultoffice' option:

**STEP 2**: Install MySQL.

Follow [these steps](#installing-mysql) to install MySQL server.

**STEP 3**: Generate JWT Secret

JWT secret defines the secret key to validate the JSON Web Token in the request to the **UNIVAULTOFFICE Document Server**. You can specify it yourself or easily get it using the command:
```
JWT_SECRET=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 12);
```

**STEP 4**: Install UNIVAULTOFFICE Document Server.

```bash
sudo docker run --net univaultoffice -i -t -d --restart=always --name univaultoffice-document-server \
 -e JWT_ENABLED=true \
 -e JWT_SECRET=${JWT_SECRET} \
 -e JWT_HEADER=AuthorizationJwt \
 -v /app/univaultoffice/DocumentServer/logs:/var/log/univaultoffice  \
 -v /app/univaultoffice/DocumentServer/data:/var/www/univaultoffice/Data  \
 -v /app/univaultoffice/DocumentServer/lib:/var/lib/univaultoffice \
 -v /app/univaultoffice/DocumentServer/db:/var/lib/postgresql \
 univaultoffice/documentserver
```

**STEP 5**: Install UNIVAULTOFFICE Mail Server. 

For the mail server correct work you need to specify its hostname 'yourdomain.com'.

```bash
sudo docker run --init --net univaultoffice --privileged -i -t -d --restart=always --name univaultoffice-mail-server -p 25:25 -p 143:143 -p 587:587 \
 -e MYSQL_SERVER=univaultoffice-mysql-server \
 -e MYSQL_SERVER_PORT=3306 \
 -e MYSQL_ROOT_USER=root \
 -e MYSQL_ROOT_PASSWD=my-secret-pw \
 -e MYSQL_SERVER_DB_NAME=univaultoffice_mailserver \
 -v /app/univaultoffice/MailServer/data:/var/vmail \
 -v /app/univaultoffice/MailServer/data/certs:/etc/pki/tls/mailserver \
 -v /app/univaultoffice/MailServer/logs:/var/log \
 -h yourdomain.com \
 univaultoffice/mailserver
```

The additional parameters for mail server are available [here](https://github.com/UnivaultOffice/Docker-CommunityServer/blob/master/docker-compose.workspace_enterprise.yml#L87).

To learn more, refer to the [UNIVAULTOFFICE Mail Server documentation](https://github.com/UnivaultOffice/Docker-MailServer "UNIVAULTOFFICE Mail Server documentation").

**STEP 6**: Install UNIVAULTOFFICE Community Server

```bash
sudo docker run --net univaultoffice -i -t -d --privileged --restart=always --name univaultoffice-community-server -p 80:80 -p 443:443 -p 5222:5222 --cgroupns=host \
 -e MYSQL_SERVER_ROOT_PASSWORD=my-secret-pw \
 -e MYSQL_SERVER_DB_NAME=univaultoffice \
 -e MYSQL_SERVER_HOST=univaultoffice-mysql-server \
 -e MYSQL_SERVER_USER=univaultoffice_user \
 -e MYSQL_SERVER_PASS=univaultoffice_pass \
 
 -e DOCUMENT_SERVER_PORT_80_TCP_ADDR=univaultoffice-document-server \
 -e DOCUMENT_SERVER_JWT_ENABLED=true \
 -e DOCUMENT_SERVER_JWT_SECRET=${JWT_SECRET} \
 -e DOCUMENT_SERVER_JWT_HEADER=AuthorizationJwt \
 
 -e MAIL_SERVER_API_HOST=${MAIL_SERVER_IP} \
 -e MAIL_SERVER_DB_HOST=univaultoffice-mysql-server \
 -e MAIL_SERVER_DB_NAME=univaultoffice_mailserver \
 -e MAIL_SERVER_DB_PORT=3306 \
 -e MAIL_SERVER_DB_USER=root \
 -e MAIL_SERVER_DB_PASS=my-secret-pw \ 
 -e CONTROL_PANEL_PORT_80_TCP=80 \
 -e CONTROL_PANEL_PORT_80_TCP_ADDR=univaultoffice-control-panel \
 -v /app/univaultoffice/CommunityServer/data:/var/www/univaultoffice/Data \
 -v /app/univaultoffice/CommunityServer/logs:/var/log/univaultoffice \
 -v /app/univaultoffice/CommunityServer/letsencrypt:/etc/letsencrypt \
 -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
 univaultoffice/communityserver
```

Where `${MAIL_SERVER_IP}` is the IP address for **UNIVAULTOFFICE Mail Server**. You can easily get it using the command:
```
MAIL_SERVER_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' univaultoffice-mail-server)
```

Alternatively, you can use an automatic installation script to install UNIVAULTOFFICE Workspace at once. For the mail server correct work you need to specify its hostname 'yourdomain.com'.

**STEP 1**: Download the UNIVAULTOFFICE Workspace Docker script file

```bash
wget https://github.com/UnivaultOffice/DocumentServer/releases/latest/download/workspace-install.sh
```

**STEP 2**: Install UNIVAULTOFFICE Workspace executing the following command:

```bash
workspace-install.sh -md yourdomain.com
```

Or, use [docker-compose](https://docs.docker.com/compose/install "docker-compose"). First you need to clone this [GitHub repository](https://github.com/UnivaultOffice/Docker-CommunityServer/):

```bash
wget https://raw.githubusercontent.com/UNIVAULTOFFICE/Docker-CommunityServer/master/docker-compose.groups.yml
docker-compose up -d
```

## UNIVAULTOFFICE Document Server ipv6 setup

(Works and is supported only for Linux hosts)

Docker does not currently provide ipv6 addresses to containers by default. This function is experimental now.

To set up interaction via ipv6, you need to enable support for this feature in your Docker. For this you need:
- create the `/etc/docker/daemon.json` file with the following content:

```
{
"ipv6": true,
"fixed-cidr-v6": "2001:db8:abc1::/64"
}
```
- restart docker with the following command: `systemctl restart docker`

After that, all running containers receive an ipv6 address and have an inet6 interface.

You can check your default bridge network and see the field there
`EnableIPv6=true`. A new ipv6 subnet will also be added.

For more information, visit the official [Docker manual site](https://docs.docker.com/config/daemon/ipv6/)

## Issues

### Docker Issues

As a relatively new project Docker is being worked on and actively developed by its community. So it's recommended to use the latest version of Docker, because the issues that you encounter might have already been fixed with a newer Docker release.

The known Docker issue with UNIVAULTOFFICE Document Server with rpm-based distributives is that sometimes the processes fail to start inside Docker container. Fedora and RHEL/CentOS users should try disabling SELinux with setenforce 0. If it fixes the issue then you can either stick with SELinux disabled which is not recommended by Red Hat, or switch to using Ubuntu.

### Document Server usage issues

Due to the operational characteristic, **Document Server** saves a document only after the document has been closed by all the users who edited it. To avoid data loss, you must forcefully disconnect the **Document Server** users when you need to stop **Document Server** in cases of the application update, server reboot etc. To do that, execute the following script on the server where **Document Server** is installed:

```
sudo docker exec <CONTAINER> documentserver-prepare4shutdown.sh
```

Please note, that both executing the script and disconnecting users may take a long time (up to 5 minutes).

## Project Information

Official website: [www.univaultoffice.github.io](https://www.univaultoffice.github.io/?utm_source=github&utm_medium=cpc&utm_campaign=GitHubDockerDS)

Code repository: [github.com/UnivaultOffice/DocumentServer](https://github.com/UnivaultOffice/DocumentServer "https://github.com/UnivaultOffice/DocumentServer")

Docker Image: [github.com/UnivaultOffice/Docker-DocumentServer](https://github.com/UnivaultOffice/Docker-DocumentServer "https://github.com/UnivaultOffice/Docker-DocumentServer")

License: [GNU AGPL v3.0](https://onlyo.co/38YZGJh)

Free version vs commercial builds comparison: https://github.com/UnivaultOffice/DocumentServer#univaultoffice-docs-editions

## User Feedback and Support

If you face any issues or have questions about this image, visit our official forum: [forum.univaultoffice.github.io][1].

You are also welcome to ask and answer UNIVAULTOFFICE development questions on [Stack Overflow][2], as well as share your suggestions on [feedback.univaultoffice.github.io](https://feedback.univaultoffice.github.io/forums/966080-your-voice-matters).

Join [our Discord community](https://discord.gg/Hcgtf5n4uF) for connecting with fellow developers.

  [1]: https://forum.univaultoffice.github.io
  [2]: https://stackoverflow.com/questions/tagged/univaultoffice