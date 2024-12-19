# Archive Coercions Basic Overview

Content

* What is an Archive Attribute Coercion rule?
* Types of Archive Attribute Coercion rules
* Where to configure Archive Attribute Coercion rules?
* Usage of Archive Attribute Coercion rules
* Types of Coercion
* Application of Coercion Types by Archive Attribute Coercion rules
* Important Notes

### What is an Archive Attribute Coercion rule?

An archive attribute coercion rule is used to coerce DICOM attribute values on datasets coming in and / or going out of the archive.

### Types of Archive Attribute Coercion rules

Archive configurations currently contain 2 types of Archive Attribute Coercion rules :

* [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html)
* [Legacy Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html)

### Where to configure Archive Attribute Coercion rules?

Configure archive attribute coercion rules either on

* Archive Device Extension level : Applicable to DICOM datasets received or sent by **any** archive application entity. May be supplemented by attribute coercions configured for particular archive network AEs.

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/coercions-archive-device.png)

OR on

* Archive Network AE Extension level : Applicable to DICOM datasets received or sent by **this** archive application entity. Supplements attribute coercions configured on archive device extension level.

![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/attribute-coercions/coercions-archive-network-ae.png)

OR on both of the above, i.e.

* Archive Device Extension level + Archive Network AE Extension level

**Configurations in specific howtos shows the user how and what to configure using archive user interface and / or using Apache Directory Studio / ldif scripts.**

### Usage of Archive Attribute Coercion rules

Archive Attribute Coercions can be used to alter :

* Incoming datasets of

| [**DIMSE**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmDIMSE) **Transaction** | **External Client** [**DICOM Transfer Role**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dicomtransferrole) | **Archive Function**                                                     |
| --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| DICOM MPPS _N\_CREATE\_RQ_                                                                                                              | _SCU_                                                                                                                                                              | Archive receives MPPS requests from external systems                     |
| DICOM _C\_STORE\_RQ_ / STOW                                                                                                             | _SCU_                                                                                                                                                              | Archive receives DICOM objects via C\_STORE / STOW from external systems |
| DICOM _C\_FIND\_RQ_ / QIDO                                                                                                              | _SCU_                                                                                                                                                              | Archive receives DICOM C\_FIND or QIDO requests from external systems    |
| DICOM _C\_FIND\_RSP_ / QIDO                                                                                                             | _SCP_                                                                                                                                                              | Archive receives DICOM C\_FIND or QIDO responses from external systems   |

* Outgoing datasets of

| [**DIMSE**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dcmDIMSE) **Transaction** | **External Client** [**DICOM Transfer Role**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html#dicomtransferrole) | **Archive Function**                                                |
| --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| DICOM _C\_STORE\_RQ_ / WADO                                                                                                             | _SCP_                                                                                                                                                              | Archive sends DICOM objects via C\_STORE / WADO to external systems |
| DICOM _C\_FIND\_RQ_ / QIDO                                                                                                              | _SCP_                                                                                                                                                              | Archive sends DICOM C\_FIND or QIDO requests to external systems    |
| DICOM _C\_FIND\_RSP_ / QIDO                                                                                                             | _SCU_                                                                                                                                                              | Archive sends DICOM C\_FIND or QIDO responses to external systems   |

using one or more Coercion Types as described below.

### Types of Coercion

| **Coercion Type**                                  | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Merge from MWL                                     | Search for MWL in [external](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmmergemwlscp) or [local](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html#dcmmergelocalmwlscp) archive based on specific matching key (typically set to _AccessionNumber_ or _PatientIDAccessionNumber_) and coerce attributes according to configured / customized stylesheet specified in the URI field. |
| \[\[Merge or Nullify Dataset Attributes]]          | Used for merging or nullifying values of one or more DICOM attributes in a dataset using different [Attributes Formatting Options on Coercions](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Merge-or-Nullify-Dataset-Attributes#merge-attributes-formatting-options)                                                                                                                                                                                                                |
| Coerce from XSL                                    | Coerce one or more attributes using XSL stylesheet. Can be customized as per vendor's requirements.                                                                                                                                                                                                                                                                                                                                                                                        |
| \[\[Supplement Dataset Attributes from Device]]    | The device which is used for supplementing shall contain one or more DICOM fields configured on it, whose values are used for supplementing missing attributes in the dataset                                                                                                                                                                                                                                                                                                              |
| \[\[Anonymize or Deidentify Data]]                 | Anonymize one or more attributes in a dataset according to one or more [Supported De-identification Profiles](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Anonymize-or-Deidentify-Data#supported-de-identification-profiles)                                                                                                                                                                                                                                                        |
| Use Calling AE Title as                            | <p>Typically useful to set <em>Sending Application Entity Title of Series</em><br>DICOM field to calling AET of equipment storing studies to archive</p>                                                                                                                                                                                                                                                                                                                                   |
| Coerce attributes from Leading archive             | Query Leading archive for study using Study UID in received dataset and coerce based on configured \[\[Attribute Update Policy]]                                                                                                                                                                                                                                                                                                                                                           |
| \[\[Trim ISO 2022 Character Sets using Coercions]] | Used for replacing _single code for Single-Byte Character Sets with Code Extensions_ with _code for Single-Byte Character Sets without Code Extensions_ in a dataset.                                                                                                                                                                                                                                                                                                                      |
| Retrieve as Received                               | Retrieve object's dataset from local archive to be sent to a destination, as it was received from the source.                                                                                                                                                                                                                                                                                                                                                                              |
| Nullify Pixel Data                                 | Nullifies `Pixel Data (7FE0,0010)` in the dataset.                                                                                                                                                                                                                                                                                                                                                                                                                                         |

### Application of Coercion Types by Archive Attribute Coercion rules

For each combination of DIMSE transactions with incoming / outgoing datasets, **multiple Types of Coercion** can be applied

* Using **multiple** [**New Archive Attribute Coercion**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) **rules**, in any sequence and without the dependency on hardcoding of coercions' application as present in legacy archive attribute coercions above.
  * \[\[New Archive Attribute Coercion - Application of multiple coercions for one use case using multiple rules]]

OR

* Using a **single** [**Legacy Archive Attribute Coercion**](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) **rule** following a cascaded sequence depending on configured fields.
  * \[\[Legacy Archive Attribute Coercion - Application of multiple coercions using one coercion rule]]

### Important Notes

* Over long term, the [_Legacy Archive Attribute Coercion_ rules will be purged from archive code](https://github.com/dcm4che/dcm4chee-arc-light/issues/4034).
* If archive configuration contains matching rule for both [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) and [Legacy Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion.html) for a particular dataset combination, only the [New Archive Attribute Coercion](https://dcm4chee-arc-cs.readthedocs.io/en/latest/networking/config/archiveAttributeCoercion2.html) will get applied.
