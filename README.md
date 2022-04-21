# Veterinary Medicinal Products Data Model

Specification v1.0.0

A description of a data model with reference dictionaries to standardise the reporting of medicinal products in veterinary publications. Fundamentally a simplified version of the data model published by NHS Prescription Services for their dictionary of medicines + devices (dm+d).

Produced as part of the VetText project to increase collaboration between researchers working with veterinary electronic health records.

## Table of Contents
* [Introduction](#introduction)
* [Requirements](#requirements)
* [Data Model](#data-model)
* [Examples](#examples)

## Introduction
Multiple groups internationally are trying to extract medicinal product information from veterinary electronic health records.

This data is inherently messy. Identifying actual medicinal products is a challenge outside the scope of this document. However, their description varies between publications once achieved, so comparing outputs between groups is hard.

An ongoing goal is to extract more detailed information about prescriptions. For this data to be useful, the products must be well described. As the tooling becomes more complex, there will be increasing time duplicating existing software.

### Purpose
1. To standardise the classification and description of medicinal products in veterinary publications.
2. To provide a data schema for software extracting information about medicinal products to reduce duplicated work.
3. To allow us to spend more time answering our core research questions.

### Scope
VetText can provide a reference data structure describing actual medicinal products for software and models that identify products to use as their output template. We can also provide reference dictionaries for standardised products and their active ingredients. Finally, we can host reference dictionaries for actual products in different geographies.

## Requirements
This dictionary is for researchers working in surveillance (and related epidemiological fields) and aims to match meaningfully equivalent products while providing consistent terminology for reporting.

Veterinarians are typically allowed to prescribe medicinal products authorised for use in any species, so the data model must be valid across veterinary and human domains. It also should be valid across the geographical and regulatory jurisdictions our VetText researchers work within.

The data model has three components.
1. Therapeutic Moieties (active ingredients) associate medicinal components with a standardised list of class and families to which they belong.
2. Virtual Products describe a meaningfully distinct product that may be available to prescribers. These need to capture the differentiating details about a product our researchers are interested in interrogating.
3. Actual Products are a realised version of a virtual product. These are the products that are invoiced and sold to clients.

We can provide reference dictionaries for active ingredients, virtual products, drug classes and families, medicinal forms, and routes of administration. Some degree of curation of the reference dictionaries is unavoidable. Still, the goal is to keep the information in the reference tables to the minimum needed to standardise reporting and leave other details for the Actual Products table.

The dictionaries will not be exhaustive. The data will be more useful if they are easy to parse manually. Hence, a design goal is to keep components in all dictionaries to the lowest clinical detail our researchers need.

Mapping to other computer systems is not a specific requirement or goal. However, we intend to use SNOMED CT core terms to identify Therapeutic Moieties and Virtual Products uniquely.

## Data Model

### Therapeutic Moiety
Therapeutic Moiety is the substance that a prescriber wishes to provide to a patient with therapeutic intent. It can be a single active ingredient (e.g. amoxicillin), multiple active ingredients where it makes conceptual sense to treat them as their own class (e.g. amoxicillin & clavulanic acid), or a combination of other Therapeutic Moieties (e.g. miconazole, polymyxin B & prednisolone).

| Attribute     | Type                  | Description
| ------------- | --------------------- | -----------
| `id`          | `int` <br>`SNOMED CT` | A unique identifier that may be linked to one or more Virtual Products
| `name`        | `string`              | English-language name
| `class`       | `foreign key`         | id for the Therapeutic Class
| `family`      | `foreign key`         | id for the Therapeutic Family
| `combination` | `boolean`             | Is this moiety a combination of others?<br>Default to FALSE. If TRUE, class and family must be blank

### Virtual Product
Virtual Product represents the properties of clinically equivalent Actual Products. It represents the abstract concept of a medicinal product before a prescriber had to select an individual product (or products) to dispense. Each Virtual Product is associated with a single Therapeutic Moiety (which may contain multiple active ingredients).

| Attribute     | Type                  | Description
| ------------- | --------------------- | -----------
| `id`          | `int` <br>`SNOMED CT` | A unique identifier that may be linked to one or more Actual Products
| `name`        | `string`              | English-language descriptive name
| `moiety`      | `foreign key`         | id for the Therapeutic Moiety
| `form`        | `foreign key`         | id for the Therapeutic Drug Form
| `route`       | `foreign key`         | id for the Drug Route

### Actual Product
Actual Products are items that appear on invoices and labels that are dispensed to a patient. They have a specified amount (strength) of the Virtual Product and may have legal or regulatory statuses depending on jurisdiction. We expect this data schema to be extended by researchers to include fields relevant to their country. For example, in the UK, we might add the legal status (POM-V, POM-VPS, NFA-VPS, AVM-GSL, etc) or the controlled drug status (Schedule 1–5 or no status).

| Attribute     | Type                  | Description
| ------------- | --------------------- | -----------
| `id`          | `int` <br>`SNOMED CT` | A unique identifier
| `name`        | `string`              | The name as authorised by the relevant regulatory body
| `product`     | `foreign key`         | id for the Virtual Product
| `SVN`         | `float`               | Strength value numerator<br>25 in “250mg per 5ml”<br>500 in “500mg tablets”
| `SVNU`        | `foreign key`         | Strength value numerator unit<br>mg in “250mg per 5ml”<br>mg in “500mg tablets”
| `SVD`         | `float`               | Strength value denominator<br>5 in “250mg per 5ml”<br>Omit in “500mg tablets”
| `SVDU`        | `foreign key`         | Strength value denominator unit<br>ml in “250mg per 5ml”<br>Omit in “500mg tablets”

## Examples

### Actual Products

| id     | name                                       | product | SVN | SVNU | SVD | SVDU
|:------:| ------------------------------------------ |:-------:|:---:|:----:|:---:|:----:
| `3210` | Synulox Palatable Tablets 250 mg           | `7399`  | 250 | `mg` | •   | •
| `3211` | Clavubactin 500/125 mg Tablets for Dogs    | `7399`  | 625 | `mg` | •   | •
| `3354` | Surolan Ear Drops and Cutaneous Suspension | `7132`  | •   | •    | •   | •
| `3682` | Baytril 25 mg/ml Solution for Injection    | `7464`  | 25  | `mg` | 1   | `ml`

### Virtual Products

| id     | name                                                     | moiety | form                     | route
|:------:| -------------------------------------------------------- |:------:| ------------------------ | -----
| `7399` | Co-amoxiclav Tablets                                     | `9152` | `Tablet`                 | `Oral`
| `7132` | Miconazole, Polymyxin B, Prednisolone Topical Suspension | `9744` | `Topical suspension`     | `Topical`
| `7464` | Enrofloxacin Injectable                                  | `9031` | `Solution for injection` | `Injectable`

### Therapeutic Moiety

| id     | name                                  | class                                                         | family            | combination
|:------:| ------------------------------------- | ------------------------------------------------------------- | ----------------- | -----------
| `9152` | Co-amoxiclav                          | Penicillins (aminopenicillins with beta-lactamase inhibitors) | Antimicrobial     | FALSE
| `9744` | Miconazole, Polymyxin B, Prednisolone | •                                                             | •                 | TRUE
| `9031` | Enrofloxacin                          | Quinolones                                                    | Antimicrobial     | FALSE
| `9237` | Miconazole                            | Imidazoles                                                    | Antimycotic       | FALSE
| `9645` | Polymixin B                           | Polymixins                                                    | Antimicrobial     | FALSE
| `9955` | Prednisolone                          | Glucocorticoids                                               | Anti-inflammatory | FALSE

### Therapeutic Moiety Combinations

| combination id | component id
|:--------------:|:------------:
| `9744`         | `9237`
| `9744`         | `9645`
| `9744`         | `9955`