# The MAML Metadata Format

**MAML** is a **YAML** based metadata format for tabular data (roughly implying Metadata **yAML**). When developing this format specification we were highly influenced by **VOTable** and **FITS** header formats, so there is a bias to this being astronomy-centric, but it should be functional for a wide variety of table types.

### Why MAML? 

We have **VOTable** and **FITS** header already?! Well, for various projects we were keen on a rich metadata format that was easy for humans and computers to both read and write. **VOTable** headers are very hard for humans to read and write (boo), and **FITS** is very restrictive with its formatting and only useful for **FITS** files directly. In comes **YAML**, a very human and machine readable and writeable format. By restricting ourselves to a narrow subset of the language we can easily describe fairly complex table metadata (including all IVOA information). So this brings us to **MAML**: Metadata **yAML** (it kinda works :-P).

**MAML** format files should be saved as example.maml etc. And the idea is the **YAML** string can be inserted directly into a number of different file formats that accept key-value metadata (like Apache Arrow Parquet files). In the case of Parquet files they should be written to a 'maml' extension in the metadata section of the file (so something like parquet_file\$metadata\$maml in R world).

## MAML Metadata Format

The **MAML** metadata format is a structured way to describe datasets, surveys, and tables using **YAML**. This format ensures that all necessary information about the data is captured in a clear and organized manner.

### Informal Structure Overview

The superset of allowed entries for **MAML** is below. Not all are required, but if present they should obey the order and naming.

- **survey**: The name of the survey. *Scalar string*. **[optional]**
- **dataset**: The name of the dataset. *Scalar string*. **[recommended]**
- **table**: The name of the table. *Scalar string*. **[required]**
- **version**: The version of the dataset. *Scalar string, integer or float*. **[required]**
- **date**: The date of the dataset in `YYYY-MM-DD` format (ISO-8601). *Scalar string*. **[required]**
- **author**: The lead author name, including their email. *Scalar string*. **[required]**
- **coauthors**: A list of co-authors, optionally each with their email. *Vector string*. **[optional]**
- **DOIs**: A list of DOI, which can be related to this dataset, relevant papers, or code. *Vector string*. **[optional]**
- **depends**: A list of datasets that this dataset depends on. *Vector string*. **[optional]**
- **description**: A sentence or two describing the table. *Scalar string*. **[recommended]**
- **comments**: A list of comments or interesting facts about the data. *Vector string*. **[optional]**
- **license**: The license for the table. *Scalar string*. **[recommended]**
- **keywords** A list of key word tags to enrich the linking and association of this table. *Vector string*. **[optional]**
- **fields**: A list of fields in the dataset, each with the following attributes: **[required]**
  - **name**: The name of the field. *Scalar string*. **[required]**
  - **unit**: The unit of measurement for the field (if applicable). *Scalar string*. **[recommended]**
  - **info**: A short description of the field. *Scalar string*. **[recommended]**
  - **ucd**: Unified Content Descriptor for IVOA (can have many). *Vector string*. **[recommended]**
  - **data_type**: The data type of the field (e.g., `int32`, `string`, `bool`, `double`). *Scalar string*. **[required]**
  - **array_size**: Maximum length of character strings. *Scalar integer* or *Scalar string*. **[optional]**
  - **qc**: Quality control check values (min, max, miss). *Vector string*. **[optional]**

This metadata format can be used to document datasets in a standardised way, making it easier to understand and share data within the research community. By following this format, you ensure that all relevant information about the dataset is captured and easily accessible (for both machines and humans). This format contains the superset of metadata requirements for IVOA, Data Central and astronomy surveys.

A note on the *qc* field entries, these should reflect expectations for the column data held, rather than just what is there. As an example we might expects a position angle to be bounded between 0 and 180 degrees, so it is more useful to specify those limits. Basically, the *qc* entries should be used by a later validator to check the internal consistency of the data provided (and potentially catch data corruption issues). The missing value entry should usually be something sensible like NA or Null (depending on data formats), but could also be a string or integer (-999) if that is the only option for the format being used (some types of **FITS** and **CSV** files, for instance).

If producing a maximal **MAML** then the metadata can be considered a **MAML**-Whale, and if only containing the required minimum entries it would be a **MAML**-Mouse. Between these two extremes you can choose your mammal of interest to reflect the quality/quantity of metadata. The sweet spot is obviously a **MAML**-Honey-Badger.

### MAML Schema Example

The basic **YAML** schema of the **MAML** format looks like the following:

```yaml
survey: Optional survey name
dataset: Recommended dataset name
table: Required table name
version: Required version (string, integer, or float)
date: Required date in YYYY-MM-DD format (ISO-8601)
author: Required lead author name and <email>
coauthors:
  - Optional coauthor name and optionally <email>
  - ...
DOIs:
  - Optional DOI string
  - ...
depends:
  - Optional dataset dependency
  - ...
description: Recommended short description of the table
comments:
  - Optional comment or interesting fact
  - ...
license: Recommended license for the data
keywords:
  - Optional keyword tag
  - ...
fields:
  - name: Required field name
    unit: Recommended unit of measurement
    info: Recommended short description
    ucd:
      - Recommended UCD string
      - ...
    data_type: Required data type (e.g., int32, string, bool, double)
    array_size: Optional max length of string (integer or string)
    qc:
      - min: minimum expected data value (optional)
      - max: maximum expected data value (optional)
      - miss: missing value flag (optional)
  - name: Another field name
    ...
```

Various legal example **MAML**s are included in this repo, all based on the example.parquet table (e.g. example_default.maml etc).

### Formal JSON Schema Draft

More formally, we can represent it using the json schema outline standard. Note this is not what the file should look like when saved (that should look like the above **YAML** mark-up), this is really a formal way to encode the schema for validation etc. So if youa re making a **MAML**, then focus on the above example, but if you want to strictly validate it, the below is useful.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Dataset Metadata Schema",
  "type": "object",
  "properties": {
    "survey": {
      "type": "string",
      "description": "Optional survey name"
    },
    "dataset": {
      "type": "string",
      "description": "Recommended dataset name"
    },
    "table": {
      "type": "string",
      "description": "Required table name"
    },
    "version": {
      "type": ["integer", "string", "number"],
      "description": "Required version (string, integer, or float)"
    },
    "date": {
      "type": "string",
      "format": "date",
      "description": "Required date in YYYY-MM-DD format (ISO-8601)"
    },
    "author": {
      "type": "string",
      "description": "Required lead author name and <email>"
    },
    "coauthors": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional coauthor name and optionally <email>"
    },
    "DOIs": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional DOI string"
    },
    "depends": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional dataset dependency"
    },
    "description": {
      "type": "string",
      "description": "Recommended short description of the table"
    },
    "comments": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional comment or interesting fact"
    },
    "license": {
      "type": "string",
      "description": "Recommended license for the data"
    },
    "keywords": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Optional keyword tag"
    },
    "fields": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["name", "data_type"],
        "properties": {
          "name": {
            "type": "string",
            "description": "Required field name"
          },
          "unit": {
            "type": ["string", "null"],
            "description": "Recommended unit of measurement"
          },
          "info": {
            "type": ["string", "null"],
            "description": "Recommended short description"
          },
          "ucd": {
            "type": ["array", "null"],
            "items": { "type": ["string", "null"] },
            "description": "Recommended UCD string"
          },
          "data_type": {
            "type": "string",
            "description": "Required data type (e.g., int32, string, bool, double)"
          },
          "array_size": {
            "type": ["integer", "string", "null"],
            "description": "Optional max length of string"
          },
          "qc": {
            "type": ["object", "null"],
            "properties": {
              "min": { "type": ["number", "string", "null"] },
              "max": { "type": ["number", "string", "null"] },
              "miss": { "type": ["number", "string", "null"], }
            },
            "description": "Optional quality control parameters"
          }
        }
      }
    }
  },
  "required": ["table", "version", "date", "author", "fields"],
  "additionalProperties": false
}
```

The contents of the above is also in the MAML_schema.json file included in this repo.

## Conclusions

**MAML** is fab, you should consider using it for your survey/project.

### Credit

The format was developed with input from the following people:

- **Aaron Robotham** (original specification, R implementation)
- **Sabine Bellstedt** (original Python implementation)
- **Trystan Lambert** (Python implementation and Validation)
- **Dior Etherton** (content suggestions)
- **Jochen Liske** (content suggestions)

### Logo

People always ask for logos, here's a logo:

<img src="MAML_logo.png" alt="MAML-Lion" width="400" style="display: block; margin: 0 auto;"/>
