# Anonymize or Deidentify Data

This page briefly guides and acquaints the user with coercion type to anonymize or de-identify DICOM dataset attributes' values.

Content

* Brief Description
* Applicability using Archive Attribute Coercion rules
* Configurable fields on Archive Attribute Coercion rules
* De-identification Action Codes
* Supported De-identification Profiles

### Brief Description

A DICOM dataset coming in or going out of the archive may require anonymization or de-identification of certain attribute values, to allow DICOM data to be used for research or teaching purposes or to share with any third party systems. Anonymize or de-identify certain DICOM attributes with help of a _Anonymize Data_ coercion type.

Anonymization or de-identification of DICOM dataset attribute values is done in conformance with [DICOM Part 15's Attribute Confidentiality Profiles](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#chapter_E)

Refer

* \[\[Archive Coercions Basic Overview]] for general understanding on archive attribute coercions.

### Applicability using Archive Attribute Coercion rules

[New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) can be used to anonymize or de-identify DICOM attributes in :

* incoming C-STORE requests on receive of DICOM objects to archive sent from external systems
* outgoing C-STORE requests on sending DICOM objects from archive to external systems
* incoming (MWL) C-FIND requests to archive
* incoming (MWL) C-FIND responses to archive
* outgoing (MWL) C-FIND requests from archive
* incoming MPPS N-CREATE requests to archive

[Legacy Archive Attribute Coercions](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) can be used to anonymize or de-identify DICOM attributes in :

* outgoing C-STORE requests on sending DICOM objects from archive to external systems

### Configurable fields on Archive Attribute Coercion rules

Anonymizing or de-identifying DICOM dataset attributes can be applied using either :

(Recommended) Specifying one or more Supported De-identification Profiles as _comma separated_ in [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri), f.e. `deidentify:RetainDeviceIdentityOption,RetainUIDsOption` on [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/new-coercion.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/anonymize/bg-exp/new-anonymize-attrs.png)

OR

Selecting one or more Supported De-identification Profiles in field [De-identification(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmdeidentification) on [Legacy Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/legacy-coercion.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/anonymize/bg-exp/legacy-anonymize-attrs.png)

See

* [Important Notes](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Archive-Coercions-Basic-Overview#important-notes)

### De-identification Action Codes

Each de-identification profile or option protects or retains specific DICOM attributes in a dataset, based on specific de-identification action codes.

| Action Code | Meaning                                                                                                                             |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| D           | replace with a non-zero length value that may be a dummy value and consistent with the Value Representation                         |
| Z           | replace with a zero length value, or a non-zero length value that may be a dummy value and consistent with the Value Representation |
| X           | remove the attribute value                                                                                                          |
| U           | replace with a non-zero length UID that is internally consistent within a set of Instances                                          |
| K           | keep (unchanged for non-Sequence Attributes, cleaned for Sequences)                                                                 |

Action codes are applicable to both Sequence and non Sequence attributes. In the case of Sequences, the action is applicable to the Sequence and all of its contents.

### Supported De-identification Profiles

#### 1. [Basic Application Confidentiality Profile](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_E.2)

#### 2. [Retain Longitudinal Temporal Information Full Dates Option](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_E.3.6)

If this option is used in the coercion i.e. [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) is set as `deidentify:RetainLongitudinalTemporalInformationFullDatesOption`, following _**date time**_ fields in incoming DICOM objects are retained in the DB.

* `Curve Date (0008,0025)`
* `Curve Time (0008,0035)`
* `Expected Completion Date Time (0040,4011)`
* `Instance Coercion Date Time (0008,0015)`
* `Instance Creation Date (0008,0012)`
* `Instance Creation Time (0008,0013)`
* `Last Menstrual Date (0010,21D0)`
* `Observation Date Time (0040,A032)`
* `Observation Date (Trial) (0040,A192)`
* `Observation Time (Trial) (0040,A193)`
* `Overlay Date (0008,0024)`
* `Overlay Time (0008,0034)`
* `Performed Procedure Step End Date (0040,0250)`
* `Performed Procedure Step End Date Time (0040,4051)`
* `Performed Procedure Step End Time (0040,0251)`
* `Performed Procedure Step Start Date (0040,0244)`
* `Performed Procedure Step Start Date Time (0040,4050)`
* `Performed Procedure Step Start Time (0040,0245)`
* `Procedure Step Cancellation Date Time (0040,4052)`
* `Scheduled Procedure Step End Date (0040,0004)`
* `Scheduled Procedure Step End Time (0040,0005)`
* `Scheduled Procedure Step Modification Date Time (0040,4010)`
* `Scheduled Procedure Step Start Date (0040,0002)`
* `Scheduled Procedure Step Start Date Time (0040,4005)`
* `Scheduled Procedure Step Start Time (0040,0003)`
* `Timezone Offset From UTC (0008,0201)`
* `Acquisition Date (0008,0022)`
* `Acquisition Time (0008,0032)`
* `Admitting Date (0038,0020)`
* `Admitting Time (0038,0021)`
* `Series Date (0008,0021)`
* `Series Time (0008,0031)`
* `Study Date (0008,0020)`
* `Study Time (0008,0030)`
* `Acquisition Date Time (0008,002A)`
* `Content Date (0008,0023)`
* `Content Time (0008,0033)`
* `End Acquisition Date Time (0018,9517)`
* `Start Acquisition Date Time (0018,9516)`
* `Verification Date Time (0040,A030)`

#### 3. [Retain Device Identity Option](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_E.3.8)

If this option is used in the coercion i.e. [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) is set as `deidentify:RetainDeviceIdentityOption`, following _**device identity**_ specific fields in incoming DICOM objects are retained in the DB.

* `Cassette ID (0018,1007)`
* `Gantry ID (0018,1008)`
* `Generator ID (0018,1005)`
* `Performed Station AE Title (0040,0241)`
* `Performed Station Geographic Location Code Sequence (0040,4030)`
* `Performed Station Name (0040,0242)`
* `Performed Station Name Code Sequence (0040,4028)`
* `Plate ID (0018,1004)`
* `Scheduled Procedure Step Location (0040,0011)`
* `Scheduled Station AE Title (0040,0001)`
* `Scheduled Station Geographic Location Code Sequence (0040,4027)`
* `Scheduled Station Name (0040,0010)`
* `Scheduled Station Name Code Sequence (0040,4025)`
* `Scheduled Study Location (0032,1020)`
* `Scheduled Study Location AE Title (0032,1021)`
* `Source Serial Number (3008,0105)`
* `Detector ID (0018,700A)`
* `Device Serial Number (0018,1000)`
* `Station Name (0008,1010)`
* `Device UID (0018,1002)`

#### 4. [Retain Institution Identity Option](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_E.3.11)

If this option is used in the coercion i.e. [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) is set as `deidentify:RetainInstitutionIdentityOption`, following _**institution identity**_ specific fields in incoming DICOM objects are retained in the DB.

* `Institution Name (0008,0080)`
* `Institution Code Sequence (0008,0082)`
* `Institution Address (0008,0081)`
* `Institutional Department Name (0008,1040)`
* `Institutional Department Type Code Sequence (0008,1041)`

#### 5. [Retain UIDs Option](https://dicom.nema.org/medical/dicom/current/output/html/part15.html#sect_E.3.9)

If this option is used in the coercion i.e. [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) is set as `deidentify:RetainUIDsOption`, SOP Class & SOP Instance UIDs within following fields in incoming DICOM objects are retained in the DB.

* `Referenced Performed Procedure Step Sequence (0008,1111)`
* `Referenced Study Sequence (0008,1110)`

#### 6. `Retain Patient ID Hash Option` ([proprietary to DCM4CHEE 5](https://github.com/dcm4che/dcm4chee-arc-light/issues/3696))

If this option is used in the coercion i.e. [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) is set as `deidentify:RetainPatientIDHashOption`, _**patient identifier**_ values in incoming DICOM objects are hashed and are retained in the DB.

* `Patient ID (0010,0020)`
* `Issuer of Patient ID (0010,0021)`
* `Issuer of Patient ID Qualifiers Sequence (0010,0024)`
