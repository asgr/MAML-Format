## Table of Contents

1. [The MAML Metadata Format]
4. [MAML Format Example]
5. [MAML Version Control]
   - [Formal JSON Schema Definition]
   - [Adaptation]
6. [Conclusions]
   - [License]
   - [Credit]
   - [Logo]

# The MAML Metadata Format

**MAML** is a **YAML** based metadata format for tabular data (roughly implying Metadata **yAML**). When developing this format specification we were highly influenced by **VOTable** and **FITS** header formats, so there is a bias to this being astronomy-centric, but it should be functional for a wide variety of table types.

### Why MAML? 

We have **VOTable** and **FITS** header already?! Well, for various projects we were keen on a rich metadata format that was easy for humans and computers to both read and write. **VOTable** headers are very hard for humans to read and write (boo), and **FITS** is very restrictive with its formatting and only useful for **FITS** files directly. **FITS** in particular is also very open with its key-value liberty, which can be useful when writing metadata, but makes interpreting much later (when the intent of the authors is lost in time) difficult.

In comes **YAML**, a very human and machine readable and writeable format. By restricting ourselves to a narrow subset of the language we can easily describe fairly complex table metadata (including most common IVOA information). So this brings us to **MAML**: Metadata **yAML** (it kinda works :-P). By keeping metadata on strictly defined rails it makes reading easier, and removes interpretation from the intent. Say in **FITS** somebody wrote Date = '2025-03-08', but is that the 8<sup>th</sup> of March or the 3<sup>rd</sup> of August? In **MAML** we know it is the 8<sup>th</sup> of March because we enforce the ISO-8601 standard.

**MAML** format files should be saved as example.maml etc. And the idea is the **MAML** string can be inserted directly into a number of different file formats that accept key-value metadata (like Apache Arrow Parquet files). In the case of Parquet files they should be written to a 'maml' extension in the Arrow metadata section of the file (so something like parquet_file\$metadata\$maml in R world). See the schema of the example_maml.parquet file to see this in practice.

## MAML Metadata Format

The **MAML** metadata format is a structured way to describe datasets, surveys, and tables using **YAML**. This format ensures that all necessary information about the data is captured in a clear and organized manner.

Below we present an informal overview of the **MAML** format. A stricter machine readable **JSON** schema for strict validation is presented later. If in doubt, what **MAML** is and what it allows is definitively encoded in the formal **JSON** schema. Any tension with the informal description is an error, but the informal description should always be ignored in these cases.

### Informal Structure Overview MAML v1.0

The super set of allowed entries for **MAML** v1.0 is below. Not all are required, but if present they should obey the order and naming.

- **survey**: The name of the survey. *Scalar string*. **[optional]**
- **dataset**: The name of the dataset. *Scalar string*. **[recommended]**
- **table**: The name of the table. *Scalar string*. **[required]**
- **version**: The version of the table. *Scalar string, integer or float*. **[required]**
- **date**: The date of the dataset in `YYYY-MM-DD` format (ISO-8601). *Scalar string*. **[required]**
- **author**: The lead author name, including their email. *Scalar string*. **[required]**
- **coauthors**: A list of co-authors, optionally each with their email. *Scalar/Vector string*. **[optional]**
- **DOIs**: A list of DOI, which can be related to this table, each with the following attributes: **[optional]**
  - **DOI**: Valid DOI reference *Scalar string*. **[required]**
  - **type**: Type of DOI ('paper', 'software', 'data', etc) *Scalar string*. **[required]**
- **depends**: A list of other tables that this table depends on, each with the following attributes: **[optional]**
  - **survey**: The name of the dependent survey. *Scalar string*. **[optional]**
  - **dataset**: The name of the dependent dataset. *Scalar string*. **[optional]**
  - **table**: The name of the dependent table. *Scalar string*. **[required]**
  - **version**: The version of the dependent table. *Scalar string*. **[optional]**
- **description**: A sentence or two describing the table. *Scalar string*. **[recommended]**
- **comments**: A list of comments or interesting facts about the data. *Vector string*. **[optional]**
- **license**: The license for the table. *Scalar string*. **[recommended]**
- **keywords**: A list of key word tags to enrich the linking and association of this table. *Scalar/Vector string*. **[optional]**
- **MAML_version**: The version of the **MAML** schema being used (where this version is 1.0) *Scalar integer or float*. **[recommended]**
- **fields**: A list of fields in the dataset, each with the following attributes: **[required]**
  - **name**: The name of the field. *Scalar string*. **[required]**
  - **unit**: The unit of measurement for the field (if applicable). *Scalar string*. **[recommended]**
  - **info**: A short description of the field. *Scalar string*. **[recommended]**
  - **ucd**: Unified Content Descriptor for IVOA (can have many). *Scalar/Vector string*. **[recommended]**
  - **data_type**: The data type of the field (e.g., `int32`, `string`, `bool`, `double`). *Scalar string*. **[required]**
  - **array_size**: Maximum length of character strings. *Scalar integer* or *Scalar string*. **[optional]**
  - **qc**: Quality control check array (min-max-miss): **[optional]**
    - **min**: Minimum value expected in column data. *Scalar numeric* **[required]**
    - **max**: Maximum value expected in column data. *Scalar numeric* **[required]**
    - **miss**: Missing value value. *Scalar numeric/string* **[required]**

This metadata format can be used to document datasets in a standardised way, making it easier to understand and share data within the research community. By following this format, you ensure that all relevant information about the dataset is captured and easily accessible (for both machines and humans). This format contains the superset of metadata requirements for IVOA, Data Central and astronomy surveys.

A note on the *data_type* field entries. Some table formats have strictly well defined and self-describing data types for columns (FITS, Parquet) and it is almost never a good idea to supersede those. In these cases *data_type* when present is more like a validation entry because you might want to ensure the column data type has not been converted from what you expect as some point. *data_type* is critical for ASCII based tables (CSV etc) since it can be entirely ambiguous how you want an ASCII column to be interpreted, e.g. [1, 2, 3] could be integers, float16, float32 etc.

A note on the *qc* field entries, these should reflect expectations for the column data held, rather than just what is there. As an example we might expects a position angle to be bounded between 0 and 180 degrees, so it is more useful to specify those limits. Basically, the *qc* entries should be used by a later validator to check the internal consistency of the data provided (and potentially catch data corruption issues). The missing value entry should usually be something sensible like NA or Null (depending on data formats), but could also be a string or integer (-999) if that is the only option for the format being used (some types of **FITS** and **CSV** files, for instance).

If producing a maximal **MAML** then the metadata can be considered a **MAML**-Whale, and if only containing the required minimum entries it would be a **MAML**-Mouse. Between these two extremes you can choose your mammal of interest to reflect the quality/quantity of metadata. The sweet spot is obviously a **MAML**-Honey-Badger.

### Informal Structure Overview MAML v1.1

**MAML** v1.1 adds two additional fields: *keyarray* and *extra*. This makes it a less restricted format (which can be good and bad). If you do not require these fields then it is better you officially target **MAML** v1.0 since it allows for stricter validation. Naturally if using this extended format the *MAML_version* field should be 1.1.

- **keyarray**: A FITS style key-value-comment array: **[optional]**
  - **key**: Name of key *Scalar string*. **[required]**
  - **value**: Value/s of key *Scalar/vector string/number*. **[required]**
  - **comment**: Description of key *Scalar string*. **[required]**
- **extra**: Any type of legal YAML. **[optional]**

**keyarray** is similar in spirit to traditional FITS key-value-comment headers. The main difference is the value can be a scalar or a vector. This might be useful is the key is really a lot of similar quantities. The basic key-value-comment array structure is validated, so if this field can be used rather than **extra** it should be.

**extra** is a totally free-form field that can contain any legal **YAML**. This cannot be validated beyond checking the name of the field. Using this field is therefore "user beware". We would not expect **MAML** readers and code that interacts with it to necessarily even make use of this field since it is effectively impossible to predict the contents. Avoid using this field unless it really cannot be avoided!

## MAML Format Example

Imagine we have the following table:

```
 ID Name       Date  Flag   RA Dec  Mag
  1    A 2025-08-26  TRUE 45.1 3.5 20.5
  2    B 2025-07-22 FALSE 47.2 2.8 20.3
  3    C 2025-09-03  TRUE 43.1 1.2 15.2
  4    D 2025-06-13  TRUE 48.9 2.9 18.8
  5    E 2025-07-26 FALSE 45.5 1.8 22.1
```

A basic example of the **MAML** format for the above could like the following (not using all of the available fields):

```yaml
survey: The Big Survey
dataset: InputCatalogue
table: Input_Cat_North_03
version: 1.3
date: '2025-09-01'
author: Dave Smith <dave_smith_is_not_here@gmail.com>
coauthors:
- Joe Bloggs <joe_bloggs_is_not_here@gmail.com>
- Jane Doe <jane_doe_is_not_here@gmail.com>
DOIs:
- DOI: 10.1093/mnras/sty440
  type: paper
- DOI: 10.5281/zenodo.10059903
  type: software
depends:
- survey: The Medium Survey
  dataset: SpecZ
  table: Spec_field_01
  version: 3.7
- survey: The Tiny Survey
  dataset: Stars
  table: Phot_South
  version: 2
description: Just an example. Probably do not write tonnes here. A few sentences is usually about right.
license: MIT
MAML_version: 1.0
fields:
- name: ID
  unit:
  ucd:
  - meta.id
  - meta.main
  data_type: int32
- name: Name
  unit:
  ucd:
  data_type: string
- name: Date
  unit:
  ucd:
  - time
  - obs.exposure
  data_type: string
- name: Flag
  unit:
  ucd:
  data_type: boolean
- name: RA
  unit: deg
  ucd: pos.eq.ra
  data_type: float64
- name: Dec
  unit: deg
  ucd: pos.eq.dec
  data_type: float64
- name: Mag
  unit:
  ucd: phot.mag
  data_type: float64
```

Various legal example **MAML**s are included in this repo inside version folders, all based on the example.parquet table (e.g. example_default.maml etc).

## MAML Version Control

The normal path will be that minor version increments expand fields and make the standard more expansive and flexible, and major version increments change behaviour, fix weaker aspects of the format, and are not guaranteed to be backwards compatible. In general you should pick the major version via the features and style you prefer, and then attempt to choose the strictest (lowest valued) minor version that covers your use case. This will make the **MAML** useful for you, and the validation as strict as possible which should reduce errors.

### Formal JSON Schema Definition

The v1.X folders contain MAML_schema_v1pX.json files that strictly encode the specific version of **MAML** being targeted. Look at these to see a more machine rigorous definition of what constitutes a valid format.

Note, if you want to rigorously validate a **MAML** then you should explicitly encode the *MAML_version* being targeted. Different minor version **JSON** schema are included in the repo for backwards compatibility (so use MAML_schema_v1.0 to validate a **MAML** that claims to be using *MAML_version* 1.0). The README front page will contain the informal evolution of the standard via describing major and minor version changes. These **JSON** schema are used directly in the **R** **MAML** package **validate_MAML** function.

Why do we not require *MAML_version* in the **MAML**? On balance we recognised certain use cases where people are using a very small subset of **MAML** perhaps do not want to pollute their metadata with a *MAML_version* field. In these cases people can still informally benefit from using the well structured **MAML** metadata format (and the associated tools). 

### Adaptation

The aim is the format should always be backwards compatible within major versions, so a newer v1.X of the **JSON** schema validator should pass older v1.Y **MAML** schema as long as X <= Y. If and when **MAML** has a version v2.0 release, this will be because we are explicitly making changes that could make v1 **MAML**s invalid (but they might not, if a reduced set of **MAML** is being written).

Given this is released under an **MIT** license, it is of course valid to take **MAML** and expand the standard to your use case (e.g. adding new root fields), but in that situation you must be careful to encode an explicitly different *MAML_version*. The official **MAML** schema validator requires the version to be a number (fo float or integer). A modified **MAML** should change this to a string, and encode it such as '1.1_example' or similar, where the leading number refers to the version of **MAML** that was modified. Adapted **MAML**s will then always fail validation when using the official **JSON** schema validator, but it should at least be clear to users why. Note adapting **MAML** comes with the risk that third parties cannot parse your specific metadata schema, so in general I would warn against doing this unless you have very good reasons. This is not being protective, rather pragmatic and reducing needless fragmentation.

A more typical use case might be a survey implements exactly **MAML** v1.1, but also decides a specific list of *unit* to allow, or *ucd* IVOA version to use. This will not be formally required by the **JSON** schema validator, so in that case other quality control checks might be required to ensure the metadata adheres to additional strictness constraints. Creating a stricter subset of **MAML** should always mean the standard validation tools still work, but you may still want to adapt the **JSON** schema validator to reduce errors (e.g. explicitly remove fields you do not want people to be using etc).

## Conclusions

**MAML** is fab, you should consider using it for your survey/project.

### License

This is released and made available under the standard MIT license (see MIT_license file).

### Credit

The format was developed with input from the following people:

- **Aaron Robotham** (original specification, R implementation)
- **Sabine Bellstedt** (original Python implementation)
- **Trystan Lambert** (Python implementation and validation)
- **Dior Etherton** (content and format suggestions)
- **Jochen Liske** (content and format suggestions)
- **Mark Taylor** (content and format suggestions)

### Logo

People always ask for logos, here's a logo:

<img src="MAML_logo.png" alt="MAML-Lion" width="400" style="display: block; margin: 0 auto;"/>
