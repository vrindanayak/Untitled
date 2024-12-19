# Update Vendor Data

This page explains how one can add Vendor data to an archive device. Vendor data pertains to the stylesheets used by the archive for various purposes like processing of HL7 messages, coercion of object attributes to ensure DICOM VR compliance etc. The `vendor-data.zip` file is present in the distribution package or if one manually builds the dcm4chee-arc-light, it is present in `ldap` folder inside `/dcm4chee-arc-light/dcm4chee-arc-assembly/target/dcm4chee-arc-5.x.x-psql.zip`

### Configuration

#### Using Archive UI

1. Go to `Menu->Configuration`, then on `Devices` page, `Edit` the `dcm4chee-arc` device.
2. If one's device does not have the vendor data, one can see the `Upload` button against the field `Vendor Device Data`. Use `Upload` to select the `vendor-data.zip` file that is available in the release. One can find this zip file in the distribution package under `ldap` folder as `vendor-data.zip`.
3. If `Download` button is seen i.e Vendor data already present in device configuration, then one has the option to update the stylesheets in two ways :

* Complete update for new release : Delete existing vendor data (using the `X` against Vendor Data field) and upload the latest version that one has from the release. (Recommended)
* Partial Update : Download the existing vendor data as zip file and update the stylesheet that one wants to change. Next, delete existing vendor data (using the `X` against Vendor Data field) and upload the updated zip file back to the device.

#### Using LDAP

One may create a LDIF file and import it to the LDAP Server by using the _ldapmodify_ command line utility.

```
version: 1

dn: dicomDeviceName=dcm4chee-arc,cn=Devices,cn=DICOM Configuration,dc=dcm4che,dc=org
changetype: modify
add: dicomVendorData
dicomVendorData:< file:vendor-data.zip
```

* or use the _Add Attribute..._ and _Add Value..._ function of [Apache Directory Studio](https://directory.apache.org/studio/) to add attributes on Archive Device level (e.g.: `dicomDeviceName=dcm4chee-arc`) in the Archive Configuration. Use `Load Data` option to select the `vendor-data.zip` file and save it using `OK`.

One may refer to [Device](http://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#id1) to understand the description of the attribute.

Go to the Control tab on Configuration page in archive UI and reload the configuration.
