# Upgrade

For older releases go [here](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Upgrade-older-releases)

**Latest released archive version is 5.31.2**

## Upgrade DCM4CHEE Archive 5.x

Content

* Update Database Schema
  * H2
  * PostgreSQL
  * MySQL and MariaDB
  * Firebird
  * DB2
  * Oracle
  * MS SQL Server
* Update LDAP Schema
  * OpenLDAP
    * OpenLDAP with slapd.conf configuration file
    * OpenLDAP with dynamic runtime configuration
  * OpenDJ
  * Apache DS 2.0.0
* Update WildFly deployment

### Update Database Schema

SQL scripts for updating the database schema from previous versions can be found in directory `$DCM4CHEE_ARC/sql`. A change in the DB schema is reflected by an incremented second component of the version number (e.g. from 5.28.x to 5.29.y). So one does not need to update the database schema for upgrading from a version which only differs in the third component of the version number (e.g. from 5.26.0 to 5.26.1). For upgrading from an older version which second component of the version number in not the previous number, one has to first update the database schema to the previous version - by applying update scripts for previous versions in the right order - before applying the update script for the current version.

Note : Archive version 5.24.0 onwards, JMS messaging subsystem of Wildfly is no longer used for queue messages and tasks processing. While upgrading from an older archive version (i.e. any version upto 5.23.0) to archive versions 5.24.0 or higher, ensure that any existing queue messages and / or export tasks and / or retrieve tasks are fully processed before the following DB scripts are executed as the `queue_msg` and individual task tables (i.e. `export_task`, `retrieve_task`, `stgver_task` and `diff_task` / `diff_task_attrs` tables) are deleted as part of the 5.24.0 database scripts.

#### H2

One has to start Wildfly before updating the tables using the H2 console. Access the console at [http://localhost:8080/h2console](http://localhost:8080/h2console), login and update the database scheme by:

```
    RUNSCRIPT  FROM '$DCM4CHEE_ARC/sql/h2/update-5.29-h2.sql'
```

#### PostgreSQL

```
    > psql -h localhost <database-name> <user-name> < $DCM4CHEE_ARC/sql/psql/update-5.29-psql.sql
```

#### MySQL and MariaDB

```
    > mysql -u <user-name> -p<user-password> <database-name> < $DCM4CHEE_ARC/sql/mysql/update-5.29-mysql.sql
```

#### Firebird

```
    > isql-fb
    Use CONNECT or CREATE DATABASE to specify a database
    SQL> connect 'localhost:<database-name>'
    CON> user '<user-name>' password '<user-password>';
    SQL> in DCM4CHEE_ARC/sql/firebird/update-5.29-firebird.sql;
    SQL> exit;
```

#### DB2

```
    > su <user-name>
    Password: <user-password>
    > db2 connect to <database-name>
    > db2 -t < $DCM4CHEE_ARC/sql/db2/update-5.29-db2.sql
    > db2 terminate
```

#### Oracle

```
    $ sqlplus <user-name>/<user-password>
    SQL> @$DCM4CHEE_ARC/sql/oracle/update-5.29-oracle.sql
```

#### MS SQL Server

_Not yet tested_

### Update LDAP Schema

#### OpenLDAP

**OpenLDAP with slapd.conf configuration file**

1.  Replace previous schema files in OpenLDAP schema configuration directory by new versions from DCM4CHEE Archive distribution:

    ```
    > cp $DCM4CHEE_ARC/ldap/schema/* /etc/openldap/schema/ [UNIX]
    > copy %DCM4CHEE_ARC%\ldap\schema\* \Program Files\OpenLDAP\schema\ [Windows]
    ```
2.  Restart slapd:

    ```
    > sudo service slapd restart [UNIX]
    ```

**OpenLDAP with dynamic runtime configuration**

Update LDAP schemas in OpenLDAP runtime configuration by applying provided LDIF files using OpenLDAP CL utility ldapmodify:

```
> sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dcm4che-modify.ldif
> sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dcm4chee-archive-modify.ldif
> sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f $DCM4CHEE_ARC/ldap/slapd/dcm4chee-archive-ui-modify.ldif
```

#### OpenDJ

1.  Replace previous schema files in OpenDJ schema configuration directory by new versions from DCM4CHEE Archive distribution:

    ```
    > cp $DCM4CHEE_ARC/ldap/opendj/* $OPENDJ_HOME/config/schema/ [UNIX]
    > copy %DCM4CHEE_ARC%\ldap\opendj\* %OPENDJ_HOME%\config\schema\ [Windows]
    ```
2.  Restart OpenDJ by

    ```
    > $OPENDJ_HOME/bin/stop-ds
    > $OPENDJ_HOME/bin/start-ds
    ```

#### Apache DS 2.0.0

1.  Delete the `ou=objectclasses` child entry from the schema entries

    ```
    ou=schema
    + cn=dcm4chee-archive-ui
      + ou=objectclasses
    + cn=dcm4chee-archive
      + ou=objectclasses
    + cn=dcm4che
      + ou=objectclasses
    ```

    before deleting the schema entries itself, using the _Delete Entry_ function of Apache Directory Studio LDAP Browser.
2.  Import new LDAP schema files for Apache DS:

    ```
    $DCM4CHEE_ARC/ldap/apacheds/dcm4che.ldif
    $DCM4CHEE_ARC/ldap/apacheds/dcm4chee-archive.ldif
    $DCM4CHEE_ARC/ldap/apacheds/dcm4chee-archive-ui.ldif
    ```

    using the LDIF import function of Apache Directory Studio LDAP Browser.

### Update LDAP Data

* If one made structural changes - e.g. by adding/removing Network Application Entities or by configuring multiple Archive devices - to the provided default configuration, one has to adjust the provided `update-config-VERSION.ldif` files in `$DCM4CHEE_ARC/ldap` according to one's changes, before applying them, using the LDIF import function of Apache Directory Studio LDAP Browser or using LDAP CLI utility `ldapmodify` of OpenLDAP.
  *   If upgrading upto version 5.22.6 : To update from an older version than the most-recent previous version, e.g. from 5.22.1 to 5.22.6, one has to apply the update scripts for the previous versions, e.g.:

      ```
      > $ cd $DCM4CHEE_ARC/ldap
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f update-config-5.22.2.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f update-config-5.22.3.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f update-config-5.22.4.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f update-config-5.22.5.ldif
      ```

      before one can update the LDAP configuration for the current version by:

      ```
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f update-config-5.22.6.ldif
      Enter LDAP Password:
      modifying entry "dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org"
      :
      ```
  *   If upgrading to version 5.23.0 and above : Version 5.23.0 onwards, [individual ldif files are provided for an update of device and AE specific configuration](https://github.com/dcm4che/dcm4chee-arc-light/issues/2865). To update from an older version than the most-recent previous version, e.g. from 5.30.0 to 5.31.2, one has to apply the update scripts for the previous versions, e.g.:

      ```
      > $ cd $DCM4CHEE_ARC/ldap
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.0/update-AS_RECEIVED-5.31.0.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.0/update-DCM4CHEE-5.31.0.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.0/update-IOCM_REGULAR_USE-5.31.0.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.0/update-dev-5.31.0.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.0/update-storescp-5.31.0.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.1/update-dev-5.31.1.ldif
      ```

      before one can update the LDAP configuration for the current version by:

      ```
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.2/update-AS_RECEIVED-5.31.2.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.2/update-DCM4CHEE-5.31.2.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.2/update-dev-5.31.2.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.2/update-IOCM_REGULAR_USE-5.31.2.ldif
      > $ ldapmodify -xW -Dcn=admin,dc=dcm4che,dc=org -f 5.31.2/update-storescp-5.31.2.ldif
      Enter LDAP Password:
      modifying entry "dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org"
      :
      ```
*   Note :

    * There are no update ldif scripts for archive versions 5.25.1 / 5.25.2 / 5.27.0 / 5.29.0
    * There are no update ldif scripts for 5.24.2. However, if UI configuration has been previously imported in archive's installation

    ```
    $DCM4CHEE_ARC/ldap/default-ui-config.ldif
    ```

    and archive is being upgraded from any version older than or equal to 5.23.3 to a newer version, then delete the UI configuration and import it again. It is a [workaround for missing / incomplete update scripts concerning UI related config](https://github.com/dcm4che/dcm4chee-arc-light/issues/3399)
* Before v5.10.4, [any configuration change of the archive using the UI prevents further emission of Audit messages](https://github.com/dcm4che/dcm4chee-arc-light/issues/783) caused by the insert of an universal Audit Suppress Criteria to existing Audit Loggers of the Archive Device. You may either remove that Audit Suppress Criteria from the Audit Logger(s)
  * using the UI in v5.10.4+, or
  *   remove the Audit Logger child node directly in LDAP by applying

      ```
      dn: cn=cn,cn=Audit Logger,dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org
      changetype: delete
      ```
*   In v5.10.2, the configuration which Storage System is used by a particular Archive AE changed: The LDAP attribute which reference the Storage ID for object storage used by the AE changed from `dcmStorageID` to `dcmObjectStorageID`, and it's no longer possible to configure a default Storage ID used for object storage on Device level. The corresponding lines in `update-config-5.10.2.ldif` are

    ```
    dn: dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org
    changetype: modify
    delete: dcmStorageID
    -

    dn: dicomAETitle=DCM4CHEE,dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org
    changetype: modify
    add: dcmObjectStorageID
    dcmObjectStorageID: fs1
    -
    ```

    If one changed the default Storage ID `fs1`, one may either adjust `update-config-5.10.2.ldif` before applying it oneself, or apply one's change afterwards again - either directly in LDAP or using the UI.
*   Attribute Coercions (used either on Archive Device or Archive Network AE level) : The LDAP attribute which referenced the _**AE Title**_ (`dcmAETitle`) and _**Hostname**_ (`dcmHostname`) have been now removed and no longer supported. These can now be specified using _**Conditions**_ (`dcmProperty`) field dependent on their _**DICOM Transfer Role**_ (`dicomTransferRole`).

    | Restriction | SCU                                        | SCP                                          |
    | ----------- | ------------------------------------------ | -------------------------------------------- |
    | AE Title    | `SendingApplicationEntityTitle={ae-title}` | `ReceivingApplicationEntityTitle={ae-title}` |
    | Hostname    | `SendingHostname={hostname}`               | `ReceivingHostname={hostname}`               |
*   In v5.19.1, following configurations for Invoke Image Display URLs have changed. Invoke Image Display URLs : The LDAP attribute which referenced the _**Invoke Image Display Patient URL**_ (`dcmInvokeImageDisplayPatientURL`) and _**Invoke Image Display Study URL**_ (`dcmInvokeImageDisplayStudyURL`) used either on Archive Device or Archive Network AE level have been now removed and no longer supported. This has been now moved on Web Application level, where it can be set using Property (`dcmProperty`) in the format `<name>=<value>`.

    Replace any existing configuration

    | Name (LDAP Attribute)                                              | Configured on Level                  | Value (example)                                                     |
    | ------------------------------------------------------------------ | ------------------------------------ | ------------------------------------------------------------------- |
    | Invoke Image Display Patient URL (dcmInvokeImageDisplayPatientURL) | Archive Device or Archive Network AE | http(s)://:/IHEInvokeImageDisplay?requestType=PATIENT\&patientID={} |
    | Invoke Image Display Study URL (dcmInvokeImageDisplayStudyURL)     | Archive Device or Archive Network AE | http(s)://:/IHEInvokeImageDisplay?requestType=STUDY\&studyUID={}    |

    By

    | Name (LDAP Attribute)  | Configure on Level | Value (example)                                                                                                                                       |
    | ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
    | Property (dcmProperty) | Web Application    | IID\_PATIENT\_URL=weasis://$dicom:rs --url "\{{qidoBaseURL\}}\{{qidoBasePath\}}" -r "patientID=\{{patientID\}}" --query-ext "\&includedefaults=false" |
    | Property (dcmProperty) | Web Application    | IID\_STUDY\_URL=weasis://$dicom:rs --url "\{{qidoBaseURL\}}\{{qidoBasePath\}}" -r "studyUID=\{{studyUID\}}" --query-ext "\&includedefaults=false"     |
    | Property (dcmProperty) | Web Application    | IID\_URL\_TARGET=\_self                                                                                                                               |

    For secured version of archive, skip the above point and instead follow the changes mentioned in Upgrade Secured IID
* Version 5.19.1 onwards, the new Study page uses Web Applications also for querying the own archive. In default configuration, the Web Applications uses `http` and `https` connection references which are configured with hostname as `localhost` by default. If one is accessing the archive UI from a browser which is not on the same host as that of archive, then one needs to change the hostname of these connections from `localhost` to actual hostname or IP address where the archive is running.
* For any existing vendor data present in archive, download it (present in `Configuration -> Devices -> dcm4chee-arc`) and compare the contents with `vendor-data.zip` from release package. If there are differences in the [stylesheets provided by archive](https://github.com/dcm4che/dcm4chee-arc-light/tree/master/dcm4chee-arc-conf-data/src/main/resources), then update \[\[Vendor Data]]

### Update WildFly Deployment

Latest version of DCM4CHEE Archive available is 5.31.2

* Version 5.22.0 onwards, the private key `key.jks` and the trusted CA certificate `cacerts.jks` files shall be moved from `$WILDFLY_HOME/standalone/configuration/dcm4chee-arc` to `$WILDFLY_HOME/standalone/configuration/keystores` to accommodate the [configuration changes](https://github.com/dcm4che/dcm4chee-arc-light/issues/2494). Version 5.22.5 onwards, replace the `key.jks` and `cacerts.jks` in `$WILDFLY_HOME/standalone/configuration/keystores` with `key.p12` and `cacerts.p12` files.
*   Version 5.22.4 onwards, download [Client Adapter for Wildfly from Keycloak](https://www.keycloak.org/downloads.html) and unzip it inside `WILDFLY_HOME`.

    ```
      > cd  $WILDFLY_HOME
      > unzip $Downloads/keycloak-wildfly-adapter-dist-11.0.1.zip
    ```
*   Version 5.22.5 onwards, the formats of default keystore and truststore in LDAP have been [changed from JKS to PKCS12](https://github.com/dcm4che/dcm4chee-arc-light/issues/2733). Hence, copy these configuration files into the WildFly installation:

    ```
      > cp $DCM4CHEE_ARC/configuration/keystores/key.p12 $WILDFLY_HOME/standalone/configuration/keystores [UNIX]
      > cp $DCM4CHEE_ARC/configuration/keystores/cacerts.p12 $WILDFLY_HOME/standalone/configuration/keystores [UNIX]
      > xcopy %DCM4CHEE_ARC%\configuration\keystores\key.p12 %WILDFLY_HOME%\standalone\configuration\keystores [Windows]
      > xcopy %DCM4CHEE_ARC%\configuration\keystores\cacerts.p12 %WILDFLY_HOME%\standalone\configuration\keystores [Windows]
        (Select D = directory)
    ```

    _Note_: The private key `keystores/key.p12` and the trusted CA certificate `keystores/cacerts.p12` used in TLS connections are not stored in LDAP.

If upgrading from a version older than 5.16.1, it requires complete new setup of Wildfly (version 14 onwards). **QueryDSL is no longer supported with DCM4CHEE Archive version 5.16.1 onwards.**

1.  Update DCM4CHE dcm4chee-arc-light libraries as JBoss modules. This also contains the Keycloak Admin Client (Version 5.22.4 onwards, this module is required)

    ```
     > cd  $WILDFLY_HOME
     > rm -r modules/org/dcm4che
     > unzip $DCM4CHEE_ARC/jboss-modules/dcm4che-jboss-modules-dcm4chee-arc-light-5.x.x.zip
     
    ```
2.  If upgrading to version 5.19.1 or newer, update jclouds libraries to 2.3.0 as JBoss modules:

    ```
     > cd  $WILDFLY_HOME
     > rm -r modules/org/apache/jclouds
     > unzip $DCM4CHEE_ARC/jboss-modules/jclouds-jboss-modules-2.3.0-noguava.zip
    ```
3.  (Re-)Start WildFly in standalone mode with the correct configuration file:

    ```
     > $WILDFLY_HOME/bin/standalone.sh -c dcm4chee-arc.xml [UNIX]
     > %WILDFLY_HOME%\bin\standalone.bat -c dcm4chee-arc.xml [Windows]
    ```
4.  If upgrading from version 5.5.x or older and only upto archive version 5.23.3, create JMS Queue `HL7Send` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=HL7Send --entries=java:/jms/queue/HL7Send
    ```
5.  If upgrading from version 5.6.x or older and only upto archive version 5.23.3, create JMS Queue `StgCmtSCU` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=StgCmtSCU --entries=java:/jms/queue/StgCmtSCU
    ```
6.  If upgrading from version 5.7.2 or older and only upto archive version 5.23.3, create JMS Queue `RSClient` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=RSClient--entries=java:/jms/queue/RSClient
    ```
7.  If upgrading from version 5.10.4 or older and only upto archive version 5.23.3, create JMS queue `CMoveSCU` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=CMoveSCU --entries=java:/jms/queue/CMoveSCU
    ```
8.  If upgrading from version 5.12.0 or older and only upto archive version 5.23.3, create JMS queue `DiffTasks` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=DiffTasks --entries=java:/jms/queue/DiffTasks
    ```
9.  If upgrading from version 5.13.0 or older and only upto archive version 5.23.3, create JMS queue `Export4` and `Export5` using JBoss CLI:

    ```
     > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
     > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
     [standalone@localhost:9990 /] jms-queue add --queue-address=Export4 --entries=java:/jms/queue/Export4
     [standalone@localhost:9990 /] jms-queue add --queue-address=Export5 --entries=java:/jms/queue/Export5
    ```
10. If upgrading to version 5.14.1 or newer and only upto archive version 5.23.3, create JMS Queue `StgVerTasks` using JBoss CLI:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] jms-queue add --queue-address=StgVerTasks --entries=java:/jms/queue/StgVerTasks
    ```
11. If upgrading from version 5.14.1 or older and only upto archive version 5.23.3, create additional JMS queues `Export6`, `Export7`, `Export8`, `Export9` and `Export10`by default to enable greater configurability for processing tasks from various exporters using JBoss CLI:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export6 --entries=java:/jms/queue/Export6
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export7 --entries=java:/jms/queue/Export7
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export8 --entries=java:/jms/queue/Export8
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export9 --entries=java:/jms/queue/Export9
    [standalone@localhost:9990 /] jms-queue add --queue-address=Export10 --entries=java:/jms/queue/Export10
    ```
12. If upgrading from version 5.15.1 or older and only upto archive version 5.23.3, create JMS Queue `Rejection` using JBoss CLI:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] jms-queue add --queue-address=Rejection --entries=java:/jms/queue/Rejection
    ```
13. If upgrading to 5.16.2 or newer and only upto archive version 5.23.3, remove the `CMoveSCU` queue and create new JMS queues for `Retrieve`

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] jms-queue remove --queue-address=CMoveSCU
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve1 --entries=java:/jms/queue/Retrieve1
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve2 --entries=java:/jms/queue/Retrieve2
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve3 --entries=java:/jms/queue/Retrieve3
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve4 --entries=java:/jms/queue/Retrieve4
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve5 --entries=java:/jms/queue/Retrieve5
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve6 --entries=java:/jms/queue/Retrieve6
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve7 --entries=java:/jms/queue/Retrieve7
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve8 --entries=java:/jms/queue/Retrieve8
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve9 --entries=java:/jms/queue/Retrieve9
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve10 --entries=java:/jms/queue/Retrieve10
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve11 --entries=java:/jms/queue/Retrieve11
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve12 --entries=java:/jms/queue/Retrieve12
    [standalone@localhost:9990 /] jms-queue add --queue-address=Retrieve13 --entries=java:/jms/queue/Retrieve13
    ```
14. Upto version 5.23.2, undeploy old DCM4CHEE Archive 5.x.x (replace 5.x.x with version number to be undeployed) using JBoss CLI, e.g.:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ear-5.x.x-psql.ear
    ```
15. Version 5.23.3 onwards, old DCM4CHEE Archive Backend and UI 5.x.x shall be undeployed separately. (replace 5.x.x with version number to be undeployed) using JBoss CLI, e.g.:

    ```
    > $WILDFLY_HOME/bin/jboss-cli.sh -c [UNIX]
    > %WILDFLY_HOME%\bin\jboss-cli.bat -c [Windows]
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ear-5.x.old-psql.ear
    [standalone@localhost:9990 /] undeploy dcm4chee-arc-ui2-5.x.old.war
    ```
16. Upto version 5.23.2, deploy new DCM4CHEE Archive 5.x.x (replace 5.x.x by the version number you are deploying) using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ear-5.x.new-psql.ear
    ```
17. Version 5.23.3 onwards, deploy new DCM4CHEE Archive Backend and UI 5.x.x separately (replace 5.x.x by the version number you are deploying) using JBoss CLI, e.g.:

    ```
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ear-5.x.new-psql.ear
    [standalone@localhost:9990 /] deploy $DCM4CHEE_ARC/deploy/dcm4chee-arc-ui2-5.x.new.war
    ```
18. Optionally, as there are no longer `queue_msg`, and task specific tables (i.e. `export_task`, `retrieve_task`, `stgver_task` and `diff_task` / `diff_task_attrs`) in the database, choose to remove the corresponding JMS queues from archive's Wildfly configuration.

    ```
    [standalone@localhost:9990 /] jms-queue remove --queue-address=DiffTasks
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export1
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export2
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export3
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export4
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export5
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export6
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export7
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export8
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Export9
    [standalone@localhost:9990 /] jms-queue remove --queue-address=HL7Send
    [standalone@localhost:9990 /] jms-queue remove --queue-address=IANSCU
    [standalone@localhost:9990 /] jms-queue remove --queue-address=MPPSSCU
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Rejection
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve1
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve2
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve3
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve4
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve5
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve6
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve7
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve8
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve9
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve10
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve11
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve12
    [standalone@localhost:9990 /] jms-queue remove --queue-address=Retrieve13
    [standalone@localhost:9990 /] jms-queue remove --queue-address=RSClient
    [standalone@localhost:9990 /] jms-queue remove --queue-address=StgCmtSCU
    [standalone@localhost:9990 /] jms-queue remove --queue-address=StgCmtSCP
    [standalone@localhost:9990 /] jms-queue remove --queue-address=StgVerTasks
    ```
19. If upgrading secured version of archive, then also refer Secured Archive Wildfly Upgrade
20. If upgrading to archive versions 5.31.1+, underlying Wildfly is 29.0.1 Final. Archive server log may contain several occurrences INFO messages from underlying Wildfly cached connection manager.

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

Refer Secured Archive Upgrade, for :

* Use / test of [secured DICOM Upper Layer services](https://github.com/dcm4che/dcm4chee-arc-light/issues/2703), optionally using [dcm4che tools](https://github.com/dcm4che/dcm4che/issues/768).
* Upgrading secured version of archive.
