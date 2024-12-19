# Cluster Keycloak with MariaDB and LDAP using Docker Standalone

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/uml/cluster-keycloak-with-mariadb-and-ldap.svg)

s.o.:

* \[\[Cluster Keycloak with MariaDB and LDAP using Docker Swarm]]
* [Keycloak - Blog - Keycloak Cluster Setup](https://www.keycloak.org/2019/05/keycloak-cluster-setup.html)
* \[\[Master to Master Replication between two MariaDB Servers]]

Note : The whole setup entitles starting two types of Databases. MariaDB shall be the database used for Keycloak backend, whereas Postgres shall be the database used for Archive backend. The LDAP containers of both clusters are also synced which shall be used by archive containers for its configurations and by the Keycloak containers for accessing their User Federation.

1.  Create `docker-compose.env` for first and second clusters as follows :

    ```INI
    ARCHIVE_HOST=<host1-IP-addr>
    STORAGE_DIR=/storage/fs1
    POSTGRES_DB=pacsdb
    POSTGRES_USER=pacs
    POSTGRES_PASSWORD=pacs
    AUTH_SERVER_URL=http://keycloak:8843
    ```

    Note:

    * `AUTH_SERVER_URL` here uses `keycloak` in its hostname. On the first node, map this host `keycloak` to the IP Address of the first node in its `/etc/hosts` file. This is done in order to simulate a loadbalancer for two clusters.
    * Replace `<host1-IP-addr>` with IP address / hostname of _**first node**_
2.  Specify services for _**first**_ node in a configuration file `docker-compose.yml` (e.g.) :

    ```yaml
    version: "3"
    services:
      host1-ldap:
        image: dcm4che/slapd-dcm4chee:2.6.5-31.2
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
          - "389:389"
        env_file: docker-compose.env
        environment:
          LDAP_URLS: "ldap://host1-ldap/"
          LDAP_REPLICATION_HOSTS: "ldap://host1-ldap/ ldap://host2-ldap/"
        extra_hosts:
          - "host2-ldap:host2-IP-addr"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/ldap1:/var/lib/openldap/openldap-data
          - /var/local/dcm4chee-arc/slapd1.d:/etc/openldap/slapd.d
      mariadb:
        image: mariadb:10.7.3
        ports:
          - "3306:3306"
        environment:
          MYSQL_ROOT_PASSWORD: verys3cret
          MYSQL_DATABASE: keycloak
          MYSQL_USER: keycloak
          MYSQL_PASSWORD: keycloak
        command:
          - "--log-bin"
          - "--log-basename=host1"
          - "--server-id=1"
          - "--replicate-do-db=keycloak"
          - "--auto_increment_increment=2"
          - "--auto_increment_offset=1"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/mysql:/var/lib/mysql
      keycloak:
        image: dcm4che/keycloak:23.0.3
        logging:
          driver: json-file
          options:
            max-size: "10m"   
        ports:
          - "8843:8843"
          - "7600:7600"
        environment:
          KC_DB: mariadb
          KC_DB_URL: jdbc:mariadb://mariadb:3306/keycloak?serverTimezone=Europe/Vienna
          JGROUPS_DISCOVERY_EXTERNAL_IP: host1
          JGROUPS_DISCOVERY_PROTOCOL: TCPPING
          JGROUPS_DISCOVERY_INITIAL_HOSTS: "host2[7600],host1[7600]"
          KC_HTTPS_PORT: 8843
          KC_HOSTNAME: host1-IP-addr
          KEYCLOAK_ADMIN: admin
          KEYCLOAK_ADMIN_PASSWORD: changeit
          KC_LOG: file
          KEYCLOAK_WAIT_FOR: host1-ldap:389 mariadb:3306
        depends_on:
          - host1-ldap
          - mariadb
        extra_hosts:
          - "ldap:host1-IP-addr"
          - "host1:host1-IP-addr"
          - "host2:host2-IP-addr"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/keycloak:/opt/keycloak/data
      db:
        image: dcm4che/postgres-dcm4chee:15.4-31
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
          - "5432:5432"
        env_file: docker-compose.env
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/db:/var/lib/postgresql/data
      arc:
        image: dcm4che/dcm4chee-arc-psql:5.31.2-secure
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
          - "2575:2575"
        env_file: docker-compose.env
        extra_hosts:
          - "ldap:host1-IP-addr"
          - "host1:host1-IP-addr"
        environment:
          LDAP_URL: "ldap://host1-ldap/"
          WILDFLY_CHOWN: /opt/wildfly/standalone /storage
          WILDFLY_WAIT_FOR: host1-ldap:389 db:5432
        depends_on:
          - host1-ldap
          - keycloak
          - db
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/wildfly:/opt/wildfly/standalone
          - /var/local/dcm4chee-arc/storage:/storage
    ```

    replace _host1-IP-addr_ and _host2-IP-addr_ by the IP Addresses of the docker hosts where each of the two clusters are running, which must be resolvable by your DNS server.
3.  Specify services for _**second**_ node in a configuration file `docker-compose.yml` (e.g.) :

    ```yaml
    version: "3"
    services:
      host2-ldap:
        image: dcm4che/slapd-dcm4chee:2.6.5-31.2
        logging:
          driver: json-file
          options:
            max-size: "10m"
        ports:
          - "389:389"
        env_file: docker-compose.env
        environment:
          LDAP_URLS: "ldap://host2-ldap/"
          LDAP_REPLICATION_HOSTS: "ldap://host1-ldap/ ldap://host2-ldap/"
          SKIP_INIT_CONFIG: "true"
        extra_hosts:
          - "host1-ldap:host1-IP-addr"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc2/ldap2:/var/lib/openldap/openldap-data
          - /var/local/dcm4chee-arc2/slapd2.d:/etc/openldap/slapd.d
      mariadb:
        image: mariadb:10.7.3
        ports:
          - "3306:3306"
        environment:
          MYSQL_ROOT_PASSWORD: verys3cret
          MYSQL_DATABASE: keycloak
          MYSQL_USER: keycloak
          MYSQL_PASSWORD: keycloak
        command:
          - "--log-bin"
          - "--log-basename=host2"
          - "--server-id=2"
          - "--replicate-do-db=keycloak"
          - "--auto_increment_increment=2"
          - "--auto_increment_offset=2"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc2/mysql:/var/lib/mysql
      keycloak:
        image: dcm4che/keycloak:23.0.3
        logging:
          driver: json-file
          options:
            max-size: "10m"   
        ports:
          - "8843:8843"
          - "7600:7600"
        env_file: docker-compose.env
        environment:
          KC_DB: mariadb
          KC_DB_URL: jdbc:mariadb://mariadb:3306/keycloak?serverTimezone=Europe/Vienna
          JGROUPS_DISCOVERY_EXTERNAL_IP: host2
          JGROUPS_DISCOVERY_PROTOCOL: TCPPING
          JGROUPS_DISCOVERY_INITIAL_HOSTS: "host1[7600],host2[7600]"
          KC_HTTPS_PORT: 8843
          KC_HOSTNAME: host2-IP-addr
          KC_LOG: file
          KEYCLOAK_WAIT_FOR: host2-ldap:389 mariadb:3306
        depends_on:
          - host2-ldap
          - mariadb      
        extra_hosts:
          - "ldap:host2-IP-addr"
          - "host2:host2-IP-addr"
          - "host1:host1-IP-addr"
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc/keycloak:/opt/keycloak/data
      db:
        image: dcm4che/postgres-dcm4chee:15.4-31
        ports:
          - "5432:5432"
        env_file: docker-compose.env
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc2/db:/var/lib/postgresql/data
      arc:
        image: dcm4che/dcm4chee-arc-psql:5.31.2-secure
        ports:
          - "8080:8080"
          - "8443:8443"
          - "9990:9990"
          - "9993:9993"
          - "8787:8787"
          - "11112:11112"
          - "2575:2575"
        extra_hosts:
          - "ldap:host2-IP-addr"
          - "host2:host2-IP-addr"
        env_file: docker-compose.env
        environment:
          LDAP_URL: "ldap://host2-ldap/"
          WILDFLY_CHOWN: /opt/wildfly/standalone /storage
          WILDFLY_WAIT_FOR: host2-ldap:389 db:5432
        depends_on:
          - host2-ldap
          - keycloak
          - db
        volumes:
          - /etc/localtime:/etc/localtime:ro
          - /etc/timezone:/etc/timezone:ro
          - /var/local/dcm4chee-arc2/wildfly:/opt/wildfly/standalone
          - /var/local/dcm4chee-arc2/storage:/storage
    ```

    Note:

    * Replace _host1-IP-addr_ and _host2-IP-addr_ by the IP Addresses of the docker hosts where each of the two clusters are running, which must be resolvable by your DNS server.
    * In LDAP container configuration, we have used `SKIP_INIT_CONFIG: "true"`. This is to start LDAP without any configuration\
      on second host; on enabling replication as explained in following steps, it shall pull the configuration from first LDAP.
    * [KEYCLOAK\_ADMIN](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#keycloak_admin) / [KEYCLOAK\_ADMIN\_PASSWORD](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#keycloak_admin_password) are not required in the keycloak container settings on second node, as the user credentials to authenticate Keycloak's master realm are already setup and initialized by keycloak container settings on first node.
4.  Start `host1-ldap` and `mariadb` containers on first cluster

    ```
    docker-compose up -d host1-ldap mariadb
    ```
5.  Start `host2-ldap` and `mariadb` containers on second cluster

    ```
    docker-compose up -d host2-ldap mariadb
    ```
6. You may verify on second cluster, by logging in to its LDAP using [Apache Directory Studio](https://directory.apache.org/studio/), that it is started without any configuration in it, whereas on the LDAP of first cluster, it is initialized with default configuration of archive.
7.  Prepare replication for ldap on host 1 as follows :

    ```
    docker exec <host1-ldap-container-name> prepare-replication
    ```

    replace with the ldap container name on host1
8.  Prepare replication for ldap on host 2 as follows :

    ```
    docker exec <host2-ldap-container-name> prepare-replication
    ```

    replace with the ldap container name on host2
9.  Enable replication _**first for ldap on host 2**_, since we have used `SKIP_INIT_CONFIG: "true"`, as follows :

    ```
    docker exec <host2-ldap-container-name> enable-replication
    ```

    replace with the ldap container name on host2
10. Enable replication for ldap on host 1 as follows :

```
docker exec <host1-ldap-container-name> enable-replication
```

replace with the ldap container name on host1

11. You may test replication of LDAP was successful or not, by logging in to LDAP of second cluster using [Apache Directory Studio](https://directory.apache.org/studio/) and verify that it has pulled the configuration from LDAP of first cluster. Alternatively, also change value of some attribute on LDAP of first cluster and verify it gets reflected in LDAP of second cluster.
12. Enable \[\[Master to Master Replication between two MariaDB Servers]]. You may skip points 1 and 4 as we have already started the mariadb containers on the two clusters.
13. Once replications of LDAP and MariaDB are successful, start the `keycloak`, `db` (postgres) and `arc` containers on first cluster

```
docker-compose up -d keycloak db arc
```

14. Verify that Keycloak on first cluster is completely started and is up and running by verifying in the server logs as :

```
2023-01-20 08:35:01,084 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [608a234f883d-50184|0] (1) [608a234f883d-50184]
2023-01-20 08:35:01,101 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `608a234f883d-50184`, physical addresses are `[172.31.0.4:40072]`
2023-01-20 08:35:45,439 INFO  [org.keycloak.connections.infinispan.DefaultInfinispanConnectionProviderFactory] (main) Node name: 608a234f883d-50184, Site name: null
2023-01-20 08:35:45,543 INFO  [org.keycloak.broker.provider.AbstractIdentityProviderMapper] (main) Registering class org.keycloak.broker.provider.mappersync.ConfigSyncEventListener
2023-01-20 08:35:45,551 INFO  [org.keycloak.services] (main) KC-SERVICES0050: Initializing master realm
2023-01-20 08:35:49,777 INFO  [org.keycloak.services] (main) KC-SERVICES0004: Imported realm dcm4che from file /opt/keycloak/bin/../data/import/dcm4che-realm.json.
2023-01-20 08:35:50,194 INFO  [io.quarkus] (main) Keycloak 21.0.0 on JVM (powered by Quarkus 2.13.3.Final) started in 59.278s. Listening on: https://0.0.0.0:8843
2023-01-20 08:35:50,194 INFO  [io.quarkus] (main) Profile prod activated. 
2023-01-20 08:35:50,194 INFO  [io.quarkus] (main) Installed features: [agroal, cdi, hibernate-orm, jdbc-h2, jdbc-mariadb, jdbc-mssql, jdbc-mysql, jdbc-oracle, jdbc-postgresql, keycloak, logging-gelf, narayana-jta, reactive-routes, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-metrics, vault, vertx]
2023-01-20 08:35:50,378 INFO  [org.keycloak.services] (main) KC-SERVICES0009: Added user 'admin' to realm 'master'
```

This is done so that, when Keycloak is started on second cluster, it shall find the `dcm4che` realm in the clustered MariaDB database and shall not override the settings of the same.

15. Start the `keycloak`, `db` (postgres) and `arc` containers on second cluster.

```
docker-compose up -d keycloak db arc
```

Verify the logs of Keycloak on second cluster as explained in previous point.

```
2023-01-20 07:37:34,906 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000094: Received new cluster view for channel ISPN: [1d1ff27e148f-58712|0] (1) [1d1ff27e148f-58712]
2023-01-20 07:37:34,909 INFO  [org.infinispan.CLUSTER] (keycloak-cache-init) ISPN000079: Channel `ISPN` local address is `1d1ff27e148f-58712`, physical addresses are `[172.25.0.5:60822]`
2023-01-20 07:37:35,259 INFO  [org.keycloak.connections.infinispan.DefaultInfinispanConnectionProviderFactory] (main) Node name: 1d1ff27e148f-58712, Site name: null
2023-01-20 07:37:35,865 INFO  [org.keycloak.services] (main) KC-SERVICES0003: Not importing realm dcm4che from file /opt/keycloak/bin/../data/import/dcm4che-realm.json.  It already exists.
2023-01-20 07:37:35,878 INFO  [org.keycloak.services] (main) KC-SERVICES0003: Not importing realm dcm4che from file /opt/keycloak/bin/../data/import/dcm4che-realm.json.  It already exists.
2023-01-20 07:37:36,176 INFO  [io.quarkus] (main) Keycloak 21.0.0 on JVM (powered by Quarkus 2.13.3.Final) started in 9.549s. Listening on: https://0.0.0.0:8843
2023-01-20 07:37:36,176 INFO  [io.quarkus] (main) Profile prod activated. 
2023-01-20 07:37:36,176 INFO  [io.quarkus] (main) Installed features: [agroal, cdi, hibernate-orm, jdbc-h2, jdbc-mariadb, jdbc-mssql, jdbc-mysql, jdbc-oracle, jdbc-postgresql, keycloak, logging-gelf, narayana-jta, reactive-routes, resteasy, resteasy-jackson, smallrye-context-propagation, smallrye-health, smallrye-metrics, vault, vertx]
```

16. Once Keycloak and archive containers have started, proceed to Verify OIDC Client for Archive UI in Keycloak on Keycloak of first cluster. `Valid Redirect URIs` and `Web Origins` shall contain archive specific URLs of both clusters, (e.g.) :

```
Root URL             : https://host1:8443/dcm4chee-arc/ui2
Admin URL            : https://host1:8443/dcm4chee-arc/ui2
Valid Redirect URI   : http://host1:8080/dcm4chee-arc/ui2/*
                       https://host1:8443/dcm4chee-arc/ui2/*
                       http://host2:8080/dcm4chee-arc/ui2/*
                       https://host2:8443/dcm4chee-arc/ui2/*
Web Origins          : http://host1:8080
                       https://host1:8443
                       http://host2:8080
                       https://host2:8443
```

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-dcm4chee-arc-ui-clustered-setup.png)

17. Open two browsers (eg. one Firefox normal and one Firefox in Private mode) simultaneously and login to archive UIs of two nodes.

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-mariadb-ldap-db-clustered-arc-uis-login.png)

18. On Keycloak admin console, verify that the logged-in user has two sessions.

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-mariadb-ldap-db-arc-clustered-user-sessions.png)
