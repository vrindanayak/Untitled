# Overview

Distributed environments frequently require the use of a reverse proxy.

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/uml/reverse-proxy.svg)

### Configure Nginx passthrough reverse proxy for HTTPs, DICOM and HL7 connections

According [Nginx Admin Guide, Configuring Reverse Proxy](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/#configuring-reverse-proxy) add a top‑level `stream {}` block in the Nginx configuration file `/etc/nginx/nginx.conf`, with a `server {}` configuration block for each TCP connection which shall be forwarded to Keycloak, the Archive or the OAuth2 Proxy, including the `listen` directive to define the port on the Proxy Node, and the `proxy_pass` directive to define host and port of the proxied service. E.g.:

```
# Archive DICOM
stream {
  listen     11112;
  proxy_pass arc-node:11112
}

# Archive DICOM-TLS
stream {
  listen     2762;
  proxy_pass arc-node:2762
}

# Archive HL7
stream {
  listen     2575;
  proxy_pass arc-node:2575
}

# Archive HL7-TLS
stream {
  listen     12575;
  proxy_pass arc-node:12575
}

# Archive UI HTTPs
stream {
  listen     9443;
  proxy_pass arc-node:8443
}

# Archive Wildfly Adminstration Console HTTPs
stream {
  listen     9993;
  proxy_pass arc-node:9993
}

# Keycloak HTTPs
stream {
  listen     9843;
  proxy_pass arc-node:8843
}

# OAuth2 Proxy HTTPs
stream {
  listen     9643;
  proxy_pass arc-node:8643
}
```

### Adjust Keycloak server configuration

Specify proxy mode as `passthrough` by commandline option [`--proxy`](https://www.keycloak.org/server/reverseproxy#_configure_the_proxy_mode_in_keycloak) or environment variable [`KC_PROXY`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kc_proxy).

Adjust configured frontend endpoint by commandline options [`--hostname` and `--hostname-port`](https://www.keycloak.org/server/hostname) or environment variables [`KC_HOSTNAME`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kc_hostname) and [`KC_HOSTNAME_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kc_hostname_port) to the hostname of the proxy and the port on the proxy configured to forward requests to Keycloak.

### Adjust _Valid Redirect URIs_ and _Web Origins_ of configured Keycloak OIDC clients

Add/Change _Valid Redirect URI_ and _Web Origins_ of configured Keycloak OIDC client `dcm4chee-arc-ui` for the Archive UI reflecting the hostname of the proxy and the port on the proxy configured to forward requests to the Archive HTTPs port. Or adjust the environment variables [`ARCHIVE_HOST`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_host) and [`ARCHIVE_HTTPS_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_https_port) accordingly **before** the first start of the keycloak container.

Add/Change _Valid Redirect URI_ and _Web Origins_ of configured Keycloak OIDC client `wildfly-console` for the Archive Wildfly Adminstration Console reflecting the hostname of the proxy and the port on the proxy configured to forward requests to the Archive Wildfly Management HTTPs port. Or adjust the environment variables [`ARCHIVE_HOST`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_host) and [`ARCHIVE_MANAGEMENT_HTTPS_PORT`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#archive_management_https_port) accordingly **before** the first start of the keycloak container.

Add/Change _Valid Redirect URI_ of configured Keycloak OIDC client `kibana` for Kibana reflecting the hostname of the proxy and the port on the proxy configured to forward requests to the OAuth2 Proxy in front of Kibana. Or adjust the environment variables [`KIBANA_REDIRECT_URL`](https://github.com/dcm4che-dockerfiles/keycloak-quarkus#kibana_redirect_url) accordingly **before** the first start of the keycloak container.

### Configure Keycloak Frontend URL for accessing Keycloak by the Archive UI from Web Browsers

Configure the Keycloak Frontend URL reflecting the hostname of the proxy and the port on the proxy configured to forward requests to Keycloak by environment variable [`UI_AUTH_SERVER_URL`](https://github.com/dcm4che-dockerfiles/dcm4chee-arc-psql#ui_auth_server_url) of the archive container.

### Adjust configured OAuth Redirect URL of the OAuth2 Proxy

Adjust configured OAuth Redirect URL (option `redirect-url` or environment variable `OAUTH2_PROXY_REDIRECT_URL`) of the OAuth2 Proxy reflecting the hostname of the proxy and the port on the proxy configured to forward requests to OAuth2 Proxy.
