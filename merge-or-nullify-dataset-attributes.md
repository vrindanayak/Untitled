# Merge or Nullify Dataset Attributes

This page briefly guides and acquaints the user with coercion type to merge or nullify DICOM dataset attributes' values.

Content

* Brief Description
* Applicability using Archive Attribute Coercion rules
* Configurable fields on Archive Attribute Coercion rules
* Merge Attributes Formatting Options

### Brief Description

A DICOM dataset coming in or going out of the archive may have one or more **missing or incorrectly valued DICOM attributes** in it. Merge or nullify such attributes with help of a _Merge or nullify dataset attributes_ coercion type.

Refer

* \[\[Archive Coercions Basic Overview]] for general understanding on archive attribute coercions.

### Applicability using Archive Attribute Coercion rules

[New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) or [Legacy Archive Attribute Coercions](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) can be used to merge or nullify DICOM attributes in :

* incoming C-STORE requests on receive of DICOM objects to archive sent from external systems
* outgoing C-STORE requests on sending DICOM objects from archive to external systems
* incoming (MWL) C-FIND requests to archive
* incoming (MWL) C-FIND responses to archive
* outgoing (MWL) C-FIND requests from archive
* incoming MPPS N-CREATE requests to archive

Note : [Legacy Archive Attribute Coercions](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) can not be used to nullify `Issuer of Patient ID (0010,0021)` and `Issuer of Patient ID Qualifiers Sequence (0010,0024)` DICOM attributes as there are separate configuration fields to handle their nullification.

### Configurable fields on Archive Attribute Coercion rules

Merging or nullifying dataset attributes can be applied using either :

(Recommended) [DICOM Attribute Coercion Parameters](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmmergeattribute) field with [Attribute Coercion URI](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmuri) as `merge-attrs:` on [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/new-coercion.png)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/merge-attrs/bg-exp/new-merge-nullify-attrs.png)

OR

Using [Legacy Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) fields

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/legacy-coercion.png)

* [Merge Attributes](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmmergeattribute)
* [Nullify Attributes](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmnullifytag)
* [Nullify Issuer of Patient ID](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmnullifyissuerofpatientid)

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/merge-attrs/bg-exp/legacy-merge-nullify-attrs.png)

See

* [Important Notes](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Archive-Coercions-Basic-Overview#important-notes)

### Merge Attributes Formatting Options

* Applicable format _{attributeID}={value}_ wherein specified _{attributeID}_ can be a _Keyword_ or _Tag_ number, some examples :
  * `PatientID=JMS{PatientID}`
  * `PatientID={PatientID,slice,3}`
  * `IssuerOfPatientID={00100010,hash}-{00100030}`
* _{attributeID}_ inside _{value}_ will be replaced by the value of that attribute in the original dataset.
* To nullify an attribute, just specify attribute without any value _{attributeID}=_, for example :
  * `AccessionNumber=`
  * similar to [Legacy archive attribute coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html)'s - [Nullify Attribute Tag(s)](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmnullifytag)

Formatting options available to merge DICOM dataset attributes are :

| Attributes format | Meaning                                                                                                   | Example                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `slice`           | <p>Slice attribute value from a certain index / position<br>(optionally upto an end index / position)</p> | <p><code>PatientID={PatientID,slice,3}</code><br><code>PatientID={PatientID,slice,3[,7]}</code></p>                          |
| `hash`            | Hash the value of specified attribute                                                                     | <p><code>IssuerOfPatientID={00100010,hash}-{00100030}</code><br><code>AccessionNumber=ACC-{StudyInstanceUID,hash}</code></p> |
| `upper`           | Convert specified attribute's value to upper case                                                         | `AccessionNumber={AccessionNumber,upper}`                                                                                    |
