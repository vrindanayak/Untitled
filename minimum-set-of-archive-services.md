# Minimum Set of Archive services

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/uml/min-archive-services.svg)

**(Optional) Create system groups and users with particular group and user IDs used by the archive services on the host**

E.g.:

```
$ sudo -i
# groupadd -r slapd-dcm4chee --gid=1021 && useradd -r -g slapd-dcm4chee --uid=1021 slapd-dcm4chee
# groupadd -r postgres-dcm4chee --gid=999 && useradd -r -g postgres-dcm4chee --uid=999 postgres-dcm4chee
# groupadd -r dcm4chee-arc --gid=1023 && useradd -r -g dcm4chee-arc --uid=1023 dcm4chee-arc
# exit
```

Otherwise listed information of directory and files created by the containers in bind mounted host directories will show the numerical user and group IDs as owner information. E.g.:

```console
$ sudo ls -l /var/local/dcm4chee-arc/wildfly/
total 20
-rw-r--r-- 1 1023 1023    0 Sep  3 22:34 chown.done
drwxr-sr-x 4 1023 1023 4096 Sep  3 22:34 configuration
drwxr-sr-x 9 1023 1023 4096 Sep  3 22:34 data
drwxr-sr-x 2 1023 1023 4096 Sep  3 22:34 deployments
drwxr-sr-x 2 1023 1023 4096 Sep  3 22:34 log
drwxr-sr-x 5 1023 1023 4096 Sep  4 00:53 tmp
```

Continue using Docker Command Line or Docker Compose alternatively:

[**Use Docker Command Line**](https://docs.docker.com/engine/reference/commandline/cli/)

1.  **Create an user-defined bridge network**

    ```
    $ docker network create dcm4chee_network
    ```
2.  **Start OpenLDAP Server**

    Launch a container providing the LDAP server into the created network, e.g:

    ```
    $ docker run --network=dcm4chee_network --name ldap \
               -p 389:389 \
               -v /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data \
               -v /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d \
               -d dcm4che/slapd-dcm4chee:2.6.5-31.2
    ```

    Publishing ldap port `389` from the container to the host by `-p 389:389` enables to access the LDAP server also by external LDAP clients, like [Apache LDAP Directory Studio](http://directory.apache.org/studio/). It is not required for running the Archive, because the Archive container connects the LDAP port of the LDAP server container over the created bridge network.

    Bind mount `-v /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data` and `-v /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d` takes care to store the LDAP database and the OpenLDAP server configuration in the specified host directories. They are initialized on first container start-up if they are not already present in the specified host directories. That ensures that changes on the Archive configuration does not get lost on deletion and re-creation of the OpenLDAP Server container.

    The initialized LDAP data and OpenLDAP server configuration can be adjusted by setting [environment variables](https://github.com/dcm4che-dockerfiles/slapd-dcm4chee/blob/master/README.md#environment-variables). E.g.:

    ```
    -e LDAP_BASE_DN=dc=ihe,dc=net \
    -e LDAP_ROOTPASS=mypass \
    -e ARCHIVE_DEVICE_NAME=central-archive \
    -e AE_TITLE=CENTRAL \
    -e DICOM_PORT=10104 \
    -e HL7_PORT=12575
    -e STORAGE_DIR=/storage/fs1
    ```

    will use

    * `dc=ihe,dc=net` (default: `dc=dcm4che,dc=org`) as database suffix (or base DN) of the LDAP directory,
    * `mypass` (default: `secret`) as root password for accessing the LDAP directory,
    * `central-archive` (default: `dcm4chee-arc`) as Archive _Device Name_,
    * `CENTRAL` (default: `DCM4CHEE`) as primary _Application Entity Title_ of the Archive,
    * `10104` (default: `11112`) as listening port for DICOM connections to the Archive,
    * `12575` (default: `2575`) as listening port for the HL7 receiver of the Archive,
    * `/storage/fs1` (default: `/opt/wildfly/standalone/data/fs1`) as path to the directory - inside of the Archive container - where the Archive stores received DICOM objects.
3.  **Start PostgreSQL Server**

    Launch a container providing the database server into the created network, e.g:

    ```
    $ docker run --network=dcm4chee_network --name db \
               -p 5432:5432 \
               -e POSTGRES_DB=pacsdb \
               -e POSTGRES_USER=pacs \
               -e POSTGRES_PASSWORD=pacs \
               -v /etc/localtime:/etc/localtime:ro \
               -v /etc/timezone:/etc/timezone:ro \
               -v /var/local/dcm4chee-arc/db:/var/lib/postgresql/data \
               -d dcm4che/postgres-dcm4chee:15.4-31
    ```

    Publishing PostgreSQL Server port `5432` from the container to the host by `-p 5432:5432` enables to access the PostgreSQL Server also by external PostgreSQL clients. It is not required for running the Archive, because the Archive container connects the Server port of the PostgreSQL Server container over the created bridge network.

    ```
    -e POSTGRES_DB=pacsdb \
    -e POSTGRES_USER=pacs \
    -e POSTGRES_PASSWORD=pacs \
    ```

    set the name of the database, a user and its password.

    See further available [environment variables of the PostgreSQL container](https://github.com/dcm4che-dockerfiles/postgres-dcm4chee#environment-variables).

    Bind mount `-v /etc/localtime:/etc/localtime:ro` and `-v /etc/timezone:/etc/timezone:ro` duplicates your host timezone inside the container. Otherwise the container timezone is UTC. **Attention:** If there is no `/etc/timezone` on your host, you have to create one (e.g.: `$ echo "Europe/Vienna" > /etc/timezone`) before launching the container, otherwise the container will not start.

    Bind mount `-v /var/local/dcm4chee-arc/db:/var/lib/postgresql/data` takes care to store the database in the specified host directory. It is initialized on first container start-up if it is not already present in the specified host directory. That ensures that the data does not get lost on deletion and re-creation of the PostgreSQL Server container.
4.  **Start Wildfly with deployed dcm4che Archive 5 application**

    Launch a container providing Wildfly with deployed dcm4che Archive 5 application into the created network, e.g:

    ```
    $ docker run --network=dcm4chee_network --name arc \
               -p 8080:8080 \
               -p 8443:8443 \
               -p 9990:9990 \
               -p 9993:9993 \
               -p 11112:11112 \
               -p 2762:2762 \
               -p 2575:2575 \
               -p 12575:12575 \
               -e POSTGRES_DB=pacsdb \
               -e POSTGRES_USER=pacs \
               -e POSTGRES_PASSWORD=pacs \
               -e WILDFLY_WAIT_FOR="ldap:389 db:5432" \
               -v /etc/localtime:/etc/localtime:ro \
               -v /etc/timezone:/etc/timezone:ro \
               -v /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone \
               -d dcm4che/dcm4chee-arc-psql:5.31.2
    ```

    ```
    -p 8080:8080 \
    -p 8443:8443 \
    -p 9990:9990 \
    -p 9993:9993 \
    -p 11112:11112 \
    -p 2762:2762 \
    -p 2575:2575 \
    -p 12575:12575 \
    ```

    publishes the http (`8080`) and https (`8443`) port of the web server and the http (`9990`) and the https (`9993`) port of the WildFly Administration Console, and the DICOM (`11112`), DICOM TLS (`2762`), HL7 (`2575`) and HL7 TLS (`12575`) port of the Archive application from the container to the host to enable external http(s) clients, DICOM applications and HL7 senders to connect to WildFly and the Archive application.

    The environment variables:

    ```
    -e POSTGRES_DB=pacsdb \
    -e POSTGRES_USER=pacs \
    -e POSTGRES_PASSWORD=pacs \
    ```

    set database name, user and its password to connect to the PostgreSQL server, which have to match with the values specified for the PostgreSQL Server container. Otherwise the Archive application will fail to connect to the PostgreSQL server.

    If you set the environment variables `LDAP_BASE_DN`, `LDAP_ROOTPASS` or `ARCHIVE_DEVICE_NAME` for the LDAP server container, you have to set that environment variables also for the Archive container. Otherwise the Archive application will fail to connect to the LDAP Server or will not find its configuration in the LDAP directory.

    `-e WILDFLY_WAIT_FOR="ldap:389 db:5432"` delays the start of WildFly until OpenLDAP slapd and the PostgreSQL server are listening on the specified ports, which prevents error on restart of all containers caused by failures to connect to LDAP and PostgreSQL by the Archive application, before OpenLDAP slapd and PostgreSQL are ready to accept connections.

    Bind mount `-v /etc/localtime:/etc/localtime:ro` and `-v /etc/timezone:/etc/timezone:ro` duplicates your host timezone inside the container. Otherwise the container timezone is UTC. **Attention:** If there is no `/etc/timezone` on your host, you have to create one (e.g.: `$ echo "Europe/Vienna" > /etc/timezone`) before launching the container, otherwise the container will not start.

    Bind mount `-v /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone` takes care to store the Wildfly server configuration in the specified host directory. It is initialized on first container start-up if it is not already present in the specified host directory. That ensures that the data does not get lost on deletion and re-creation of the Archive Server container.

    If you set the environment variable `STORAGE_DIR` for the LDAP server container, which specifies the path to the directory where the Archive stores received DICOM objects, you should also bind mount that directory to a host directory, to ensure that the stored DICOM objects does not get lost on deletion and re-creation of the Archive Server container. E.g. if `STORAGE_DIR=/storage/fs1` you may bind mount `-v /var/local/dcm4chee-arc/storage:/storage`. You also have to take care that the system user running WilfFly in the Archive container has write access to that directory, either by creating the directory and changing its owner to that system user:

    ```
    $ sudo -i
    # mkdir -p /var/local/dcm4chee-arc/storage
    # chown 1023:1023 /var/local/dcm4chee-arc/storage
    # exit
    ```

    or by appending the default value of environment variable `WILDFLY_CHOWN=/opt/wildfly/standalone` with the path of the bind mounted directory (e.g. `WILDFLY_CHOWN="/opt/wildfly/standalone /storage"`).

    The default of `STORAGE_DIR=/opt/wildfly/standalone/data/fs1` of the LDAP container specifies a sub-directory of `/opt/wildfly/standalone` and is therefore already mapped out from the container to a host directory by `-v /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone`.

    By default the https services will use the certificate ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/cert.png) for handling TLS connections. To avoid the security warning of Web Browsers accessing the Archive UI or the WildFly Administration Console, you have to provide a certificate which _Common Name_ and/or _Subject Alt Name_ matches the host name and which is signed by a trusted issuer in a keystore file in [PKCS #12](https://en.wikipedia.org/wiki/PKCS_12) or [Java KeyStore (JKS)](https://www.pixelstech.net/article/1409966488-Different-types-of-keystore-in-Java----JKS) format, bind mount the file or its containing directory into the container, and refer it by environment variable [KEYSTORE](https://github.com/dcm4che-dockerfiles/dcm4chee-arc-psql#keystore).

    See further available [environment variables of the Archive container](https://github.com/dcm4che-dockerfiles/dcm4chee-arc-psql#environment-variables).

    Check the Wildfly server log if the Archive started successfully:

    ```console
    $ tail -f /var/local/dcm4chee-arc/wildfly/log/server.log
    2022-06-09 14:03:58,946 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Start TCP Listener on /0.0.0.0:11112
    2022-06-09 14:03:58,946 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Start TCP Listener on /0.0.0.0:2575
    2022-06-09 14:03:59,048 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Start TCP Listener on /0.0.0.0:12575
    2022-06-09 14:03:59,048 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Start TCP Listener on /0.0.0.0:2762
    2022-06-09 14:03:59,211 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 46) WFLYSRV0010: Deployed "dcm4chee-arc-ui2-5.31.2-secure.war" (runtime-name : "dcm4chee-arc-ui2-5.31.2-secure.war")
    2022-06-09 14:03:59,211 INFO  [org.jboss.as.server] (ServerService Thread Pool -- 46) WFLYSRV0010: Deployed "dcm4chee-arc-ear-5.31.2-psql-secure.ear" (runtime-name : "dcm4chee-arc-ear-5.31.2-psql-secure.ear")
    2022-06-09 14:03:59,238 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0212: Resuming server
    2022-06-09 14:03:59,246 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 27.0.1.Final (WildFly Core 18.1.1.Final) started in 10685ms - Started 2819 of 3031 services (446 services are lazy, passive or on-demand) - Server configuration file in use: dcm4chee-arc-oidc.xml
    2022-06-09 14:03:59,248 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0062: Http management interface listening on http://0.0.0.0:9990/management and https://0.0.0.0:9993/management
    2022-06-09 14:03:59,248 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0053: Admin console listening on http://0.0.0.0:9990 and https://0.0.0.0:9993
    ```
5.  **(Optional) Send CT images to the Archive using** [**dcm4che/dcm4che-tools**](https://github.com/dcm4che-dockerfiles/dcm4che-tools)

    ```console
    $ docker run --rm --network=dcm4chee_network dcm4che/dcm4che-tools storescu -cDCM4CHEE@arc:11112 /opt/dcm4che/etc/testdata/dicom
    Scanning files to send
    ................
    Scanned 16 files in 0.053s (=3ms/file)
    17:20:53,552 INFO  - Initiate connection from 0.0.0.0/0.0.0.0:0 to 192.168.2.131:11112
    17:20:53,560 INFO  - Established connection Socket[addr=/192.168.2.131,port=11112,localport=59369]
    17:20:53,570 DEBUG - /172.18.0.2:59369->/192.168.2.131:11112(1): enter state: Sta4 - Awaiting transport connection opening to complete
    17:20:53,571 INFO  - STORESCU->DCM4CHEE(1) << A-ASSOCIATE-RQ
    :
    17:20:54,657 INFO  - STORESCU->DCM4CHEE(1) >> A-RELEASE-RP
    17:20:54,657 INFO  - STORESCU->DCM4CHEE(1): close Socket[addr=/192.168.2.131,port=11112,localport=59369]
    17:20:54,657 DEBUG - STORESCU->DCM4CHEE(1): enter state: Sta1 - Idle
    Sent 16 objects (=2.027MB) in 1.06s (=1.912MB/s)
    ```

    The received images should show up in the UI of the Archive at http://localhost:8080/dcm4chee-arc/ui2 or https://localhost:8443/dcm4chee-arc/ui2: ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/received_ct_images.png)
6.  **You may stop all 3 containers by:**

    ```
    $ docker stop ldap db arc
    ```

    and start all 3 containers again by:

    ```
    $ docker start ldap db arc
    ```
7.  **You may delete the stopped containers by**

    ```
    $ docker rm -v ldap db arc
    ```

    You may delete the created bridge network by

    ```
    $ docker network rm dcm4chee_network
    ```

**Use** [**Docker Compose**](https://docs.docker.com/compose)

Alternatively to Docker Command Line one may use [Docker Compose](https://docs.docker.com/compose/) to take care for starting all 3 containers:

1. **(Optional) Create system groups and users with particular group and user IDs used by the archive services as described above.**
2.  **Specify the services in a configuration file `docker-compose.yml` (e.g.):**

    ```yaml
    version: "3"
    services:
      ldap:
        image: dcm4che/slapd-dcm4chee:2.6.5-31.2
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
          - "389:389"
        environment:
          STORAGE_DIR: /storage/fs1
        volumes:
          - /var/local/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data
          - /var/local/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d
      db:
        image: dcm4che/postgres-dcm4chee:15.4-31
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
         - "5432:5432"
        environment:
          POSTGRES_DB: pacsdb
          POSTGRES_USER: pacs
          POSTGRES_PASSWORD: pacs
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/db:/var/lib/postgresql/data
      arc:
        image: dcm4che/dcm4chee-arc-psql:5.31.2
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
          - "8080:8080"
          - "8443:8443"
          - "9990:9990"
          - "9993:9993"
          - "11112:11112"
          - "2762:2762"
          - "2575:2575"
          - "12575:12575"
        environment:
          POSTGRES_DB: pacsdb
          POSTGRES_USER: pacs
          POSTGRES_PASSWORD: pacs
          WILDFLY_CHOWN: /storage
          WILDFLY_WAIT_FOR: ldap:389 db:5432
        depends_on:
          - ldap
          - db
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone
          - /var/local/dcm4chee-arc/storage:/storage
    ```

    See available environment variables for

    * [Slapd container](https://github.com/dcm4che-dockerfiles/slapd-dcm4chee/blob/master/README.md#environment-variables)
    * [PostgreSQL container](https://github.com/dcm4che-dockerfiles/postgres-dcm4chee#environment-variables)
    * [Archive container](https://github.com/dcm4che-dockerfiles/dcm4chee-arc-psql#environment-variables)

    Note : If there are difficulties starting the archive service due to

    ```
    There is insufficient memory for the Java Runtime Environment to continue. Cannot create GC thread. Out of system resources.
    ```

    adding the following option to the `arc` service

    ```
    security_opt:
     - seccomp:unconfined
    ```
3.  **Create and start the 3 containers by invoking**

    ```console
    $ docker-compose -p dcm4chee up -d
    Creating network "dcm4chee_network" with the default driver
    Creating dcm4chee_ldap_1 ... 
    Creating dcm4chee_db_1 ... 
    Creating dcm4chee_ldap_1
    Creating dcm4chee_db_1 ... done
    Creating dcm4chee_arc_1 ... 
    Creating dcm4chee_arc_1 ... done
    ```

    in the directory containing `docker-compose.yml`.
4.  **You may stop all 3 containers by:**

    ```console
    $ docker-compose -p dcm4chee stop
    Stopping dcm4chee_arc_1  ... done
    Stopping dcm4chee_db_1   ... done
    Stopping dcm4chee_ldap_1 ... done
    ```

    and start all 3 containers again by:

    ```console
    $ docker-compose -p dcm4chee start
    Starting db   ... done
    Starting ldap ... done
    Starting arc  ... done
    ```
5.  **You may stop and delete all 3 containers by**

    ```console
    $ docker-compose -p dcm4chee down
    Stopping dcm4chee_arc_1  ... done
    Stopping dcm4chee_db_1   ... done
    Stopping dcm4chee_ldap_1 ... done
    Removing dcm4chee_arc_1  ... done
    Removing dcm4chee_db_1   ... done
    Removing dcm4chee_ldap_1 ... done
    Removing network dcm4chee_network
    ```
