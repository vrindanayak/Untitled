# Supplement Dataset Attributes from Device

This page briefly guides and acquaints the user with coercion type to supplement DICOM dataset attributes' values from a device configured in archive's configuration.

Content

* Brief Description
* Applicability using Archive Attribute Coercion rules
* Configurable fields on Archive Attribute Coercion rules
* Supplementable Attributes

### Brief Description

A DICOM dataset coming in or going out of the archive may have one or more **missing DICOM attributes** in it. Supplement these missing attributes with help of a _Supplement dataset attributes from device_ coercion type.

The device which is used for supplementing shall contain one or more DICOM specific fields configured on it. These configured values are then used to supplement corresponding missing attributes in the applicable DICOM dataset.

Refer

* \[\[Archive Coercions Basic Overview]] for general understanding on archive attribute coercions.

### Applicability using Archive Attribute Coercion rules

[New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) can be used to supplement DICOM attributes to :

* incoming C-STORE requests on receive of DICOM objects to archive sent from external systems
* outgoing C-STORE requests on sending DICOM objects from archive to external systems
* incoming (MWL) C-FIND requests to archive
* incoming (MWL) C-FIND responses to archive
* outgoing (MWL) C-FIND requests from archive
* incoming MPPS N-CREATE requests to archive

[Legacy Archive Attribute Coercions](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) can be used to supplement DICOM attributes to :

* incoming C-STORE requests on receive of DICOM objects to archive sent from external systems
* incoming (MWL) C-FIND requests to archive
* incoming (MWL) C-FIND responses to archive
* outgoing (MWL) C-FIND requests from archive
* incoming MPPS N-CREATE requests to archive

See

* [Important Notes](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Archive-Coercions-Basic-Overview#important-notes)

### Configurable fields on Archive Attribute Coercion rules

Supplementing dataset attributes from device can be applied using either :

(Recommended) [Device Name Coercion Parameter](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmsupplementfromdevicereference) field with [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) as `merge-attrs:` on [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/new-coercion.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/new-supplement-from-device.png)

OR

[Supplement from Device](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmsupplementfromdevicereference) field on [Legacy Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/legacy-coercion.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/legacy-supplement-from-device.png)

See

* [Important Notes](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Archive-Coercions-Basic-Overview#important-notes)

### Supplementable attributes

[Device level attributes](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html) supplementable on each entity level are as follows :

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/supplement-device-attrs.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/supplement-device-attrs-institution.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/supplement-device-attrs-issuer.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/supplement-dataset/bg-exp/supplement-device-attrs-issuer-2.png)

Following table enlists the applicability of device level fields supplemented on each of the entities / datasets.

| <p>Supplement entity -><br>-----------------<br>Supplementable Device level fields<br>(below)</p>                                                                          | Instance                                          | MWL | MPPS                                               | Query                                             |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | --- | -------------------------------------------------- | ------------------------------------------------- |
| <p><a href="https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofpatientid">Issuer of Patient ID</a> (0010,0021)<br>(0010,0024)</p> | Y                                                 | Y   | Y                                                  | Y                                                 |
| [Issuer of Accession Number](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofaccessionnumber) (0008,0051)                      | <p>Y<br>also inside RequestAttributesSequence</p> | Y   | <p>Y<br>inside ScheduledStepAttributesSequence</p> | <p>Y<br>also inside RequestAttributesSequence</p> |
| [Issuer of Admission ID](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofadmissionid) (0038,0014)                              | Y                                                 | Y   | Y                                                  | Y                                                 |
| [Issuer of Service Episode ID](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofserviceepisodeid)                               | Y                                                 | Y   | Y                                                  | Y                                                 |
| [Issuer of Container Identifier](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofcontaineridentifier) (0040,0513)              | N                                                 | N   | N                                                  | Y                                                 |
| [Issuer of Specimen Identifier](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomissuerofspecimenidentifier) (0040,0562)                | N                                                 | N   | N                                                  | Y                                                 |
| [Order Placer Identifier](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomorderplaceridentifier) (0040,0026)                           | <p>Y<br>inside RequestAttributesSequence</p>      | Y   | <p>Y<br>inside ScheduledStepAttributesSequence</p> | <p>Y<br>inside RequestAttributesSequence</p>      |
| [Order Filler Identifier](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomorderfilleridentifier) (0040,0027)                           | <p>Y<br>inside RequestAttributesSequence</p>      | Y   | <p>Y<br>inside ScheduledStepAttributesSequence</p> | <p>Y<br>inside RequestAttributesSequence</p>      |
| [Manufacturer](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicommanufacturer) (0008,0070)                                               | Y                                                 | N   | N                                                  | N                                                 |
| [Manufacturer Model Name](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicommanufacturermodelname) (0008,1090)                           | Y                                                 | N   | N                                                  | N                                                 |
| [Software Version(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomsoftwareversion) (0018,1020)                                     | Y                                                 | N   | N                                                  | N                                                 |
| [Station Name](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomstationname) (0008,1010)                                                | Y                                                 | N   | N                                                  | N                                                 |
| [Device Serial Number](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicomdeviceserialnumber) (0018,1000)                                 | Y                                                 | N   | N                                                  | N                                                 |
| [Institution Name(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicominstitutionname) (0008,0080)                                     | Y                                                 | N   | N                                                  | N                                                 |
| [Institution Code(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicominstitutioncode) (0008,0082)                                     | Y                                                 | N   | N                                                  | N                                                 |
| [Institution Department Name(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicominstitutiondepartmentname) (0008,1040)                | Y                                                 | N   | N                                                  | N                                                 |
| [Institution Address(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/device.html#dicominstitutionaddress) (0008,0081)                               | Y                                                 | N   | N                                                  | N                                                 |
