# Installation

For older releases go [here](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Installation-older-releases)

**Latest released archive version is 5.31.2**

## Install DCM4CHEE Archive 5.x

Content

* Requirements
* Download and extract binary distribution package
* Initialize Database
  * H2
  * PostgreSQL
  * MySQL and MariaDB
  * Firebird
  * DB2
  * Oracle
  * MS SQL Server
* Setup LDAP Server
  * OpenLDAP
    * OpenLDAP with slapd.conf configuration file
    * OpenLDAP with dynamic runtime configuration
  * OpenDJ
  * Apache DS 2.0.0
* Import default configuration into LDAP Server
* Setup WildFly

### Requirements

* Java SE 11 or later - tested with [OpenJDK](http://openjdk.java.net)
* [Wildfly 29.0.1](https://github.com/wildfly/wildfly/releases/download/29.0.1.Final/wildfly-29.0.1.Final.zip)
* Supported SQL Database:
  * [H2](http://www.h2database.com) embedded in Wildfly (Not for production use!)
  * [PostgreSQL 14.3](http://www.postgresql.org/download)
  * [MariaDB 10.6.8](https://downloads.mariadb.org)
  * [MySQL 8.0.29](http://dev.mysql.com/downloads/mysql)
  * [Firebird 3.0.10](http://www.firebirdsql.org/en/firebird-3-0-10/)
  * [Oracle 11g](http://www.oracle.com/technetwork/products/express-edition/downloads)
  * [DB2 10.5](http://www-01.ibm.com/software/data/db2/express/download.html)
  * [MS SQL Server](https://go.microsoft.com/fwlink/?linkid=866662) (not yet tested!)
* Supported LDAP Servers:
  * [OpenLDAP 2.5.11](http://www.openldap.org/software/download)
  * [OpenDJ 2.6.0](https://backstage.forgerock.com/#/downloads/enterprise/OpenDJ)
  * [Apache DS 2.0.0-M20](http://directory.apache.org/apacheds/downloads.html)
* LDAP Browser: [Apache Directory Studio 2.0.0-M17](https://tracker.iplocation.net/ivwz/)

### Download and extract binary distribution package

DCM4CHEE Archive 5.x binary distributions for different databases can be obtained from [Sourceforge](https://tracker.iplocation.net/ivxb/). Extract (unzip) release package version as required.

Secured versions are provided for PostgreSQL, MySQL, SQLServer and Oracle.

### Initialize Database

_Note_: DCM4CHEE Archive 5.x does not provide SQL scripts and utilities to migrate database schema from DCM4CHEE Archive 2.x to DCM4CHEE Archive 5.x.

#### H2

There is no need to initialize the database before starting Wildfly. But create tables and indexes using H2 Console before deploying the archive.

#### PostgreSQL

1.  Create user with permission to create databases:

    ```
     > sudo -u postgres createuser -U postgres -P -d <user-name>
     Enter password for new role: <user-password>
     Enter it again: <user-password>
    ```
2.  Create database:

    ```
     > createdb -h localhost -U <user-name> <database-name>
    ```
3.  Create tables and indexes:

    ```
     > psql -h localhost <database-name> <user-name> < $DCM4CHEE_ARC/sql/psql/create-psql.sql
     > psql -h localhost <database-name> <user-name> < $DCM4CHEE_ARC/sql/psql/create-fk-index.sql
     > psql -h localhost <database-name> <user-name> < $DCM4CHEE_ARC/sql/psql/create-case-insensitive-index.sql
    ```

#### MySQL and MariaDB

1.  create database user

    ```
     mysql> CREATE USER 'USERNAME'@localhost IDENTIFIED BY 'PASSWORD';
    ```
2.  Create database and grant access to user::

    ```
     > mysql -u root -p<root-password>
     mysql> create database <database-name>;
     mysql> grant all on <database-name>.* to '<user-name>' identified by '<user-password>';
     mysql> quit
    ```
3.  Create tables and indexes::

    ```
     > mysql -u <user-name> -p<user-password> <database-name> < $DCM4CHEE_ARC/sql/mysql/create-mysql.sql
    ```

#### Firebird

1.  Create user

    ```
     > gsec -user sysdba -password <sysdba-password>
     GSEC> add <user-name> -pw <user-password>
     GSEC> quit
    ```
2.  Define database name in configuration file `aliases.conf`:

    ```
     <database-name> = <database-file-path>
    ```
3.  Create database, tables and indexes

    ```
     > isql-fb 
     Use CONNECT or CREATE DATABASE to specify a database
     SQL> create database 'localhost:<database-name>' pagesize 16384
     CON> user '<user-name>' password '<user-password>';
     SQL> in DCM4CHEE_ARC/sql/firebird/create-firebird.sql;
     SQL> exit;
    ```

#### DB2

1.  Create database and grant authority to create tables to user (must match existing OS user)

    ```
     > sudo su db2inst1
     > db2
     db2 => create database <database-name> pagesize 16 K
     db2 => connect to <database-name>
     db2 => grant createtab on database to user <user-name>
     db2 => terminate
    ```
2.  Create tables and indexes

    ```
     > su <user-name>
     Password: <user-password>
     > db2 connect to <database-name>
     > db2 -t < $DCM4CHEE_ARC/sql/db2/create-db2.sql
     > db2 -t < $DCM4CHEE_ARC/sql/db2/create-fk-index.sql
     > db2 terminate
    ```

#### Oracle

1.  Connect to Oracle and create a new tablespace

    ```
     $ sqlplus / as sysdba
     SQL> create bigfile tablespace <tablespace-name> datafile '<data-file-location>' size <size>;

     Tablespace created.
    ```
2.  Create a new user with privileges for the new tablespace

    ```
     $ sqlplus / as sysdba
     SQL> create user <user-name>
     2  identified by <user-password>
     3  default tabelspace <tablespace-name>
     4  quota unlimited on <tablespace-name>
     5  quota 50M on system;

     User created.

     SQL> grant create session to <user-name>;
     SQL> grant create table to <user-name>;
     SQL> grant create any index to <user-name>;
     SQL> grant create sequence to <user-name>;
     SQL> exit
    ```
3.  Create tables and indexes

    ```
     $ sqlplus <user-name>/<user-password>
     SQL> @$DCM4CHEE_ARC/sql/oracle/create-oracle.sql
     SQL> @$DCM4CHEE_ARC/sql/oracle/create-fk-index.sql
     SQL> @$DCM4CHEE_ARC/sql/oracle/create-case-insensitive-index.sql
    ```

#### MS SQL Server

_Not yet tested_

### Setup LDAP Server

#### OpenLDAP

OpenLDAP binary distributions are available for most Linux distributions as well as for [Windows](http://www.userbooster.de/en/download/openldap-for-windows.aspx).

Alternatively, configure OpenLDAP by

* [slapd.conf configuration file](http://www.openldap.org/doc/admin24/slapdconfig.html)
* [dynamic runtime configuration](http://www.openldap.org/doc/admin24/slapdconf2.html)

See also [Converting old style slapd.conf file to cn=config format](http://www.openldap.org/doc/admin24/slapdconf2.html#Converting%20old%20style%20\{{slapd.conf\}}%285%29%20file%20to%20\{{cn=config\}}%20format)

**OpenLDAP with slapd.conf configuration file**

1.  Copy LDAP schema files for OpenLDAP from DCM4CHEE Archive distribution to OpenLDAP schema configuration directory:

    ```
    > cp $DCM4CHEE_ARC/ldap/schema/* /etc/openldap/schema/ [UNIX]
    > copy %DCM4CHEE_ARC%\ldap\schema\* \Program Files\OpenLDAP\schema\ [Windows]
    ```
2.  Add references to schema files in `slapd.conf`, e.g.:

    ```
    include         /etc/openldap/schema/core.schema
    include         /etc/openldap/schema/dicom.schema
    include         /etc/openldap/schema/dcm4che.schema
    include         /etc/openldap/schema/dcm4chee-archive.schema
    include         /etc/openldap/schema/dcm4chee-archive-ui.schema
    ```
3.  Directory Base DN and Root User DN are specified in `slapd.conf` by

    ```
    suffix          "dc=nodomain"
    rootdn          "cn=admin,dc=nodomain"
    rootpw          secret
    ```

    and may be modified to

    ```
    suffix          "dc=dcm4che,dc=org"
    rootdn          "cn=admin,dc=dcm4che,dc=org"
    rootpw          secret
    ```

**OpenLDAP with dynamic runtime configuration**

1.  Import LDAP schema files for OpenLDAP runtime configuration, using OpenLDAP CL utility `ldapadd`, e.g.:

    ```
     > sudo ldapadd -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dicom.ldif
     > sudo ldapadd -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dcm4che.ldif
     > tr -d \\r < $DCM4CHEE_ARC/ldap/slapd/dcm4chee-archive.ldif | sudo ldapadd -Y EXTERNAL -H ldapi:///
     > sudo ldapadd -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dcm4chee-archive-ui.ldif
    ```

    The conversion of line separators from `\r` to  for importing `$DCM4CHEE/ldap/slapd/dcm4chee-archive.ldif` workarounds a [flaw in ldapadd distributed by Ubuntu](https://github.com/dcm4che/dcm4chee-arc-light/issues/2568) concerning large content records.

    For use against docker images, use basic authentication and the password (secret by default), e.g.:

    ```
      > ldapadd -xw secret -H ldap://localhost -D "cn=admin,dc=dcm4che,dc=org" -f $DCM4CHEE_ARC/ldap/slapd/dicom.ldif
    ```
2.  Directory Base DN and Root User DN can be modified by changing the values of attributes

    ```
     olcSuffix: dc=nodomain
     olcRootDN: cn=admin,dc=nodomain
     olcRootPW
    ```

    of object `olcDatabase={1}hdb,cn=config` by specifying the new values in a LDIF file (e.g. `modify-baseDN.ldif`)

    ```
     dn: olcDatabase={1}hdb,cn=config
     changetype: modify
     replace: olcSuffix
     olcSuffix: dc=dcm4che,dc=org
     -
     replace: olcRootDN
     olcRootDN: cn=admin,dc=dcm4che,dc=org
     -
     replace: olcRootPW
     olcRootPW: <{SSHA}xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx>
     -
    ```

    **Note:** In order to execute the commands on CentOS7 use _dn: olcDatabase={2}hdb,cn=config_ instead of _dn: olcDatabase={1}hdb,cn=config_.

    **Note:** Replace _<{SSHA}xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx>_ by the hash generated by the command :

    ```
     > sudo slappasswd
    ```

    and applying it using OpenLDAP CL utility ldapmodify, e.g.:

    ```
     > sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f modify-baseDN.ldif
    ```

    After that you have to create the top element of the directory tree (dc=dcm4che,dc=org) and the object for the LDAP administrator account (cn=admin,dc=dcm4che,dc=org).

    LDIF file `slapd_setup_basic.ldif`:

    ```
     dn: dc=dcm4che,dc=org
     changetype: add
     objectClass: top
     objectClass: dcObject
     objectClass: organization
     o: Example Organisation name
     dc: dcm4che

     dn: cn=admin,dc=dcm4che,dc=org
     changetype: add
     objectClass: simpleSecurityObject
     objectClass: organizationalRole
     cn: admin
     description: LDAP administrator
     userPassword:
    ```

    To create these objects, you have to use the databases root account and to enter your LDAP admin password as specified during installation

    ```
     > ldapmodify -x -W -D cn=admin,dc=dcm4che,dc=org -H ldapi:/// -f slapd_setup_basic.ldif
    ```

#### OpenDJ

1.  Copy LDAP schema files for OpenDJ from DCM4CHEE Archive distribution to OpenDJ schema configuration directory:

    ```
     > cp $DCM4CHEE_ARC/ldap/opendj/* $OPENDJ_HOME/config/schema/ [UNIX]
     > copy %DCM4CHEE_ARC%\ldap\opendj\* %OPENDJ_HOME%\config\schema\ [Windows]
    ```
2.  Run OpenDJ GUI based setup utility

    ```
     > $OPENDJ_HOME/setup
    ```

    Log the values chosen for

    * LDAP Listener port (1389)
    * Root User DN (cn=Directory Manager)
    * Root User Password (secret)
    * Directory Base DN (dc=dcm4che,dc=org)

    needed for the LDAP connection configuration of DCM4CHEE Archive.
3.  After initial setup, one may start and stop OpenDJ by

    ```
     > $OPENDJ_HOME/bin/start-ds
     > $OPENDJ_HOME/bin/stopt-ds
    ```

#### Apache DS 2.0.0

1. Install [Apache DS 2.0.0-M20](http://directory.apache.org/apacheds/downloads.html) on the system and start Apache DS.
2.  Install [Apache Directory Studio 2.0.0-M9](http://directory.apache.org/studio/downloads.html) and create a new LDAP Connection with:

    ```
     Network Parameter:
         Hostname: localhost
         Port:     10389
     Authentication Parameter:
         Bind DN or user: uid=admin,ou=system
         Bind password:   secret
    ```
3.  Import LDAP schema files for Apache DS:

    ```
     $DCM4CHEE_ARC/ldap/apacheds/dicom.ldif
     $DCM4CHEE_ARC/ldap/apacheds/dcm4che.ldif
     $DCM4CHEE_ARC/ldap/apacheds/dcm4chee-archive.ldif
     $DCM4CHEE_ARC/ldap/apacheds/dcm4chee-archive-ui.ldif
    ```

    using the LDIF import function of Apache Directory Studio LDAP Browser.
4.  One may modify the default Directory Base DN `dc=example,dc=com` by changing the value of attribute

    ```
     ads-partitionsuffix: dc=dcm4che,dc=org`
    ```

    of object

    ```
     ou=config
     + ads-directoryServiceId=default
       + ou=partitions
           ads-partitionId=example
     
     to ads-partitionId=dcm4che
    ```

    using Apache Directory Studio LDAP Browser.

Note : Restart the Apache DS service to effect the above changes.

Alternatively, run ApacheDS using docker

### Import default configuration into LDAP Server

1.  If not already done, install [Apache Directory Studio 2.0.0-M9](http://directory.apache.org/studio/downloads.html) and create a new LDAP Connection corresponding to your LDAP Server configuration, e.g:

    ```
    Network Parameter:
        Hostname: localhost
        Port:     389
    Authentication Parameter:
        Bind DN or user: cn=admin,dc=dcm4che,dc=org
        Bind password:   secret
    Browser Options:
        Base DN: dc=dcm4che,dc=org
    ```
2.  If configured Directory Base DN is other than`dc=dcm4che,dc=org`, replace all occurrences of `dc=dcm4che,dc=org` in LDIF files

    ```
    $DCM4CHEE_ARC/ldap/init-baseDN.ldif
    $DCM4CHEE_ARC/ldap/init-config.ldif
    $DCM4CHEE_ARC/ldap/default-config.ldif
    $DCM4CHEE_ARC/ldap/default-ui-config.ldif
    $DCM4CHEE_ARC/ldap/add-vendor-data.ldif
    ```

    by your Directory Base DN, e.g.:

    ```
    > cd $DCM4CHEE_ARC/ldap
    > sed -i s/dc=dcm4che,dc=org/dc=my-domain,dc=com/ init-baseDN.ldif
    > sed -i s/dc=dcm4che,dc=org/dc=my-domain,dc=com/ init-config.ldif
    > sed -i s/dc=dcm4che,dc=org/dc=my-domain,dc=com/ default-config.ldif
    > sed -i s/dc=dcm4che,dc=org/dc=my-domain,dc=com/ default-ui-config.ldif
    > sed -i s/dc=dcm4che,dc=org/dc=my-domain,dc=com/ add-vendor-data.ldif
    ```
3. If there is no base entry in the directory database, import `$DCM4CHEE_ARC/ldap/init-baseDN.ldif` using the LDIF import function of Apache Directory Studio LDAP Browser.
4. If DICOM configuration root entries are not present in the directory database, import `$DCM4CHEE_ARC/ldap/init-config.ldif` using the LDIF import function of Apache Directory Studio LDAP Browser.
5.  Import

    * `$DCM4CHEE_ARC/ldap/default-config.ldif`
    * `$DCM4CHEE_ARC/ldap/default-ui-config.ldif`

    using the LDIF import function of Apache Directory Studio LDAP Browser.
6.  a. Using [Apache Directory Studio](https://directory.apache.org/studio/) : On dcm4chee-arc device level, add new attribute _dicomVendorData_ using the New Attribute function (or Ctrl+Shift++) and in its value, load the data by selecting `$DCM4CHEE_ARC/ldap/vendor-data.zip` file. This zip file contains XSLT stylesheets required for\
    attribute coercions of incoming or outgoing DICOM messages and mapping of HL7 fields in incoming / outgoing HL7 messages.

    b. Using command line : Import vendor data, directly using `$DCM4CHEE_ARC/ldap/add-vendor-data.ldif` script.

    ```
    > cd $DCM4CHEE_ARC/ldap
    > ldapadd -xwsecret -Dcn=admin,dc=dcm4che,dc=org -f add-vendor-data.ldif 
    modifying entry "dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org"
    ```

    Verify that vendor-data is imported in your LDAP using

    ```
    > ldapsearch -LLLsbase -xwsecret -Dcn=admin,dc=dcm4che,dc=org -b "dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org" dicomVendorData | head
    dn: dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,
    dc=org
    dicomVendorData:: UEsDBAoAAAgAAFRIslIAAAAAAAAAAAAAAAALAAAAdGh1bWJuYWlscy9QSwME
    FAAACAgANWxIUhgB/i3nDgAADGQAAAwAAABkc3IyaHRtbC54c2ztHWlv28j1e4H+hykjhFZXNiUf8
    SWpVS1510Blu7Y2u8AiG1DkyGJLkQw58tGi/70zPIdzkEPJ8aZFgyS2Rc6bd19zuP8cuWcRenFhtI
    QQgUcYRo7vDbTeXlcDzyvXi87wKwNtiVBwZhhPT097Twd7fvhg9E5PT42f7/9qzELTixZ+uNKGv/8
    dAH0C0l+jYI3ACqKlb5PRK1cD0LN82/EeBtqPs8vdEw0YxYDADM0V8MwVHGhPpu3vrkNXG6azur5l
    uks/Qmcn3ZOuYVurQwuju2uGlmFCFBnji+nhxQ+TiUGG9o0cYAH/0Qwdc+7CdIrID+7gItJABF1oo
    YFmXJvIeYRjx/JXU9+GrhF/O0IodOZrBH/5MzIfBnq3e9gdHRwf6cAPAf3RyZH+ybhCcEVm5P7IgH
    VPer1eNrLqpdPT9CWN8CynyvEsd21DsAzhYqBZa8feJ/Tt4WcJd/M38dDANREEKxNZSwG9ifDw20R
    ```
7. As required by your application setup needs,
   *   Change the default AE Title(s) of the Archive:

       ```
       AS_RECEIVED
       DCM4CHEE 
       IOCM_REGULAR_USE
       IOCM_EXPIRED
       IOCM_QUALITY
       IOCM_PAT_SAFETY
       IOCM_WRONG_MWL
       ```
   *   If one or more of the above default application entities are changed, ensure to also update the associated `AE Title` and AET value in `Web Service Path` in the corresponding \[\[Web Application]]

       ```
       AS_RECEIVED
       AS_RECEIVED-WADO
       DCM4CHEE
       DCM4CHEE-WADO
       IOCM_REGULAR_USE
       IOCM_REGULAR_USE-WADO
       IOCM_EXPIRED
       IOCM_EXPIRED-WADO
       IOCM_QUALITY
       IOCM_QUALITY-WADO
       IOCM_PAT_SAFETY
       IOCM_PAT_SAFETY-WADO
       IOCM_WRONG_MWL
       IOCM_WRONG_MWL-WADO
       ```
   * Change the default Storage Directory
   * Configure AE Title(s) of external DICOM Applications to which the Archive shall be able to connect to when acting as a [Service Class User](http://dicom.nema.org/medical/dicom/current/output/html/part04.html#glossentry_ServiceClassUser).
   * Configure [external HL7 receivers](https://github.com/dcm4che/dcm4chee-arc-light/wiki/HL7-Receiver) to which archive shall send HL7 messages / NOTIFICATIONS.

### Setup WildFly

NOTE : _**Below configurations for Wildfly can be done only with archive versions 5.16.1 onwards. Older archive versions rely on Query DSL which is incompatible with Hibernate 5.x included in Wildfly versions 13.0.0 onwards.**_

1.  Copy configuration files into the WildFly installation:

    ```
     > cp -r $DCM4CHEE_ARC/configuration $WILDFLY_HOME/standalone [UNIX]
     > xcopy %DCM4CHEE_ARC%\configuration %WILDFLY_HOME%\standalone\configuration [Windows]
       (Select D = directory)
    ```

    _Note_: Beside LDAP Connection configuration `dcm4chee-arc/ldap.properties`, the private key `keystores/key.p12` and the trusted CA certificate `keystores/cacerts.p12` used in TLS connections are not stored in LDAP.

    _**Ensure that one's LDAP specific values are present in the file****&#x20;****`dcm4chee-arc/ldap.properties`****.**_
2.  a. Upto archive version 5.23.3, the Jakarta Full Platform Profile certified configuration can be used as base configuration. To preserve the original WildFly configuration one may copy the original configuration file for Jakarta Full Platform Profile:

    ```
     > cd $WILDFLY_HOME/standalone/configuration/
     > cp standalone-full.xml dcm4chee-arc.xml
    ```

    b. Archive Version 5.24.0 onwards, the Jakarta EE Web Profile certified configuration can be used as base configuration. To preserve the original WildFly configuration one may copy the original configuration file for Jakarta EE Web Profile:

    ```
     > cd $WILDFLY_HOME/standalone/configuration/
     > cp standalone.xml dcm4chee-arc.xml
    ```
3.  Install JBoss module containing DCM4CHE libraries, Keycloak admin client and Apache commons.

    ```
     > cd  $WILDFLY_HOME
     > unzip $DCM4CHEE_ARC/jboss-modules/dcm4che-jboss-modules-5.x.x.zip
    ```
4.  Install JAI Image IO 1.2 libraries as JBoss module (needed for compression/decompression, does not work on Windows 64 bit and Mac OS X caused by missing native components for these platforms):

    ```
     > cd  $WILDFLY_HOME
     > unzip $DCM4CHEE_ARC/jboss-modules/jai_imageio-jboss-modules-1.2-pre-dr-b04.zip
    ```
5.  Install jclouds 2.2.1 libraries as JBoss modules:

    ```
     > cd  $WILDFLY_HOME
     > unzip $DCM4CHEE_ARC/jboss-modules/jclouds-jboss-modules-2.2.1-noguava.zip
    ```
6.  Install ecs-object-client 3.0.0 libraries as JBoss modules:

    ```
     > cd  $WILDFLY_HOME
     > unzip $DCM4CHEE_ARC/jboss-modules/ecs-object-client-jboss-modules-3.0.0.zip
    ```
7.  Version 5.22.4 onwards, download [Client Adapter for Wildfly from Keycloak](https://github.com/keycloak/keycloak/releases/download/15.0.0/keycloak-oidc-wildfly-adapter-15.0.0.zip) and unzip it inside `WILDFLY_HOME`.

    ```
     > cd  $WILDFLY_HOME
     > unzip $Downloads/keycloak-wildfly-adapter-dist-15.0.0.zip
    ```
8.  Except for H2, one has to install the JDBC Driver for the database.

    The JDBC driver can be installed either as a deployment or as a core module.

    For installation as a core module extract the corresponding ZIP file into $WILDFLY\_HOME, e.g.:

    ```
     > cd $WILDFLY_HOME
     > unzip $DCM4CHEE_ARC/jboss-modules/jdbc-jboss-modules-mysql-8.0.20.zip
    ```

    [Installation as deployment](https://docs.wildfly.org/18/Admin_Guide.html#jdbc-driver-installation) is limited to JDBC 4-compliant driver consisting of **one** JAR. You may either take the JDBC driver JAR from included core module ZIP, or download it from:

    * [MySQL](http://www.mysql.com/products/connector/)
    * [PostgreSQL](http://jdbc.postgresql.org/)
    * [Firebird](http://www.firebirdsql.org/en/jdbc-driver/)
    * [DB2](https://www.ibm.com/support/pages/db2-jdbc-driver-versions-and-downloads)
    * [Oracle](https://www.oracle.com/database/technologies/jdbc-ucp-122-downloads.html)
    * [Microsoft SQL Server](http://msdn.microsoft.com/data/jdbc/)
9.  Start WildFly in standalone mode with the correct configuration file:

    ```
     > $WILDFLY_HOME/bin/standalone.sh -c dcm4chee-arc.xml [UNIX]
     > %WILDFLY_HOME%\bin\standalone.bat -c dcm4chee-arc.xml [Windows]
    ```

    Verify, that JBoss started successfully, e.g.:

    ```
    =========================================================================

    Listening for transport dt_socket at address: 8787
    14:45:54,575 INFO  [org.jboss.modules] (main) JBoss Modules version 2.1.0.Final
    14:45:55,277 INFO  [org.jboss.msc] (main) JBoss MSC version 1.5.1.Final
    14:45:55,286 INFO  [org.jboss.threads] (main) JBoss Threads version 2.4.0.Final
    14:45:55,417 INFO  [org.jboss.as] (MSC service thread 1-2) WFLYSRV0049: WildFly Full 29.0.1.Final (WildFly Core 21.1.1.Final) starting
    14:45:56,239 INFO  [org.wildfly.security] (ServerService Thread Pool -- 28) ELY00001: WildFly Elytron version 2.2.1.Final
    :
    :
    2023-09-13 14:46:07,735 INFO  [org.jboss.as.webservices] (MSC service thread 1-1) WFLYWS0003: Starting service jboss.ws.endpoint."dcm4chee-arc-ear-5.31.2-psql.ear"."dcm4chee-arc-retrieve-xdsi-5.31.2.war"."org.dcm4chee.arc.retrieve.xdsi.ImageDocumentSource"
    2023-09-13 14:46:13,502 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 85) WFLYUT0021: Registered web context: '/dcm4chee-arc/xsl' for server 'default-server'
    2023-09-13 14:46:13,514 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 114) WFLYUT0021: Registered web context: '/dcm4chee-arc/xdsi' for server 'default-server'
    2023-09-13 14:46:13,533 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 113) WFLYUT0021: Registered web context: '/dcm4chee-proxy' for server 'default-server'
    2023-09-13 14:46:13,535 INFO  [io.undertow.websockets.jsr] (ServerService Thread Pool -- 115) UT026003: Adding annotated server endpoint class org.dcm4chee.arc.ups.rs.EventReportSender for path /aets/{AETitle}/ws/subscribers/{SubscriberAET}
    2023-09-13 14:46:13,572 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 115) WFLYUT0021: Registered web context: '/dcm4chee-arc' for server 'default-server'
    2023-09-13 14:46:14,300 INFO  [org.opencv.osgi] (ServerService Thread Pool -- 108) Successfully loaded OpenCV native library.
    2023-09-13 14:46:14,304 WARN  [org.dcm4chee.arc.impl.ArchiveDeviceProducer] (ServerService Thread Pool -- 108) UnzipVendorDataToURI=${jboss.server.temp.url}/dcm4chee-arc, but no Vendor Data
    2023-09-13 14:46:14,735 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Start TCP Listener on /0.0.0.0:2575
    2023-09-13 14:46:14,735 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Start TCP Listener on /0.0.0.0:11112
    2023-09-13 14:46:15,980 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Start TCP Listener on /0.0.0.0:12575
    2023-09-13 14:46:15,981 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Start TCP Listener on /0.0.0.0:2762
    2023-09-13 14:46:16,195 DEBUG [org.hibernate.SQL] (ServerService Thread Pool -- 108) select u1_0.ups_iuid,s1_0.subscriber_aet from subscription s1_0 join ups u1_0 on u1_0.pk=s1_0.ups_fk
    2023-09-13 14:46:16,251 DEBUG [org.hibernate.SQL] (ServerService Thread Pool -- 108) select g1_0.subscriber_aet from global_subscription g1_0 where g1_0.matchkeys_fk is null
    2023-09-13 14:46:16,258 DEBUG [org.hibernate.SQL] (ServerService Thread Pool -- 108) select g1_0.subscriber_aet from global_subscription g1_0 where g1_0.matchkeys_fk is not null
    2023-09-13 14:46:16,536 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0010: Deployed "dcm4chee-arc-ui2-5.31.2.war" (runtime-name : "dcm4chee-arc-ui2-5.31.2.war")
    2023-09-13 14:46:16,536 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0010: Deployed "dcm4chee-arc-ear-5.31.2-psql.ear" (runtime-name : "dcm4chee-arc-ear-5.31.2-psql.ear")
    2023-09-13 14:46:16,617 INFO  [org.jboss.as.server] (Controller Boot Thread) WFLYSRV0212: Resuming server
    2023-09-13 14:46:16,628 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
    2023-09-13 14:46:16,629 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
    2023-09-13 14:46:16,643 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 29.0.1.Final (WildFly Core 21.1.1.Final) started in 22503ms - Started 3346 of 3546 services (426 services are lazy, passive or on-demand) - Server configuration file in use: dcm4chee-arc.xml
    ```

    Running JBoss in domain mode should work, but was not yet tested.
10. If you installed the JDBC Driver as a core module in step 8, you have to add it into the server configuration using JBoss CLI in a new console window:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] /subsystem=datasources/jdbc-driver=<driver-name>:add(driver-name=<driver-name>,driver-module-name=<module-name>)
    ```

    One may choose any `<driver-name>` for the JDBC Driver, `<module-name>` must match the name defined in the module definition file `module.xml` of the JDBC driver, e.g.:

    ```
    [standalone@localhost:9990 /] /subsystem=datasources/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql)
    ```
11. Create and enable a new Data Source bound to JNDI name `java:/PacsDS` using JBoss CLI:

    ```
    [standalone@localhost:9990 /] data-source add --name=PacsDS \
    >     --driver-name=<driver-name> \
    >     --connection-url=<jdbc-url> \
    >     --jndi-name=java:/PacsDS \
    >     --user-name=<user-name> \
    >     --password=<user-password>
    ```

    The format of `<jdbc-url>` is JDBC Driver specific, e.g.:

    * H2: `jdbc:h2:<directory-path>/<database-name>`
    * MySQL: `jdbc:mysql://<host>:3306/<database-name>?serverTimezone=<timezone>`
    * PostgreSQL: `jdbc:postgresql://<host>:5432/<database-name>`
    * Firebird: `jdbc:firebirdsql:<host>/3050:<database-name>`
    * DB2: `jdbc:db2://<host>:50000/<database-name>`
    * Oracle: `jdbc:oracle:thin:@<host>:1521:<database-name>`
    * Microsoft SQL Server: `jdbc:sqlserver://<host>:1433;databaseName=<database-name>`

    There is also a CLI script `$DCM4CHEE_ARC/cli/add-data-source-<db>.cli` provided, which just applies the command to add the JDBC driver and creating the Data Source. Replace `<host>`, `<database-name>`, `<timezone>` (one may get this value from the system's timezone file), `<user-name>` and `<user-password>` by their actual values before executing it:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c --file=$DCM4CHEE_ARC/cli/add-data-source-<db>.cli
    ```
12. If one is using H2 as database, create tables and indexes using the H2 Console:
    1. Download the H2 Console WAR file from https://github.com/jboss-developer/jboss-eap-quickstarts/tree/7.1.0.GA/h2-console
    2.  Deploy it using JBoss CLI, e.g.:

        ```
         [standalone@localhost:9990 /] deploy ~/Downloads/h2-console.war
        ```
    3. Access the console at [http://localhost:8080/h2console](http://localhost:8080/h2console) and login with `JDBC URL`, `User name` und `Password` matching the values passed to above `data-source add --name=PacsDS`.
    4.  Create tables and indexes:

        ```
         RUNSCRIPT  FROM '$DCM4CHEE_ARC/sql/h2/create-h2.sql'
        ```
13. Only upto archive version 5.23.3, create JMS Queues using JBoss CLI:

    ```
    [standalone@localhost:9990 /] jms-queue add --queue-address=StgCmtSCP --entries=java:/jms/queue/StgCmtSCP
    [standalone@localhost:9990 /] jms-queue add --queue-address=StgCmtSCU --entries=java:/jms/queue/StgCmtSCU
    [standalone@localhost:9990 /] jms-queue add --queue-address=MPPSSCU --entries=java:/jms/queue/MPPSSCU
    [standalone@localhost:9990 /] jms-queue add --queue-address=IANSCU --entries=java:/jms/queue/IANSCU
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export1 --entries=java:/jms/queue/Export1
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export2 --entries=java:/jms/queue/Export2
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export3 --entries=java:/jms/queue/Export3
    [standalone@localhost:9990 /] jms-queue add --queue-address=HL7Send --entries=java:/jms/queue/HL7Send
    [standalone@localhost:9990 /] jms-queue add --queue-address=RSClient --entries=java:/jms/queue/RSClient
    [standalone@localhost:9990 /] jms-queue add --queue-address=DiffTasks --entries=java:/jms/queue/DiffTasks
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export4 --entries=java:/jms/queue/Export4
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export5 --entries=java:/jms/queue/Export5
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export6 --entries=java:/jms/queue/Export6
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export7 --entries=java:/jms/queue/Export7
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export8 --entries=java:/jms/queue/Export8
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export9 --entries=java:/jms/queue/Export9
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export10 --entries=java:/jms/queue/Export10
    [standalone@localhost:9990 /] jms-queue add --queue-address=StgVerTasks --entries=java:/jms/queue/StgVerTasks
    [standalone@localhost:9990 /] jms-queue add --queue-address=Rejection --entries=java:/jms/queue/Rejection
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve1 --entries=java:/jms/queue/Retrieve1
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve2 --entries=java:/jms/queue/Retrieve2
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve3 --entries=java:/jms/queue/Retrieve3
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve4 --entries=java:/jms/queue/Retrieve4
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve5 --entries=java:/jms/queue/Retrieve5
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve6 --entries=java:/jms/queue/Retrieve6
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve7 --entries=java:/jms/queue/Retrieve7
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve8 --entries=java:/jms/queue/Retrieve8
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve9 --entries=java:/jms/queue/Retrieve9
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve10 --entries=java:/jms/queue/Retrieve10
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve11 --entries=java:/jms/queue/Retrieve11
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve12 --entries=java:/jms/queue/Retrieve12
    [standalone@localhost:9890 /] jms-queue add --queue-address=Retrieve13 --entries=java:/jms/queue/Retrieve13
    ```

    Alternatively, execute same commands using provided CLI script `$DCM4CHEE_ARC/cli/add-jms-queues.cli`

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c --file=$DCM4CHEE_ARC/cli/add-jms-queues.cli
    ```

    Note : Archive version 5.24.0 onwards, addition of JMS queues is **not** required.
14. Enable property replacement in deployment descriptors by setting attribute `spec-descriptor-property-replacement` of the `ee` subsystem to `true`, and adjust the `managed-executor-services` and the `managed-scheduled-executor-service` configuration of the `ee` subsystem to avoid thread-pool related issues on long-running tasks or heavy load using JBoss CLI - you may configure a larger maximal number of threads than 100 according your needs:

    ```
    [standalone@localhost:9990 /] /subsystem=ee:write-attribute(name=spec-descriptor-property-replacement,value=true)
    [standalone@localhost:9990 /] /subsystem=ee/managed-executor-service=default:undefine-attribute(name=hung-task-threshold)
    [standalone@localhost:9990 /] /subsystem=ee/managed-executor-service=default:write-attribute(name=long-running-tasks,value=true)
    [standalone@localhost:9990 /] /subsystem=ee/managed-executor-service=default:write-attribute(name=core-threads,value=2)
    [standalone@localhost:9990 /] /subsystem=ee/managed-executor-service=default:write-attribute(name=max-threads,value=100)
    [standalone@localhost:9990 /] /subsystem=ee/managed-executor-service=default:write-attribute(name=queue-length,value=0)
    [standalone@localhost:9990 /] /subsystem=ee/managed-scheduled-executor-service=default:undefine-attribute(name=hung-task-threshold)
    [standalone@localhost:9990 /] /subsystem=ee/managed-scheduled-executor-service=default:write-attribute(name=long-running-tasks,value=true)
    ```

    Alternatively, execute same commands using provided CLI script `$DCM4CHEE_ARC/cli/adjust-managed-executor.cli`

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c --file=$DCM4CHEE_ARC/cli/adjust-managed-executor.cli
    ```

    Reload the configuration or restart Wildfly to effect the changes

    ```
    [standalone@localhost:9990 /] :reload
    ```
15. By default, DCM4CHEE Archive 5.x will assume `dcm4chee-arc` as its device name, used to find its configuration in the LDAP Server. Specify a different device name by system property `dcm4chee-arc.DeviceName` (if required) using JBoss CLI:

    ```
    [standalone@localhost:9990 /] /system-property=dcm4chee-arc.DeviceName:add(value=<device-name>)
    ```
16. By default, Wildfly supports only 10MB as maximum size of HTTP POST requests. Change this to a higher value as shown below

    ```
    [standalone@localhost:9990 /] /subsystem=undertow/server=default-server/http-listener=default:write-attribute(name=max-post-size,value=10000000000) 
    [standalone@localhost:9990 /] /subsystem=undertow/server=default-server/https-listener=https:write-attribute(name=max-post-size,value=10000000000) 
    ```

    Reload the configuration or restart Wildfly to effect the changes

    ```
    [standalone@localhost:9990 /] :reload
    ```
17. Upto version 5.23.2, deploy DCM4CHEE Archive 5.x using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ear-5.23.2-psql.ear
    ```

    Verify that DCM4CHEE Archive was deployed and started successfully, e.g.:

    ```
    2022-06-21 15:16:23,128 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-2) WFLYSRV0027: Starting deployment of "dcm4chee-arc-ear-5.23.2-psql.ear" (runtime-name: "dcm4chee-arc-ear-5.23.2-psql.ear")
    :
    2022-06-21 15:16:32,426 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Start TCP Listener on /0.0.0.0:11112
    2022-06-21 15:16:32,426 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Start TCP Listener on /0.0.0.0:2577
    2022-06-21 15:16:32,664 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Start TCP Listener on /0.0.0.0:12575
    2022-06-21 15:16:32,664 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Start TCP Listener on /0.0.0.0:2762
    2022-06-21 15:16:32,928 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0010: Deployed "dcm4chee-arc-ear-5.23.2-psql.ear" (runtime-name : "dcm4chee-arc-ear-5.23.2-psql.ear")
    ```

    Verify that the archive web UI is accessible at `http://localhost:8080/dcm4chee-arc/ui2`

    Undeploy DCM4CHEE Archive at any time using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ear-5.23.2-psql.ear

    2022-06-21 15:26:23,905 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 75) WFLYUT0022: Unregistered web context: /dcm4chee-arc
    2022-06-21 15:26:23,906 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 80) WFLYUT0022: Unregistered web context: /dcm4chee-arc/ui
    2022-06-21 15:26:23,912 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Stop TCP Listener on /0.0.0.0:11112
    2022-06-21 15:26:23,912 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Stop TCP Listener on /0.0.0.0:2762
    2022-06-21 15:26:23,915 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Stop TCP Listener on /0.0.0.0:12575
    2022-06-21 15:26:23,915 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Stop TCP Listener on /0.0.0.0:2575
    :
    2022-06-21 15:26:24,023 INFO  [org.jboss.as.server] (management-handler-thread - 8) WFLYSRV0009: Undeployed "dcm4chee-arc-ear-5.23.2-psql.ear" (runtime-name: "dcm4chee-arc-ear-5.23.2-psql.ear")
    ```
18. Version 5.23.3 onwards, archive UI and archive backend are independent of each other and shall be deployed separately. Deploy DCM4CHEE Archive Backend 5.x and DCM4CHEE Archive UI 5.x using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ear-5.31.2-psql.ear
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ui2-5.31.2.war
    ```

    Verify that DCM4CHEE Archive Backend and UI was deployed and started successfully, e.g.:

    ```
    15:05:10,547 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-3) WFLYSRV0027: Starting deployment of "dcm4chee-arc-ear-5.31.2-psql.ear" (runtime-name: "dcm4chee-arc-ear-5.31.2-psql.ear")
    :
    15:05:16,021 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-3) Start TCP Listener on /0.0.0.0:2575
    15:05:16,021 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Start TCP Listener on /0.0.0.0:11112
    15:05:16,290 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-8) Start TCP Listener on /0.0.0.0:12575
    15:05:16,290 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-9) Start TCP Listener on /0.0.0.0:2762
    15:05:16,375 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0010: Deployed "dcm4chee-arc-ear-5.31.2-psql.ear" (runtime-name : "dcm4chee-arc-ear-5.31.2-psql.ear")
    :
    15:04:27,085 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-8) WFLYSRV0027: Starting deployment of "dcm4chee-arc-ui2-5.31.2.war" (runtime-name: "dcm4chee-arc-ui2-5.31.2.war")
    15:04:27,572 INFO  [org.jboss.weld.deployer] (MSC service thread 1-1) WFLYWELD0003: Processing weld deployment dcm4chee-arc-ui2-5.31.2.war
    15:04:27,832 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 134) WFLYUT0021: Registered web context: '/dcm4chee-arc/ui2' for server 'default-server'
    15:04:27,858 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0010: Deployed "dcm4chee-arc-ui2-5.31.2.war" (runtime-name : "dcm4chee-arc-ui2-5.31.2.war")
    ```

    Verify that the archive web UI is accessible at `http://localhost:8080/dcm4chee-arc/ui2`.

    Undeploy DCM4CHEE Archive Backend and UI at any time using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ear-5.31.2-psql.ear
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ui2-5.31.2.war

    15:03:27,295 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 121) WFLYUT0022: Unregistered web context: '/dcm4chee-arc' from server 'default-server'
    15:03:27,296 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 118) WFLYUT0022: Unregistered web context: '/dcm4chee-arc/xsl' from server 'default-server'
    15:03:27,297 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 129) WFLYUT0022: Unregistered web context: '/dcm4chee-proxy' from server 'default-server'
    15:03:27,298 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 127) WFLYUT0022: Unregistered web context: '/dcm4chee-arc/xdsi' from server 'default-server'
    15:03:27,299 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-2) Stop TCP Listener on /0.0.0.0:11112
    15:03:27,299 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-4) Stop TCP Listener on /0.0.0.0:2762
    15:03:27,299 INFO  [org.dcm4che3.net.Connection] (EE-ManagedExecutorService-default-Thread-1) Stop TCP Listener on /0.0.0.0:2575
    :
    15:03:27,455 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0009: Undeployed "dcm4chee-arc-ear-5.31.2-psql.ear" (runtime-name: "dcm4chee-arc-ear-5.31.2-psql.ear")
    :
    15:02:39,869 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 85) WFLYUT0022: Unregistered web context: '/dcm4chee-arc/ui2' from server 'default-server'
    15:02:39,942 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-4) WFLYSRV0028: Stopped deployment dcm4chee-arc-ui2-5.31.2.war (runtime-name: dcm4chee-arc-ui2-5.31.2.war) in 79ms
    15:02:39,996 INFO  [org.jboss.as.repository] (management-handler-thread - 1) WFLYDR0002: Content removed from location /home/vrinda/work/archive/wildfly-29.0.1.Final/standalone/data/content/1e/e4d1fc5c7677f3b1472cd5f09bf5be48627aaf/content
    15:02:39,996 INFO  [org.jboss.as.server] (management-handler-thread - 1) WFLYSRV0009: Undeployed "dcm4chee-arc-ui2-5.31.2.war" (runtime-name: "dcm4chee-arc-ui2-5.31.2.war")
    ```
19. Starting with Wildfly 29.0.1, archive server log may contain several occurrences INFO messages from underlying Wildfly cached connection manager

    ```
    14:09:59,341 INFO  [org.jboss.jca.core.connectionmanager.listener.TxConnectionListener] (default task-4) IJ000311: Throwable from unregister connection: java.lang.IllegalStateException: IJ000152: Trying to return an unknown connection: org.jboss.jca.adapters.jdbc.jdk8.WrappedConnectionJDK8@617cc86c
            at org.jboss.ironjacamar.impl@3.0.3.Final//org.jboss.jca.core.connectionmanager.ccm.CachedConnectionManagerImpl.unregisterConnection(CachedConnectionManagerImpl.java:408)
            at org.jboss.ironjacamar.impl@3.0.3.Final//org.jboss.jca.core.connectionmanager.listener.TxConnectionListener.connectionClosed(TxConnectionListener.java:645)
            .........
    ```

    Optionally disable these by

    ```
    [standalone@localhost:9990 /] /subsystem=datasources/data-source=pacsds:write-attribute(name=use-ccm,value=false)
    ```

Refer Secured Archive Installation, for :

* Use / test of [secured DICOM Upper Layer services](https://github.com/dcm4che/dcm4chee-arc-light/issues/2703), optionally using [dcm4che tools](https://github.com/dcm4che/dcm4che/issues/768).
* Using secured version of archive.
