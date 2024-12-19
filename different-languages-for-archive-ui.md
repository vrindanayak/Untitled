# Different languages for Archive UI

* Overview
* Supported Languages
* Steps to add new language for archive UI

### Overview

This page briefly describes how one can internationalize archive UI in a language other than English. The whole text content of archive UI is broadly divided in two parts :

* Text for UI different components and pages
* Various \[\[Configurations]] schemas in the archive

_**Currently, the archive UI can be configured in a different language only for the secured version of archive. Supporting it for the unsecured version of archive is**_ [_**not implemented yet**_](https://github.com/dcm4che/dcm4chee-arc-light/issues/2293)_**.**_

Apart from the language specific changes that you do in [dcm4chee-arc-lang](https://github.com/dcm4che/dcm4chee-arc-lang), you still have to open an [issue in dcm4chee-arc-light](https://github.com/dcm4che/dcm4chee-arc-light/iisues) to support your [language code in UI's list of languages](https://github.com/dcm4che/dcm4chee-arc-light/blob/master/dcm4chee-arc-ui2/src/app/constants/globalvar.ts#L320).

### Supported Languages

| **Language**      | **Locale** |
| ----------------- | ---------- |
| English (default) | en         |
| Spanish           | es         |
| Chinese           | zh         |
| German            | de         |
| Hindi             | hi         |
| Italian           | it         |
| Japanese          | ja         |
| Marathi           | mr         |
| Russian           | ru         |

### Steps to add new language for archive UI

#### Available Supported Languages

To use archive UI using one of the Supported Languages, build and configure the archive using following steps :

* Start _**secured version of archive**_ (secured UI OR secured UI + secured REST). Refer [Run secured archive services on a single host](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Run-secured-archive-services-on-a-single-host)
* Login with `root` user credentials, alternatively with a user having `root` role.
*   Configure one or more non-English languages in your UI configuration by adding a language configuration, as shown below

    ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/select-default-ui-config.png)

    ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/add-new-language-config.png)
*   Add a single non-English language, together with English ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/single-foreign-language.png)

    Or multiple non-English languages, together with English ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/multiple-foreign-languages.png)
*   Switch to your desired non-English language from top right.

    ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/switch-language.png)

    ![](https://raw.githubusercontent.com/wiki/dcm4che/dcm4chee-arc-light/images/language/switch-language-continued.png)

#### Language not available in [Supported Languages](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Different-languages-for-Archive-UI/#supported-languages)

To use archive UI using a language not part of Supported Languages, provide translations for

* Text for UI different components and pages
* Various \[\[Configurations]] schemas in the archive

as described below.

**UI Components' specific resources**

UI Components' specific resources are present as a key/value map in JSON format used by the UI for labeling UI components.

* Copy / clone English version of UI specific texts using [en.json](https://github.com/dcm4che/dcm4chee-arc-lang/blob/master/src/main/webapp/assets/locale/en.json) on your local machine and rename it to code specific to your locale. For eg. to create a file for Portuguese rename the cloned file to `pt.json`. Refer [ISO 639-1 two letter codes](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) of languages.
* In the `translations` section in the file, translate the text on the right-hand side of `:` representing `values`.
* _**Do not change the texts on the left-hand side of****&#x20;****`:`****. These are the****&#x20;****`keys`****&#x20;****used by the UI code.**_

**Archive Configuration Schemas**

Archive configuration schemas are various JSON Schema files, specifying the `name`, `type`, `description`, `default value` of configuration attributes of various configuration entities of the archive.

* Copy / clone English version of configuration schemas, either using
  * [json files](https://github.com/dcm4che/dcm4chee-arc-lang/tree/master/src/main/webapp/assets/schema)
  * [properties files](https://github.com/dcm4che/dcm4chee-arc-lang/tree/master/src/props)
* If you have cloned properties files, provide translations for `title` and `description` separated by `|`. These are present after the `keys` and are separated from `keys` by `:` eg. In [archiveAttributeCoercion.properties](https://github.com/dcm4che/dcm4chee-arc-lang/blob/master/src/props/archiveAttributeCoercion.properties), translate
  *   `Name`

      and
  *   `Arbitrary/Meaningful name of the Archive Attribute Coercion`

      in `archiveAttributeCoercion.cn:Name|Arbitrary/Meaningful name of the Archive Attribute Coercion`
* If you have cloned json files, provide translations _**only**_ for `title` and `description` eg. In [archiveAttributeCoercion.schema.json](https://github.com/dcm4che/dcm4chee-arc-lang/blob/master/src/main/webapp/assets/schema/archiveAttributeCoercion.schema.json), translate
  *   `Name`

      and
  *   `Arbitrary/Meaningful name of the Archive Attribute Coercion`

      in

      ```
      "cn": {
        "title": "Name",
        "description": "Arbitrary/Meaningful name of the Archive Attribute Coercion",
        "type": "string"
      }
      ```

**Supporting new language in UI languages' list**

* Once you have prepared the UI components specific texts in your locale json file and properties / json configuration schemas, create an issue / pull request in [Archive UI's language project - dcm4chee-arc-lang](https://github.com/dcm4che/dcm4chee-arc-lang) to enable supporting the new language.
* The project developers will then provide you with access to directly push your language files in the aforementioned repository.
  * Push your language locale specific file eg. `pt.json` (for Portuguese) in the [locale specific folder](https://github.com/dcm4che/dcm4chee-arc-lang/tree/master/src/main/webapp/assets/locale)
  * For your language specific configuration schemas, create your locale code specific folder eg. `pt` (for Portuguese) in the [schema folder](https://github.com/dcm4che/dcm4chee-arc-lang/tree/master/src/main/webapp/assets/schema) and add all the translated configuration schemas in here.
* The project developers shall also add support for the new language locale code by adding it in the [UI languages' list](https://github.com/dcm4che/dcm4chee-arc-light/blob/master/dcm4chee-arc-ui2/src/app/constants/globalvar.ts#L465-L521).
* With subsequent next released version of archive, you can start using your language for the UI by following steps mentioned in [Available Supported Languages](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Different-languages-for-Archive-UI#available-supported-languages)

### References

[Resources for providing different languages for the Web UI of dcm4chee-arc-light](https://github.com/dcm4che/dcm4chee-arc-lang#resources-for-providing-different-languages-for-the-web-ui-of-dcm4chee-arc-light)
