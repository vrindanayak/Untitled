# Deploy Docker Images to Kubernetes

Alternatively to \[\[Running on Docker]], Docker images may also be deployed to a Kubernetes Cluster.

Following steps describes how to deploy to a local Kubernetes Cluster provided by [Canonical MicroK8s](https://microk8s.io/).

#### Install MicroK8s

So far only [installation as Snap package](https://microk8s.io/docs) on Ubuntu 20.04.4 LTS was tested:

1.  Install Snap package

    ```
    sudo snap install microk8s --classic --channel=1.24/stable
    ```
2.  Join the group

    ```
    sudo usermod -a -G microk8s $USER
    sudo chown -f -R $USER ~/.kube
    ```

    Re-enter the session for the group update to take place:

    ```
    su - $USER
    ```
3.  Check the status

    ```
    microk8s status --wait-ready
    ```
4.  Enable `dns` add-ons with IP of DNS server

    ```
    microk8s enable dns:192.168.2.10
    ```

    (for multiple DNS addresses, a comma-separated list should be used)
5.  Enable `storage` add-ons

    ```
    microk8s enable storage
    ```
6.  Verify if the host name of the node is resolve-able by coreDNS [deploying gcr.io/kubernetes-e2e-test-images/dnsutils:1.3](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

    ```
    microk8s kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
    ```

    and lookup for the host name of this node:

    ```
    microk8s kubectl exec -i -t dnsutils -- nslookup test-ng
    ```

    ```
    Server:  10.152.183.10
    Address: 10.152.183.10#53

    Name:    test-ng
    Address: 192.168.2.145
    ```

#### (Optional) [Setup a multi-node cluster](https://microk8s.io/docs/clustering)

If you want to distribute the individual archive services over several nodes, you have to install MicroK8s on each of the nodes, but you only need to enable `dns` and `storage` add-ons and adjust the coredns configure on the initial node.

For each additional node invoke

```
microk8s add-node
```

on the initial node, which prints a microk8s join command which should be executed on the MicroK8s instance that you wish to join to the cluster, e.g.:

```
microk8s join 192.168.2.145:25000/e27a206f7347358adb9719b1588694b8
```

#### Create [Opaque Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#opaque-secrets) for accessing

*   LDAP, e.g.:

    ```
    microk8s kubectl create secret generic ldap-secret \
      --from-literal=LDAP_ROOTPASS=secret \
      --from-literal=LDAP_CONFPASS=secret
    ```
*   PostgreSQL (=Archive DB), e.g.:

    ```
    microk8s kubectl create secret generic postgres-secret \
      --from-literal=POSTGRES_DB=pacsdb \
      --from-literal=POSTGRES_USER=pacs \
      --from-literal=POSTGRES_PASSWORD=pacs
    ```
*   MariaDB (=Keycloak DB), e.g.:

    ```
    microk8s kubectl create secret generic mariadb-secret \
      --from-literal=MYSQL_DATABASE=keycloak \
      --from-literal=MYSQL_USER=keycloak \
      --from-literal=MYSQL_PASSWORD=keycloak \
      --from-literal=MYSQL_ROOT_PASSWORD=secret
    ```
*   Keycloak, e.g.:

    ```
    microk8s kubectl create secret generic keycloak-secret \
      --from-literal=KEYCLOAK_ADMIN=admin \
      --from-literal=KEYCLOAK_ADMIN_PASSWORD=changeit \
      --from-literal=KIBANA_CLIENT_ID=kibana \
      --from-literal=KIBANA_CLIENT_SECRET=changeit
    ```

#### Deploy Elasticsearch

Ensure necessary [System Configuration for Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html) for the node running Elasticsearch. Particularly [disable swapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html) and [ensure sufficient virtual memory for Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) by setup required sysctl params (these persist across reboots):

```
cat <<EOF | sudo tee /etc/sysctl.d/99-elasticsearch.conf
vm.swappiness    = 1
vm.max_map_count = 262144
EOF
```

and apply sysctl params without reboot

```
sudo sysctl --system
```

Ensure that [Elasticsearch has write access to bind-mounted config, data and log dirs](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/docker.html#_configuration_files_must_be_readable_by_the_elasticsearch_user) by granting group access to gid 0 for the local directory.

```
$ sudo mkdir -p /var/local/microk8s/esdatadir
$ sudo chmod g+rwx /var/local/microk8s/esdatadir
$ sudo chgrp 0 /var/local/microk8s/esdatadir
```

Create [`elasticsearch-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/elasticsearch-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  selector:
    matchLabels:
      app: elasticsearch
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      subdomain: elasticsearch
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
      containers:
        - name: elasticsearch
          image: elasticsearch:8.4.2
          env:
            - name: ES_JAVA_OPTS
              value: -Xms1024m -Xmx1024m
            - name: http.cors.enabled
              value: "true"
            - name: http.cors.allow-origin
              value: /.*/
            - name: http.cors.allow-headers
              value: X-Requested-With,Content-Length,Content-Type,Authorization
            - name: discovery.type
              value: single-node
            - name: xpack.security.enabled
              value: "false"
          ports:
            - containerPort: 9200
            - containerPort: 9300
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
            - name: data
              mountPath: /usr/share/elasticsearch/data
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: data
          hostPath:
            path: /var/local/microk8s/esdatadir
            type: DirectoryOrCreate
```

adjusting

```yaml
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
```

to the hostname of the node running Elasticsearch and [Heap size settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-size-settings)

```yaml
            - name: ES_JAVA_OPTS
              value: -Xms1024m -Xmx1024m
```

according your needs and the amount of RAM available on your server and invoke:

```
microk8s kubectl apply -f elasticsearch-deployment.yaml
```

To expose Elasticsearch endpoints to other deployed applications - particular to Logstash - create [`elasticsearch-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/elasticsearch-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  selector:
    app: elasticsearch
  clusterIP: None
  ports:
    - name: http
      port: 9200
    - name: discovery
      port: 9300
```

and invoke:

```
microk8s kubectl apply -f elasticsearch-service.yaml
```

#### Deploy Logstash

To ensure that Logstash has write access to the file used to persist the fingerprint of the last audit message you have to mount the file or parent directory specified by environment variable [HASH\_FILE](https://github.com/dcm4che-dockerfiles/logstash-dcm4chee#hashtree_file) to a host directory on the node running Logstash to avoid to start a new hash tree on every re-creation of the container. The file (or parent directory) must be writable by the logstash user of the container (uid=1000). E.g., for mapping the file:

```
sudo mkdir -p /var/local/microk8s/logstash
sudo touch /var/local/microk8s/logstash/filter-hashtree
sudo chown 1000:1000 /var/local/microk8s/logstash/filter-hashtree
```

or for mapping the parent directory

```
sudo mkdir -p /var/local/microk8s/logstash
sudo chown 1000:1000 /var/local/microk8s/logstash  
```

Create [`logstash-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/logstash-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logstash
  labels:
    app: logstash
spec:
  selector:
    matchLabels:
      app: logstash
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: logstash
    spec:
      subdomain: logstash
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
      containers:
        - name: logstash
          image: dcm4che/logstash-dcm4chee:8.4.2-15
          ports:
            - containerPort: 12201
              protocol: UDP
            - containerPort: 8514
              protocol: UDP
            - containerPort: 8514
              protocol: TCP
            - containerPort: 6514
              protocol: TCP
          volumeMounts:
            - mountPath: /etc/localtime
              readOnly: true
              name: localtime
            - mountPath: /etc/timezone
              readOnly: true
              name: timezone
            - name: filter-hashtree
              mountPath: /usr/share/logstash/data/filter-hashtree
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: filter-hashtree
          hostPath:
            path: /var/local/microk8s/logstash/filter-hashtree
            type: File
```

adjusting

```yaml
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
```

to the hostname of the node running Elasticsearch.

For mapping the parent directory, replace

```yaml
          - name: filter-hashtree
            mountPath: /usr/share/logstash/data/filter-hashtree
```

by

```yaml
          - name: data
            mountPath: /usr/share/logstash/data
```

and

```yaml
        - name: filter-hashtree
          hostPath:
            path: /var/local/microk8s/logstash/filter-hashtree
            type: File
```

by

```yaml
        - name: data
          hostPath:
            path: /var/local/microk8s/logstash
            type: Directory
```

Invoke:

```
microk8s kubectl apply -f logstash-deployment.yaml
```

To expose Logstash endpoints to other deployed applications - particular to Keycloak and to the Archive - create [`logstash-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/logstash-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: logstash
  labels:
    app: logstash
spec:
  selector:
    app: logstash
  clusterIP: None
  ports:
    - name: gelf-udp
      port: 12201
      protocol: UDP
    - name: syslog-udp
      port: 8514
      protocol: UDP
    - name: syslog-tcp
      port: 8514
      protocol: TCP
    - name: syslog-tls
      port: 6514
      protocol: TCP
```

and invoke:

```
microk8s kubectl apply -f elasticsearch-service.yaml
```

#### Deploy Kibana

Create [`kibana-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/kibana-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  labels:
    app: kibana
spec:
  selector:
    matchLabels:
      app: kibana
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: kibana
    spec:
      subdomain: kibana
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
      containers:
        - name: kibana
          image: kibana:8.4.2
          ports:
            - containerPort: 5601
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
```

with

```yaml
      nodeSelector:
        kubernetes.io/hostname: <elk-node>
```

adjusted to the hostname of the node running Kibana and invoke:

```
microk8s kubectl apply -f kibana-deployment.yaml
```

To expose the Kibana endpoint to other deployed applications - particular to OAuth2 Proxy - create [`kibana-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/kibana-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kibana
  labels:
    app: kibana
spec:
  selector:
    app: kibana
  clusterIP: None
  ports:
    - name: http
      port: 3100
```

and invoke:

```
microk8s kubectl apply -f kibana-service.yaml
```

#### Deploy LDAP Server

Create [`ldap-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/ldap-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ldap
  labels:
    app: ldap
spec:
  selector:
    matchLabels:
      app: ldap
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: ldap
    spec:
      subdomain: ldap
      nodeSelector:
        kubernetes.io/hostname: <db-node>
      containers:
        - name: ldap
          image: dcm4che/slapd-dcm4chee:2.6.5-31.2
          env:
            - name: LDAP_URLS
              value: ldap:/// ldaps:///
            - name: STORAGE_DIR
              value: /storage/fs1
            - name: ARCHIVE_HOST
              value: <arc-node>
            - name: SYSLOG_HOST
              value: logstash
            - name: SYSLOG_PORT
              value: "8514"
            - name: SYSLOG_PROTOCOL
              value: TLS
          envFrom:
            - secretRef:
                name: ldap-secret
          ports:
            - containerPort: 389
            - containerPort: 636
          volumeMounts:
            - name: ldap-data
              mountPath: /var/lib/openldap/openldap-data
            - name: ldap-conf
              mountPath: /etc/openldap/slapd.d
      volumes:
        - name: ldap-data
          hostPath:
            path: /var/local/microk8s/ldap
            type: DirectoryOrCreate
        - name: ldap-conf
          hostPath:
            path: /var/local/microk8s/slapd.d
            type: DirectoryOrCreate
```

with

```yaml
      nodeSelector:
        kubernetes.io/hostname: <db-node>
```

adjusted to the hostname of the node running LDAP and `<arc-node>` and

```yaml
            - name: ARCHIVE_HOST
              value: <arc-node>
```

to the hostname of the/a node exposing the Archive endpoints and invoke:

```
microk8s kubectl apply -f ldap-deployment.yaml
```

To expose the LDAP endpoints to other deployed applications - particular to Keycloak and to the Archive - create [`ldap-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/ldap-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ldap
  labels:
    app: ldap
spec:
  clusterIP: None
  selector:
    app: ldap
  ports:
    - name: ldap
      port: 389
    - name: ldaps
      port: 636
```

and invoke:

```
microk8s kubectl apply -f ldap-service.yaml
```

#### Deploy PostgreSQL

Create [`db-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/db-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  labels:
    app: db
spec:
  selector:
    matchLabels:
      app: db
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: db
    spec:
      subdomain: db
      nodeSelector:
        kubernetes.io/hostname: `<db-node>`
      containers:
        - name: db
          image: dcm4che/postgres-dcm4chee:15.4-31
          envFrom:
            - secretRef:
                name: postgres-secret
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
            - name: data
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: data
          hostPath:
            path: /var/local/microk8s/db
            type: DirectoryOrCreate
```

adjusting

```yaml
      nodeSelector:
        kubernetes.io/hostname: <db-node>
```

to the hostname of the node running PostgreSQL and invoke:

```
microk8s kubectl apply -f db-deployment.yaml
```

To expose the PostgreSQL endpoint to other deployed applications - particular to the Archive - create [`db-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/db-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: db
spec:
  clusterIP: None
  selector:
    app: db
  ports:
    - port: 5432
```

and invoke:

```
microk8s kubectl apply -f db-service.yaml
```

#### Deploy MariaDB (used by Keycloak)

Create [`mariadb-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/mariadb-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  labels:
    app: mariadb
spec:
  selector:
    matchLabels:
      app: mariadb
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      subdomain: mariadb
      nodeSelector:
        kubernetes.io/hostname: <db-node>
      containers:
        - name: mariadb
          image: mariadb:10.7.3
          envFrom:
            - secretRef:
                name: mariadb-secret
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
            - name: data
              mountPath: /var/lib/mysql
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: data
          hostPath:
            path: /var/local/microk8s/mariadb
            type: DirectoryOrCreate
```

adjusting

```yaml
      nodeSelector:
        kubernetes.io/hostname: <db-node>
```

to the hostname of the node running MariaDB and invoke:

```
microk8s kubectl apply -f mariadb-deployment.yaml
```

To expose the MariaDB endpoint to other deployed applications - particular to Keycloak - create [`mariadb-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/mariadb-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  labels:
    app: mariadb
spec:
  clusterIP: None
  selector:
    app: mariadb
  ports:
    - port: 3306
```

and invoke:

```
microk8s kubectl apply -f mariadb-service.yaml
```

#### Deploy Keycloak

Create [`keycloak-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/keycloak-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  selector:
    matchLabels:
      app: keycloak
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
      containers:
        - name: keycloak
          image: dcm4che/keycloak:23.0.3
          env:
            - name: LDAP_URL
              value: "ldap://ldap:389"
            - name: ARCHIVE_HOST
              value: "<arc-node>"
            - name: KC_HOSTNAME
              value: "<arc-node>"
            - name: KC_HTTPS_PORT
              value: "8843"
            - name: KC_DB
              value: mariadb
            - name: KC_DB_URL_HOST
              value: mariadb
            - name: KC_DB_URL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mariadb-secret
                  key: MYSQL_DATABASE
            - name: KC_DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mariadb-secret
                  key: MYSQL_USER
            - name: KC_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mariadb-secret
                  key: MYSQL_PASSWORD
            - name: KC_SPI_LOGIN_PROTOCOL_OPENID_CONNECT_LEGACY_LOGOUT_REDIRECT_URI
              value: "true"
            - name: KC_LOG
              value: file
            - name: GELF_ENABLED
              value: "true"
            - name: LOGSTASH_HOST
              value: logstash
            - name: KIBANA_REDIRECT_URL
              value: "https://<arc-node>:8643/auth2/callback/*"
            - name: KEYCLOAK_WAIT_FOR
              value: ldap:636 mariadb:3306 logstash:8514
          envFrom:
            - secretRef:
                name: ldap-secret
            - secretRef:
                name: keycloak-secret
          ports:
            - containerPort: 8843
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
            - name: keycloak
              mountPath: /opt/keycloak/data
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: keycloak
          hostPath:
            path: /var/local/microk8s/keycloak
            type: DirectoryOrCreate
```

adjusting

```yaml
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
```

and

```yaml
      env:
        - name: KC_HOSTNAME
          value: "<arc-node>"
```

to the hostname of the node running Keycloak,

```yaml
      env:
        - name: ARCHIVE_HOST
          value: "<arc-node>"
```

to the hostname of the node running the archive and

```yaml
        - name: KIBANA_REDIRECT_URL
          value: "https://<arc-node>:8643/auth2/callback/*"
```

to the hostname of the node running OAuth2 Proxy for securing Kibana.

To connect to the LDAP server via TLS, replace

```yaml
            - name: LDAP_URL
              value: ldap://ldap:389
```

by

```yaml
            - name: LDAP_URL
              value: ldaps://ldap:636
```

before invoking:

```
microk8s kubectl apply -f keycloak-deployment.yaml
```

To expose the Keycloak endpoint to external Web Browsers and other deployed applications - particular to OAuth2 Proxy and to the Archive - create [`keycloak-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/keycloak-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: keycloak
  labels:
    app: keycloak
spec:
  selector:
    app: keycloak
  externalIPs:
    - <external-ip>
  ports:
    - name: https
      port: 8843
```

Adjust

```yaml
  externalIPs:
    - <external-ip>
```

to the IP address(es) of the node(s) on which the endpoints shall be exposed and invoke:

```
microk8s kubectl apply -f keycloak-service.yaml
```

**Verify OIDC client for OAuth2 Proxy in Keycloak**

Sign in with User/Password `root`/`changeit` at the Realm Admin Console of Keycloak at `https://<arc-node>:8843/dcm4che/console` - you have to replace `<arc-node>` by the hostname of the node exposing Keycloak endpoint(s). If you changed the default realm name: `dcm4che` by environment variable `REALM_NAME` for the Keycloak, the Keycloak Proxy and the Archive Container, you also have to replace `dcm4che` by the that value in the URL.

Keycloak docker image `dcm4che/keycloak:19.0.1` and newer creates an OIDC client for OAuth2 Proxy for securing Kibana on first startup, customizable by [environment variables](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#environment-variables) [`KIBANA_CLIENT_ID`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kibana_client_id), [`KIBANA_CLIENT_SECRET`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#elastic_client_secret) and [`KIBANA_REDIRECT_URL`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kibana_redirect_url):

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-kibana.png)

with Audience Token Mapper `audience`:

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-kibana-mapper.png)

and with the Client Credential:

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-kibana-secret.png)

as you specified on creating Opaque Secrets `keycloak-secret` above.

If you `Regenerate Secret` you have to update also the value in the Opaque Secrets `keycloak-secret`, which will get passed by environment variable `OAUTH2_PROXY_CLIENT_SECRET` to the OAuth2 Proxy container in the next step.

#### Deploy OAuth2 Proxy securing Kibana

Create [`kibana-oauth2proxy-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/kibana-oauth2-proxy-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana-oauth2proxy
  labels:
    app: kibana-oauth2proxy
spec:
  selector:
    matchLabels:
      app: kibana-oauth2proxy
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: kibana-oauth2proxy
    spec:
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
      containers:
        - name: kibana-oauth2proxy
          image: dcm4che/oauth2-proxy:7.5.1
          env:
            - name: OAUTH2_PROXY_CLIENT_ID
              valueFrom:
                secretKeyRef:
                  name: keycloak-secret
                  key: KIBANA_CLIENT_ID
            - name: OAUTH2_PROXY_CLIENT_SECRET
              valueFrom:
                secretKeyRef:
                  name: keycloak-secret
                  key: KIBANA_CLIENT_SECRET
          args:
            - --https-address=:8643
            - --tls-cert-file=/etc/certs/cert.pem
            - --tls-key-file=/etc/certs/key.pem
            - --ssl-insecure-skip-verify
            - --provider=keycloak-oidc
            - --oidc-issuer-url=https://<arc-node>:8843/realms/dcm4che
            - --redirect-url=https://<arc-node>:8643/oauth2/callback
            - --upstream=http://kibana:5601
            - --allowed-role=auditlog
            - --email-domain=*
            - --oidc-email-claim=sub
            - --insecure-oidc-allow-unverified-email
            - --cookie-secret=T0F1dGhLaWJhbmFUZXN0cw==
            - --skip-provider-button
            - --custom-templates-dir=/templates
          ports:
            - containerPort: 8643
```

Adjust

```yaml
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
```

and

```yaml
            - --redirect-url=https://<arc-node>:8643/auth2/callback
```

to the hostname of the node running OAuth2 Proxy.

Adjust

```yaml
            - --oidc-issuer-url=https://<arc-node>:8843/realms/dcm4che
```

to the hostname of the node running Keycloak.

Invoke:

```
microk8s kubectl apply -f kibana-oauth2proxy-deployment.yaml
```

To expose the OAuth2 Proxy endpoint to external Web Browsers create [`kibana-oauth2proxy-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/kibana-oauth2proxy-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kibana-oauth2proxy
  labels:
    app: kibana-oauth2proxy
spec:
  selector:
    app: kibana-oauth2proxy
  externalIPs:
    - <external-ip>
  ports:
    - name: https
      port: 8643
```

Adjust

```yaml
  externalIPs:
    - <external-ip>
```

to the IP address(es) of the node(s) on which the endpoint shall be exposed and invoke:

```
microk8s kubectl apply -f kibana-oauth2proxy-service.yaml
```

#### Deploy the Archive

Create [`arc-deployment.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/arc-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: arc
  labels:
    app: arc
spec:
  selector:
    matchLabels:
      app: arc
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: arc
    spec:
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
      containers:
        - name: arc
          image: dcm4che/dcm4chee-arc-psql:5.27.0-secure
          env:
            - name: HTTP_PORT
              value: "8080"
            - name: HTTPS_PORT
              value: "8443"
            - name: MANAGEMENT_HTTP_PORT
              value: "9990"
            - name: MANAGEMENT_HTTPS_PORT
              value: "9993"
            - name: LDAP_URL
              value: ldap://ldap:389
            - name: AUTH_SERVER_URL
              value: https://<arc-node>:8843
            - name: WILDFLY_CHOWN
              value: /opt/wildfly/standalone /storage
            - name: WILDFLY_WAIT_FOR
              value: ldap:389 db:5432 logstash:8514
            - name: JAVA_OPTS
              value: -Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
            - name: POSTGRES_HOST
              value: db
            - name: LOGSTASH_HOST
              value: logstash
          envFrom:
            - secretRef:
                name: ldap-secret
            - secretRef:
                name: postgres-secret
          ports:
            - containerPort: 8080
            - containerPort: 8443
            - containerPort: 9990
            - containerPort: 9993
            - containerPort: 8787
            - containerPort: 11112
            - containerPort: 2762
            - containerPort: 2575
            - containerPort: 12575
          volumeMounts:
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: timezone
              mountPath: /etc/timezone
              readOnly: true
            - name: wildfly
              mountPath: /opt/wildfly/standalone
            - name: storage
              mountPath: /storage
      volumes:
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: timezone
          hostPath:
            path: /etc/timezone
            type: File
        - name: wildfly
          hostPath:
            path: /var/local/microk8s/wildfly
            type: DirectoryOrCreate
        - name: storage
          hostPath:
            path: /var/local/microk8s/storage
            type: DirectoryOrCreate
```

Adjust

```yaml
      nodeSelector:
        kubernetes.io/hostname: <arc-node>
```

to the hostname of the node running the archive.

Adjust

```yaml
            - name: AUTH_SERVER_URL
              value: https://<arc-node>:8843
```

to the hostname of the node running Keycloak.

To connect to the LDAP server via TLS, replace

```yaml
            - name: LDAP_URL
              value: ldap://ldap:389
```

by

```yaml
            - name: LDAP_URL
              value: ldaps://ldap:636
```

Adjust

```yaml
            - name: JAVA_OPTS
              value: -Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -agentlib:jdwp=transport=dt_socket,address=*:8787,server=y,suspend=n
```

according your needs and the amount of RAM available on your server.

Invoke:

```
microk8s kubectl apply -f arc-deployment.yaml
```

To expose the Archive endpoints to external applications create [`arc-service.yaml`](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/k8s/arc-service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: arc
  labels:
    app: arc
spec:
  selector:
    app: arc
  externalIPs:
    - <external-ip>
  ports:
    - name: http
      port: 8080
    - name: https
      port: 8443
    - name: management-http
      port: 9990
    - name: management-https
      port: 9993
    - name: dicom
      port: 11112
    - name: dicom-tls
      port: 2762
    - name: mllp
      port: 2575
    - name: mllp-tls
      port: 12575
```

Adjust

```yaml
  externalIPs:
    - <external-ip>
```

to the IP address(es) of the node(s) on which the endpoints shall be exposed and invoke:

```
microk8s kubectl apply -f arc-service.yaml
```

#### Verify OIDC client for Archive UI in Keycloak

Sign in with User/Password `root`/`changeit` at the Realm Admin Console of Keycloak at `https://test-ng:8843/admin/dcm4che/console` - you have to replace `test-ng` by the hostname of the node exposing the Keycloak endpoint(s). If you changed the default realm name: `dcm4che` by environment variable `REALM_NAME` for the Keycloak and the Archive Container, you also have to replace `dcm4che` by the that value in the URL.

Keycloak docker image `dcm4che/keycloak:19.0.1` and newer creates an OIDC client for the Archive UI on first startup, customizable by [environment variables](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#environment-variables) [`UI_CLIENT_ID`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#ui_client_id), [`ARCHIVE_HOST`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_host), [`ARCHIVE_HTTP_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_http_port) and [`ARCHIVE_HTTPS_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_https_port):

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-dcm4chee-arc-ui.png)

#### Verify OIDC client for Wildfly Administration Console in Keycloak

Access to the [Wildfly Administration Console is also protected with Keycloak](https://docs.jboss.org/author/display/WFLY/Protecting%20Wildfly%20Adminstration%20Console%20With%20Keycloak.html).

Keycloak docker image `dcm4che/keycloak:19.0.1` and newer creates also another OIDC client for the Wildfly Administration Console on first startup, customizable by [environment variables](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#environment-variables) [`WILDFLY_CONSOLE`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#wildfly_console), [`ARCHIVE_HOST`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_host) and [`ARCHIVE_MANAGEMENT_HTTPS_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_management_https_port):

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/keycloak-client-wildfly-console.png)

Only users with role `ADMINISTRATOR` are permitted to access the WildFly Administration Console.

Sign out, before verifying that accessing the WildFly Administration Console at `http://<arc-node>:9990` or `https://<arc-node:9993` will redirect you to the Log in page of Keycloak. You may sign in with User/Password `root`/`changeit`.

#### Verify status of deployments and services

*   List all Deployments

    ```
    microk8s kubectl get deployments -o wide
    ```

    ```
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS          IMAGES                                    SELECTOR
    elasticsearch       1/1     1            1           133m    elasticsearch       elasticsearch:8.4.2                       app=elasticsearch
    logstash            1/1     1            1           130m    logstash            dcm4che/logstash-dcm4chee:8.4.2-15        app=logstash
    kibana              1/1     1            1           129m    kibana              kibana:8.4.2                              app=kibana
    ldap                1/1     1            1           127m    ldap                dcm4che/slapd-dcm4chee:2.6.5-31.2         app=ldap
    db                  1/1     1            1           126m    db                  dcm4che/postgres-dcm4chee:15.4-31         app=db
    keycloak            1/1     1            1           71m     keycloak            dcm4che/keycloak:23.0.3                   app=keycloak
    mariadb             1/1     1            1           54m     mariadb             mariadb:10.7.3                            app=mariadb
    kibana-oauth2proxy  1/1     1            1           9m56s   kibana-oauth2proxy  dcm4che/oauth2-proxy:7.5.1                app=kibana-oauth2proxy
    arc                 1/1     1            1           6m25s   arc                 dcm4che/dcm4chee-arc-psql:5.27.0-secure   app=arc
    ```
*   List all deployed Pods

    ```
    microk8s kubectl get pods -o wide
    ```

    ```
    NAME                               READY   STATUS    RESTARTS      AGE     IP             NODE      NOMINATED NODE   READINESS GATES
    mariadb-79cf85cf4b-jxr44           1/1     Running   0             56m     10.1.180.203   test-ng   <none>           <none>
    keycloak-dbc99f676-pdhfz           1/1     Running   0             73m     10.1.180.202   test-ng   <none>           <none>
    db-77c9898c5c-7vwrn                1/1     Running   0             128m    10.1.180.201   test-ng   <none>           <none>
    elasticsearch-864965cf6d-4g9w2     1/1     Running   0             135m    10.1.180.197   test-ng   <none>           <none>
    kibana-7ffbb4d769-hqb9w            1/1     Running   0             131m    10.1.180.199   test-ng   <none>           <none>
    logstash-6bdc9b5c9c-znqm4          1/1     Running   0             132m    10.1.180.198   test-ng   <none>           <none>
    ldap-55444979cb-cmrl9              1/1     Running   0             129m    10.1.180.200   test-ng   <none>           <none>
    kibana-oauth2proxy-77546fbb-gh9ds  1/1     Running   0             12m     10.1.180.204   test-ng   <none>           <none>
    arc-845fdc847-b28qk                1/1     Running   0             8m34s   10.1.180.205   test-ng   <none>           <none>
    ```
*   List all Services

    ```
    microk8s kubectl get services -o wide
    ```

    ```
    NAME                TYPE        CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                     AGE     SELECTOR
    kubernetes          ClusterIP   10.152.183.1     <none>          443/TCP                                                                     158m    <none>
    elasticsearch       ClusterIP   None             <none>          9200/TCP,9300/TCP                                                           137m    app=elasticsearch
    kibana              ClusterIP   None             <none>          3100/TCP                                                                    133m    app=kibana
    ldap                ClusterIP   None             <none>          389/TCP,636/TCP                                                             131m    app=ldap
    db                  ClusterIP   None             <none>          5432/TCP                                                                    130m    app=db
    mariadb             ClusterIP   None             <none>          3306/TCP                                                                    128m    app=mariadb
    logstash            ClusterIP   None             <none>          12201/UDP,8514/UDP,8514/TCP,6514/TCP                                        57m     app=logstash
    keycloak            ClusterIP   10.152.183.115   192.168.2.145   8843/TCP                                                                    45m     app=keycloak
    kibana-oauth2proxy  ClusterIP   10.152.183.5     192.168.2.145   8643/TCP                                                                    13m     app=kibana-oauth2proxy
    arc                 ClusterIP   10.152.183.238   192.168.2.145   8080/TCP,8443/TCP,9990/TCP,9993/TCP,11112/TCP,2762/TCP,2575/TCP,12575/TCP   9m48s   app=arc
    ```
*   Alternatively deploy Kubernetes Dashbord

    ```
    microk8s dashboard-proxy
    ```

    ```
    Checking if Dashboard is running.
    Warning: apiregistration.k8s.io/v1beta1 APIService is deprecated in v1.19+, unavailable in v1.22+; use apiregistration.k8s.io/v1 APIService
    Waiting for Dashboard to come up.
    Dashboard will be available at https://127.0.0.1:10443
    Use the following token to login:
    eyJhbGciOiJSUzI1NiIsImtpZCI6InU0OTZpLUJrbFRCcmdBWnFzTG5Gb05vY09SY1RWNWQtTjd5cjdkdnZsTUUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLXdxcnM4Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4ZjMwZWU2OC02YTkyLTQ2Y2ItYTk2ZS0xMWM1MDUyNTg4NDQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.T_L24S4eVdiavBTyExalCCydyBp2S4P3yune50R0QWZDLHscX0GGR-Tqn0OH1kanvyuoDlRptOuqQ6Um9BNP2sImxep6SiArCDzTUnhmPnO0ke3AiAmzvIqDI6SxKbGckfknVfzsQkRNwERtGOM1pW-bEH-XL2d0DVzDdS9SFqbbGLnzVrsvuh7JWnOzCS2MPJ10_bUfh1lR5OunOIl1Pu7uS_PGNDEsZ6T-6sue6JWcBnzhmw-e1qzV5u2Iuw8B7X3hlMAZV89PU7L6xf4XF9wwHWocc__epwWTNnN4h2WxmTz51IUy0TVcA4ii7yovuV93YSEg4_CVP1qbkBjUrQ
    ```

    and access the Dashboard URL with any Web browser:

    ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/kubernetes-dashboard.png)
