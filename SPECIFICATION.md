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

The distinction between an incentive and a program is often unclear. There are many incentives that are not part of a distinct program. When there is no single name from a program, default to using the 'authority' plus a description of the product category, such as 'Georgia Power Home Appliance Rebates'. Further clarification can be provided by asking in our discussion forum.

Several incentives may fall under the umbrella of a single "program". Some authorities have a broad grouping of incentives for home efficiency, and include insulation, air sealing, and other weatherization improvements as part of the same program. However, each of those measures may have different incentive amounts, with different amount structures.

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

| Value      | Description                                                                                                                                                              |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| FlatAmount | A static amount, denominated in dollars. Example: $250                                                                                                                   |
| Normalized | A dollar amount that scales linearly with some property of the measure, such as capacity or size. Example: $1,000 per ton (of heating/cooling capacity) for a heat pump. |
| Percent    | An amount defined as a percentage of the cost of the measure. Example: 50% of the cost of a heat pump water heater.                                                      |

### applicant

Primary applicant who can apply for the incentive. There MAY be an option in the incentive for multiple applicants to be the primary applicant. 

| Value            | Description                                                                   |
| -----------------| ------------------------------------------------------------------------------|
| Contractor       | A licensed contractor.                                                        |
| Building Owner   | The legal owner of the building where the incentive-related work will happen. |
| Tenant           | An individual renting a property.                                             |

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

NODE datasets do not include information on the physical land areas that the various types of geography record represent. Where possible, this specification prescribes a way to refer to a land area unambiguously, so that it is practical to find the corresponding geographic data from some other source.

| Value                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| State                   | One of the US states or territories, or the District of Columbia. The `identifier` field MUST contain two uppercase letters that form one of the [United States Postal Service's abbreviations](https://pe.usps.com/text/pub28/28apb.htm).                                                                                                                                                                                                                                                                                                                                                                                |
| County                  | A county or county equivalent. The `identifier` field MUST be five decimal digits, forming the county or county equivalent's [FIPS code](https://transition.fcc.gov/oet/info/maps/census/fips/fips.txt).                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Municipality            | A city, town, or similar. The `identifier` field SHOULD be the name by which the municipality is commonly known. There are many different types of municipality across the country, so guidance here is not comprehensive. The `identifier` SHOULD NOT have affixes like "City of ...", "... City", and similar, except in the rare cases when such words are genuinely part of the municipality's common name (e.g. Kansas City, MO). _Note_: consolidated city-counties like Denver, CO and San Francisco, CA SHOULD be coded as `County` instead of this.                                                              |
| UtilityServiceTerritory | The service territory of a utility. When an incentive is associated with a geography of this type, it should be taken to mean that the claimant must be a customer of this utility, rather than that they must live within a specific land area; this is especially relevant when different utilities' territories overlap. The `identifier` field SHOULD be an identifier from one of the Energy Information Administration's survey forms; e.g. for electric utilities, it SHOULD be the "entity ID" / "Utility Number" from [Form EIA-861](https://www.eia.gov/electricity/data/eia861/), consisting of 2 to 6 digits. |

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

| Value        | Description          |
| ------------ | -------------------- |
| SingleFamily | 1 to 4 family houses |
| Multifamily  | 5+ family houses     |

### range_reason

| Value                   | Description                                                                                                                                                                                                                       |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Income                  | Amount depends on whether the applicant's income is above or below some threshold.                                                                                                                                                |
| InstallationLocation    | Amount depends on the location where the measure is installed. This refers to location on a geographic level, not location within a building or property.                                                                         |
| InstallerQualifications | Amount depends on some property of the installer of the measure, such as licensing, membership in some network or association, etc.                                                                                               |
| MeasureSpecifications   | Amount depends on some metric or specification of the measure. Example: a heat pump incentive that is greater if the heat pump's SEER and HSPF ratings exceed some thresholds. Note that cost is _not_ included in this category. |
| Membership              | Amount depends on whether the applicant is a member of some association or program.                                                                                                                                               |
| Other                   | Amount depends on some factor not listed here.                                                                                                                                                                                    |
| PurchaseLocation        | Amount depends on where a measure was purchased. Example: an EV incentive that requires the vehicle to be purchased within a specific city.                                                                                       |
| ReplacementType         | Amount depends on what the measure is replacing. Example: a heat pump incentive that is greater if the heat pump is replacing gas-based heating.                                                                                  |
| Utility                 | Amount depends on the identity of the applicant's utility provider.                                                                                                                                                               |
| Vendor                  | Amount depends on whether the measure is bought from a specific vendor or distributor, or set of same. Example: an EV incentive that requires the vehicle to be purchased from a specific network of dealerships.                 |

### status

| Value         | Description                                                                                                                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Active        | The incentive can be claimed now.                                                                                                                                                                                                                      |
| InDevelopment | The incentive has never been claimable, but is expected to become claimable in future.                                                                                                                                                                 |
| OnHold        | The incentive is not currently claimable, but it was previously, and is expected to become so again in future.                                                                                                                                         |
| Retired       | The incentive was previously claimable, but it currently is not, and is _not_ expected to become so in future. **Note**: in general, incentives in this situation MAY simply be removed from datasets instead of being included and listed as Retired. |

### technology

There are a very large number of possible technologies an incentive can be for. We expect to add entries to this enum on an ongoing basis, to more accurately express incentives with finer distinctions in what they are for.

**The groupings below are for clarity in documentation only**; they are not a feature of the schema or of datasets.

#### Weatherization

| Value                | Description                                                                                                                                                                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| AtticInsulation      | Insulation in the attic or roof — generally, anything above the building's conditioned space. Includes attic kneewall insulation.                                                                                                                            |
| BasementInsulation   | Insulation of an unconditioned basement. A basement is generally distinguished from a crawlspace by having enough vertical space for a person to stand up.                                                                                                   |
| CrawlspaceInsulation | Insulation of a crawlspace, including side walls.                                                                                                                                                                                                            |
| DoorReplacement      | Replacement of exterior doors to improve the building envelope.                                                                                                                                                                                              |
| DuctInsulation       | Insulation around HVAC ducts. Distinct from duct sealing in that it aims to reduce heat transfer through the body of the ductwork, rather than air leakage.                                                                                                  |
| DuctReplacement      | Replacement of HVAC ducts. **Does not include** duct sealing or duct insulation.                                                                                                                                                                             |
| DuctSealing          | Sealing of HVAC ducts to reduce air leakage. **Does not include** duct insulation.                                                                                                                                                                           |
| FloorInsulation      | Insulation under the lowest floor of conditioned space. May overlap, in practice, with basement or crawlspace insulation. Incentives for these types of insulation are generally coded in line with how they're described in the original program materials. |
| GeneralAirSealing    | Measures to reduce air leakage from the building envelope. Can include weather stripping of doors and windows.                                                                                                                                               |
| OtherInsulation      | Any type of insulation that doesn't fall into one of the other categories.                                                                                                                                                                                   |
| WallInsulation       | Insulation in vertical walls of conditioned space. **Does not include** sidewalls of unconditioned basements, or attic kneewalls.                                                                                                                            |
| Windows              | Replacement of exterior windows to improve the building envelope.                                                                                                                                                                                            |

#### Cooking

| Value                     | Description                                                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| ElectricCookingAppliance  | A cooktop or range that uses electric resistance to produce heat. Includes permanently-installed appliances only, and excludes portable hotplates. |
| InductionCookingAppliance | A cooktop or range that uses induction to produce heat. Includes permanently-installed appliances only, and excludes portable hotplates.           |

#### Dryers

| Value                          | Description                                                                               |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| ElectricResistanceLaundryDryer | A clothes dryer that uses electric resistance to produce heat.                            |
| HeatPumpLaundryDryer           | A clothes dryer that uses a heat pump to heat and dry clothes. May be vented or ventless. |

#### Water Heaters

| Value                         | Description                                                                                                                                                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ElectricResistanceWaterHeater | An appliance that heats domestic hot water using electric resistance.                                                                                                                                            |
| HeatPumpWaterHeater           | An appliance that heats domestic hot water using a heat pump. **Does not include** applications where the produced hot water is used for space heating. Can include electric-resistance heating coils as backup. |
| SolarWaterHeater              | An appliance that uses solar energy to heat water directly. Distinct from photovoltaic solar panels in that no electricity is produced from solar energy.                                                        |

#### Transportation

| Value                           | Description                                                                                                                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ElectricBike                    | A bicycle with an electric motor. May be pedal-assist or throttle-controlled. Distinct from an electric scooter or motorcycle in that it can be pedaled.                                                                                               |
| ElectricVehicleCharger          | A specialized electrical outlet that can charge an electric vehicle faster than a standard household outlet. Also known as a Level 2 or Level 3 charger. May include "smart" or "connected" functionality for demand response, though not necessarily. |
| NewBatteryElectricVehicle       | A new vehicle whose only energy input is electricity.                                                                                                                                                                                                  |
| NewPluginHybridElectricVehicle  | A new vehicle that can take both gasoline and electricity as energy input. Note that this **does not include** hybrids that cannot be plugged in.                                                                                                      |
| UsedBatteryElectricVehicle      | A used vehicle whose only energy input is electricity.                                                                                                                                                                                                 |
| UsedPluginHybridElectricVehicle | A used vehicle that can take both gasoline and electricity as energy input. Note that this **does not include** hybrids that cannot be plugged in.                                                                                                     |

#### Heating, ventilation, and air conditioning

| Value                      | Description                                                                                                                                                                                                                                                                                             |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GroundSourceHeatPump       | Any ground-source heat pump. Can include water-source heat pumps as well. These systems generally support both heating and cooling, and may produce domestic hot water as well. This category does not distinguish the means by which the system delivers heating and cooling to the conditioned space. |
| DuctedAirSourceHeatPump    | An air-source heat pump that delivers heated or cooled air through ductwork, from a centralized air handler. Includes both split and packaged systems.                                                                                                                                                  |
| DuctlessAirSourceHeatPump  | An air-source heat pump that delivers heated or cooled refrigerant to individual air handlers, with no ductwork. Also commonly known as "mini-splits".                                                                                                                                                  |
| ElectricThermalStorageSlab | A device that stores heat in an insulated mass, and releases it over time.                                                                                                                                                                                                                              |
| EvaporativeCooler          | A device that cools dry air by adding humidity. Also commonly known as a "swamp cooler".                                                                                                                                                                                                                |
| SmartThermostat            | A thermostat that is connected to the Internet. May or may not be required to support demand response (i.e. reducing heating or cooling load in response to a request from a utility).                                                                                                                  |
| WholeHouseFan              | A device that pulls in air from outside through open doors and windows, and exhausts it into an attic or outside.                                                                                                                                                                                       |

#### Electrical infrastructure

| Value                   | Description                                                                                                                                                                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BatteryEnergyStorage    | Batteries that can power at least some of the home's electrical circuits. May or may not be required to support demand response (i.e. charging or discharging in response to requests from the utility).                                                                                                |
| CriticalLoadPanels      | An electrical panel whose purpose is to separate loads that are to be fed by a backup source of electricity, such as a battery or generator.                                                                                                                                                            |
| ElectricalPanel         | The panel of circuit breakers that controls electric distribution in the building. Can include both main panels and subpanels. Can include new and replacement panels. Can include upgrading electric service between the utility transformer and the building.                                         |
| ElectricalWiring        | Upgrade and replacement of wiring in the home, such as to support new appliances, or higher currents and voltages. Can also include replacement of non-code-compliant wiring such as knob-and-tube. **Does not include** upgrades of electric service between the utility transformer and the building. |
| SmartElectricalPanel    | An electrical panel that is connected to the Internet. May have the ability to coordinate power flow among solar photovoltaic panels, battery storage, and electrical loads. May have the ability to turn individual circuits on and off.                                                               |
| SolarPhotovoltaicPanels | Photovoltaic solar panels. Includes both rooftop and ground-mounted installations. **Does not include** solar thermal water heating.                                                                                                                                                                    |

#### Miscellaneous

| Value                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ElectricOutdoorEquipment     | Lawnmowers (both riding and push), leaf blowers, trimmers, chainsaws, etc. Individual incentives may be for specific products only.                                                                                                                                                                                                                                                                                                                                        |
| ElectricPoolHeater           | A device that heats swimming pool water using electricity.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| EnergyEfficiencyImprovements | A comprehensive service to improve a home's energy efficiency. Usually involves an energy audit, weatherization measures, and possibly HVAC and appliance upgrades. Incentive amount may depend on measured or modeled effects of the upgrades. Incentive SHOULD be coded as this technology, instead of as the service's constituent upgrades, if it is offered as a single packaged service/experience, and customers can't claim the incentive for individual upgrades. |
| HomeEnergyAudit              | A professional assessment of a home's energy efficiency and usage. **Does not include** virtual consultations — an auditor must physically visit the home. **Does not include** actually making energy efficiency improvements; incentives that combine an audit and improvement work may be better expressed as `EnergyEfficiencyImprovements`.                                                                                                                           |
| Other                        | Any product or service not covered by another category of this enum. Note that the existence of an incentive for a specific kind of product, using this `technology` value, does not imply that a dataset tracks more incentives for that kind of product.                                                                                                                                                                                                                 |

## Descriptive Text Guidelines

When writing text for the `description` fields of programs and incentives, these guidelines apply:

- The text MUST be in English.

- The text MUST be suitable for display to an end user, with correct spelling and punctuation according to US English conventions.

- The text SHOULD be kept as terse as possible. To that end, the text MAY be in sentence fragments, such as "Must be enrolled in utility demand-response program."

- It is RECOMMENDED that program descriptions not attempt to summarize the individual incentives within the program, and instead describe the common features of the program's incentives. Example: "Rebates for heating and cooling upgrades in single-family homes."

- Program descriptions SHOULD NOT include information about the authority offering the program.

- It is RECOMMENDED that incentive descriptions include anything important about the incentive that is not encoded in other fields of the incentive, and that they _not_ simply restate information that is encoded in other fields.

  For example, "$100 rebate for a water heater" conveys no information beyond what already exists in the other fields, and is not recommended. "Must be replacing electric resistance heating" is not encoded in other fields, and is good information to include in the description.

- Incentive descriptions SHOULD NOT include information about the program they are part of, or the authority offering them.

- Monetary amounts MUST be formatted with a comma (U+002C) as the thousands separator, as in `$12,345`.

- The text MUST NOT use mathematical symbols like `<` or `≤`. Such inequality relationships SHOULD be expressed in words, as follows:

  - `<`: under
  - `≤`: up to
  - `>`: over
  - `≥`: at least

- Units of measurement SHOULD be abbreviated, as follows:

  - `kWh` for kilowatt-hours
  - `kW` for kilowatts
  - `sqft` for square feet
  - `Btu` for British Thermal Units, and `Btu/h` for BTU per hour. (Note that tons of cooling/heating SHOULD NOT be converted into BTU per hour.)

- It is RECOMMENDED that the text be targeted at a non-specialist audience, and avoid excessive jargon and abbreviations.

  Some abbreviations SHOULD be used, including for units of measurement as described above, and for industry-standard metrics, such as SEER, HSPF, UEF, and COP.
