## Overview

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Concepts

### Incentive

The core concept of a NODE dataset is the incentive. An incentive is defined as an offering of _something of value_ conditioned on a customer receiving _one or more of a specific set of measures_.

#### Amount Structures

The amount of value a customer can get from an incentive can vary in several ways:

- It may depend on the cost of the measure(s) the incentive is for.

- It may depend on some aspect of the measure's size, such as the heating/cooling capacity of a heat pump, the square footage of insulation applied, the power output of a rooftop solar system, and so on.

- It may be a flat monetary amount that does not vary at all.

- It may not be a monetary amount at all; products or services may simply be provided for free.

Variable amounts may have floor or ceiling amounts, such as "30% of project cost, up to $1,000".

Incentives can combine these structures in various ways; for example, "$500, plus $100 per ton" for a heat pump.

#### Geographies

Most incentives condition eligibility on location in some way — typically the location where the measure is installed, or where the applicant lives.

### Program

Several incentives may fall under the umbrella of a single "program". TODO

The distinction between an incentive and a program is often unclear. There are many incentives that are not part of a distinct program. TODO

Take, for example, the federal [Energy Efficient Home Improvement Credit](https://www.irs.gov/credits-deductions/energy-efficient-home-improvement-credit) (25C). It is presented as a single tax credit, but it applies to several different home efficiency measures, with different amounts of money available for each. Each of those amounts is an "incentive", in our terminology, and the 25C credit as a whole is a "program".

### Authority

An "authority" is any government agency, or non-government organization, that offers incentive programs. This can include agencies at all levels of government (federal, state, tribal, county, municipal), utility companies (investor-owned or publicly-owned), nonprofit organizations, and non-utility businesses.

There are several roles that an authority may have with respect to an incentive program:

- "Front office" administration: publishing information, receiving claims, providing customer support, etc.
- "Back office" administration: processing and deciding claims, keeping and reporting statistics, paying out funds, etc.
- Setting the program's rules
- Providing funding

## Dataset Structure

A NODE dataset is a set of tables. Each table is represented in a separate Comma-Separated Values (CSV) file.

The CSV files MUST be UTF-8 encoded, use the comma (U+002C) as the field separator, newline (U+000A) as the line separator, and the double quote (U+0022) as the quote character.

The first line in each file is the header. The header MUST contain every field marked "**Required**" in the field definitions below. The header MAY contain any of the fields marked "Optional" below. All fields in the header MUST be unique.

Every file requires an `id` field. The values of this field MUST be unique within each file.

| File name         | Description                                                                                                                                        |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `amounts.csv`     | Individual components of incentives' amount structures.                                                                                            |
| `authorities.csv` | Organizations or entities that offer the incentives in this dataset.                                                                               |
| `geographies.csv` | Geographic areas in which incentives are, or are not, available.                                                                                   |
| `incentives.csv`  | Incentives for electrification measures. An incentive is generally a specific monetary amount structure, applicable to a specific set of measures. |
| `programs.csv`    | Groups of related incentives.                                                                                                                      |

### Data Types

Every field in every file has an associated data type: a constraint on what values may appear in it. This section describes the meanings of these types. Individual fields MAY impose further constraints on allowable values.

| Data Type                  | Description                                                                                                                                                                                                             |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ID                         | An identifier for the record. It MUST be unique within the file where it appears.                                                                                                                                       |
| ID (_file-name_)           | The ID of a record within the named file, within the same dataset. This ID MUST appear in the `id` field of exactly one row in the other file.                                                                          |
| string                     | Any text.                                                                                                                                                                                                               |
| URL                        | A URL according to [RFC 1738](https://datatracker.ietf.org/doc/html/rfc1738). The scheme SHOULD be `https`, and otherwise MUST be `http`.                                                                               |
| number                     | A numeric value, expressed in decimal notation. The number of allowed decimal places may vary. Constraints on the value, such as upper and lower bounds, may vary.                                                      |
| boolean                    | Either `0` (meaning false) or `1` (meaning true).                                                                                                                                                                       |
| enum (_enum-name_)         | One of an enumerated set of values, as described in the named subsection of the [Enum Definitions](#enum-definitions) section of this document.                                                                         |
| list of enum (_enum-name_) | A comma-separated list of values from the named enumerated set of values.                                                                                                                                               |
| range                      | Two decimal numeric values, separated by a comma. The first value MUST be numerically less than or equal to the second. The number of allowed decimal places, and constraints such as upper and lower bounds, may vary. |
| date                       | A "full-date" as defined by [RFC 3339 § 5.6](https://www.rfc-editor.org/rfc/rfc3339#section-5.6), such as `2024-06-30`.                                                                                                 |

## Field Definitions

### amounts.csv

There MAY be multiple records in this file that refer to the same incentive in `incentives`. When there is more than one amount row for a single incentive, it means that the two amount rows together define the total of the incentive, not that they are two alternatives. 
Example of multiple amount rows:
- situations where there is a base incentive plus a variable amount, such as $500 plus $1.50 per square foot for insulation

| Field                | Type                      | Presence     | Description                                                                                                                                                                                                                                                                                       |
| -------------------- | ------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                 | ID                        | **Required** | The unique ID of this record.                                                                                                                                                                                                                                                                     |
| `incentive_id`       | ID (incentives.csv)       | **Required** | The ID of the incentive this amount structure pertains to.                                                                                                                                                                                                                                        |
| `type`               | enum (amount_type)        | **Required** | The structure of this amount: what it depends on.                                                                                                                                                                                                                                                 |
| `normalization_unit` | enum (normalization_unit) | Optional     | The unit of the quantity the amount depends on. MUST be present for amounts of type `Normalized`, and MUST NOT be present otherwise.                                                                                                                                                              |
| `value`              | range                     | **Required** | If `type` is `FlatAmount`, a range of dollar value this amount structure may represent. If `type` is `Percent`, a range of numbers between 0 and 1, signifying a multiplier on the measure's cost. If `type` is `Normalized`, a range of dollar value per the unit named in `normalization_unit`. |
| `range_reason`       | enum (range_reason)       | Optional     | The property of the measure, or the customer, that determines where in the range specified by `value` the incentive's value falls. This field MUST be present if the two parts of `value` are different, and MUST NOT be present otherwise.                                                       |
| `floor`              | range                     | Optional     | The range of the least possible value this amount structure could be worth. Expressed in dollars. If the `type` is `FlatAmount`, this field MUST NOT be present.                                                                                                                                  |
| `ceiling`            | range                     | Optional     | The range of the greatest possible value this amount structure could be worth. Expressed in dollars. If the `type` is `FlatAmount`, this field MUST NOT be present.                                                                                                                               |
| `income_qualified`   | boolean                   | **Required** | Whether this amount is only available to customers whose income meets certain requirements.                                                                                                                                                                                                       |

## authorities.csv

| Field          | Type                   | Presence     | Description                                                                                                                                                                                                                                                                                                                                                                            |
| -------------- | ---------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`           | ID                     | **Required** | The unique ID of this authority.                                                                                                                                                                                                                                                                                                                                                       |
| `name`         | string                 | **Required** | The full name of the authority. This SHOULD NOT be an abbreviation unless the authority is near-universally known by an abbreviated form, or the abbreviation has no un-abbreviated equivalent. This SHOULD be the name by which the general public would be most likely to know the authority; official legal names that differ from the "doing business as" name SHOULD NOT be used. |
| `abbreviation` | string                 | Optional     | Any abbreviation by which the authority is commonly known.                                                                                                                                                                                                                                                                                                                             |
| `level`        | enum (authority_level) | **Required** | The nature of the authority: what level of government it belongs to, or what kind of organization it is.                                                                                                                                                                                                                                                                               |

### geographies.csv

There MAY be multiple records in this file that refer to the same incentive in `incentives.csv`. That signifies that the incentive is available if the installation location (or customer's residence, or whatever is relevant) is in _any_ of the areas for which `include` is true, _and not_ in any of the areas for which `include` is false.

| Field          | Type                  | Presence     | Description                                                                                                                                          |
| -------------- | --------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`           | ID                    | **Required** | The unique ID of this record.                                                                                                                        |
| `incentive_id` | ID (incentives.csv)   | **Required** | The ID of the incentive this geography pertains to.                                                                                                  |
| `type`         | enum (geography_type) | **Required** | The type of geographic area this record specifies.                                                                                                   |
| `identifier`   | string                | **Required** | Which geographic area this record specifies. The requirements for this field depend on the value of `type`; see the enum definition for full detail. |
| `include`      | boolean               | **Required** | Whether this geographic area is included in, or excluded from, the area in which this geography's incentive may be claimed.                          |

### incentives.csv

| Field               | Type                         | Presence     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------- | ---------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`                | ID                           | **Required** | The unique ID of this record.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `program_id`        | ID (programs.csv)            | **Required** | The ID of the program this incentive belongs to.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `mechanisms`        | list of enum (mechanism)     | **Required** | The ways in which a customer receives value from this incentive.                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `property_types`    | list of enum (property_type) | **Required** | What types of real property the measure can be applied to.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `description`       | string                       | Optional     | A short description of what the incentive consists of. See [Descriptive Text Guidelines](#descriptive-text-guidelines) for more guidance.                                                                                                                                                                                                                                                                                                                                                                     |
| `source_url`        | URL                          | Optional     | A URL to the best available official source of information on the incentive. The URL SHOULD point to a webpage, rather than a PDF; however, a PDF is acceptable if it is the only public source of details on the incentive. The URL SHOULD point to a document in which information on the incentive is visible with no additional navigation; however, this is not always possible. This field SHOULD be left blank if the best available URL is the same as the `source_url` of the corresponding program. |
| `technologies`      | list of enum (technology)    | **Required** | Which technology or technologies the incentive applies to.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `status`            | enum (status)                | **Required** | The status of the incentive, specifically addressing whether customers can currently claim it.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `primary_applicant` | enum (applicant)             | Optional     | Who should apply for the incentive, in terms of their relationship to the property where the measure is installed.                                                                                                                                                                                                                                                                                                                                                                                            |
| `income_qualified`  | boolean                      | **Required** | Whether the incentive is only available to customers whose income meets certain requirements.                                                                                                                                                                                                                                                                                                                                                                                                                 |

### programs.csv

| Field              | Type                 | Presence     | Description                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------ | -------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`               | ID                   | **Required** | The unique ID of this record.                                                                                                                                                                                                                                                                                                                                                |
| `administrator_id` | ID (authorities.csv) | **Required** | The ID of the authority that administers this program; specifically, the authority with which claimants of this program's incentives will interact.                                                                                                                                                                                                                          |
| `name`             | string               | **Required** | The customer-facing name of the program. Not all programs have a distinct customer-facing brand, and in such cases the program name SHOULD be synthesized as a combination of the authority's name and the general focus of the program's incentives. E.g. "XYZ Energy Residential Rebates".                                                                                 |
| `source_url`       | URL                  | **Required** | A URL to the best available official source of information on the program. The URL SHOULD point to a webpage, rather than a PDF; however, a PDF is acceptable if is the only public source of details on the program. The URL SHOULD point to a document in which information on the program is visible with no additional navigation; however, this is not always possible. |
| `description`      | string               | Optional     | A short description of the nature of the program. See [Descriptive Text Guidelines](#descriptive-text-guidelines) for further guidance.                                                                                                                                                                                                                                      |
| `budget`           | number               | Optional     |                                                                                                                                                                                                                                                                                                                                                                              |
| `end_date`         | date                 | Optional     |                                                                                                                                                                                                                                                                                                                                                                              |

## Enum Definitions

### amount_type

TODO

### applicant

TODO

### authority_level

| Value                     | Description                                                                     |
| ------------------------- | ------------------------------------------------------------------------------- |
| FederalGovernment         | An agency of the United States federal government.                              |
| StateGovernment           | An agency of the government of a state, territory, or the District of Columbia. |
| LocalGovernment           |                                                                                 |
| CityGovernment            |                                                                                 |
| UtilityServiceProvider    | A utility company, whether investor-owned or publicly-owned.                    |
| RegionalUtilityNetwork    |                                                                                 |
| CommunityChoiceAggregator |                                                                                 |
| NonprofitOrganization     |                                                                                 |
| Bank                      |                                                                                 |
| BusinessOrganization      |                                                                                 |

### geography_type

| Value                   | Description                                                                                                                                                                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| State                   | One of the US states or territories, or the District of Columbia. The `identifier` field MUST contain two uppercase letters that form one of the [United States Postal Service's abbreviations](https://pe.usps.com/text/pub28/28apb.htm). |
| County                  | A county or county equivalent. The `identifier` field MUST be five decimal digits, forming the county or county equivalent's [FIPS code](https://transition.fcc.gov/oet/info/maps/census/fips/fips.txt).                                   |
| Municipality            | TODO                                                                                                                                                                                                                                       |
| UtilityServiceTerritory | The service territory of a utility. TODO                                                                                                                                                                                                   |

### mechanism

| Value           | Description                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Rebate          | The customer receives money back after purchasing an item, in the form of a check or electronic payment.                  |
| TaxCredit       | The customer receives a credit against their taxes owed.                                                                  |
| AccountCredit   | The customer receives money back after purchasing an item, in the form of a credit against their utility billing account. |
| UpfrontDiscount | The customer receives a discount when purchasing an item, and does not pay the full amount.                               |
| NoCost          | The customer receives products or services for free.                                                                      |
| Financing       | The customer receives money that they must pay back.                                                                      |

### normalization_unit

| Value           | Description                                                                                                                                                                                                            |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CFM50           | A measure of a building's airtightness (cubic feet per minute to create a pressure difference of 50 pascals). Used in weatherization incentives.                                                                       |
| EquipmentUnit   | A single unit of equipment, especially for types of equipment that can involve multiple similar units as part of a single installation (e.g. indoor units of a mini-split heat pump). Usually seen in HVAC incentives. |
| Kilowatt        | A kilowatt of power output, such as from photovoltaic solar panels.                                                                                                                                                    |
| KilowattHour    | A kilowatt-hour of energy, usually associated with battery storage capacity.                                                                                                                                           |
| SquareFoot      | A square foot, usually of insulation.                                                                                                                                                                                  |
| TenThousandBtuH | 10,000 BTU per hour, a measure of heating or cooling capacity. Not to be confused with a ton (see below).                                                                                                              |
| Ton             | A measure of heating or cooling capacity, equivalent to 12,000 BTU per hour.                                                                                                                                           |

### property_type

TODO

### range_reason

TODO

### status

| Value         | Description                                                                                                                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Active        | The incentive can be claimed now.                                                                                                                                                                                                                      |
| InDevelopment | The incentive has never been claimable, but is expected to become claimable in future.                                                                                                                                                                 |
| OnHold        | The incentive is not currently claimable, but it was previously, and is expected to become so again in future.                                                                                                                                         |
| Retired       | The incentive was previously claimable, but it currently is not, and is _not_ expected to become so in future. **Note**: in general, incentives in this situation MAY simply be removed from datasets instead of being included and listed as Retired. |

### technology

TODO

## Descriptive Text Guidelines

TODO
