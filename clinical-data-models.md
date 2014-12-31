---
title: Clinical Data Models
---

## Clinical Data Models

These are some example clinical data models based off of CCDA models. You can use these as a base, or alter them as needed - these are just ideas for ways you can use custom classes. The descriptions are just the way we've imagined each field would be used, but you don't have to use it that way. Custom classes work the way **you** want them to.

Each model has a table and a `curl` command to add it as-is to your application. Use of that curl command requires an API key and a session token. The API key is acquired from the [dashboard](https://dashboard.catalyze.io/). The session token is part of the response from running the following command:

```
curl -X POST https://api.catalyze.io/v2/auth/signin \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -d '{"username": "<username>", "password": "<password>"}'
```

Where **&lt;api key&gt;**, **&lt;username&gt;**, and **&lt;password&gt;** are your applicationAPI key, username, and password, respectively (this should be an admin user - by default, when you create an application, your dashboard user is copied to that application and added as an administrator). The session token is the value of the **sessionToken** property in the JSON response.

### Patients

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
name             |object  |Object containing the names of the patient. This can include first, middle, last, maiden, and any titles.
dob              |string  |Date of birth, as an ISO 8601 timestamp.
address          |string  |Textual mailing address (multi-line) of the patient.
ssn              |string  |Social security number.
contact          |object  |Object containing contact information for the patient. This can include email addresses and phone numbers.
gender           |string  |
maritalStatus    |string  |
religion         |string  |
race             |string  |
ethnicity        |string  |

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "patients",
        "schema": {
            "name": "object",
            "dob": "string",
            "address": "string",
            "ssn": "string",
            "contact": "object",
            "gender": "string",
            "maritalStatus": "string",
            "religion": "string",
            "race": "string",
            "ethnicity": "string"
        }
    }
'
```

### Institutions

An Institution represents a physical location in which healthcare is provided. These could be health systems, hospitals, clinics, health plans, pharmaceutical companies, etc.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
name             |string  |The human-readable, public name of the institution.
abbreviation     |string  |Abbreviation/codename for the institution.
address          |string  |Textual address of the institution.
phone            |string  |Public phone number of the institution.
fax              |string  |Fax number for the institution.
email            |string  |Public contact email.
url              |string  |URL for the institution's website.
parent           |string  |ID of the institution that this one is effectively a child of.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "institutions",
        "schema": {
            "name": "string",
            "abbreviation": "string",
            "address": "string",
            "phone": "string",
            "fax": "string",
            "email": "string",
            "url": "string",
            "parent": "string"
        }
    }
'
```

### Providers

A Provider represents a single medical professional.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
name             |object  |Object containing the names of the provider. This can include first, middle, last, maiden, and any titles.
licensingState   |string  |US State name (or abbreviation).
licenseNumber    |string  |State license number.
gender           |string  |
medSchool        |string  |ID or name of the medical school this provider graduated from.
graduationYear   |integer |The year the provider graduated.
taxonomy         |string  |CMS Taxonomy code for the provider.
specialties      |array   |List of specialties by ID and/or name. IDs could come from [here](http://www.bcbsil.com/labor/pdf/code_manual/provider_specialty_codes.pdf).
subSpecialties   |array   |List of sub-specialties by ID and/or name.
locations        |array   |Array of Institution IDs where this provider operates.
languages        |array   |List of languages this provider speaks.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "providers",
        "schema": {
            "name": "object",
            "licensingState": "string",
            "licenseNumber": "string",
            "gender": "string",
            "medSchool": "string",
            "graduationYear": "integer",
            "taxonomy": "string",
            "specialties": "array",
            "subSpecialties": "array",
            "locations": "array",
            "languages": "array"
        }
    }
'
```

### Encounters

An Encounter is a documented meeting of a Provider and a Patient. This can be an office visit, emergency room visit, surgery, etc.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
date             |string  |ISO 8601 date string marking when this encounter occurred.
patient          |string  |The ID of the patient.
provider         |string  |The ID of the provider.
institution      |string  |The ID of the institution at which this encounter occurred.
type             |string  |Type of encounter - office visit, in-patient admission etc. Drawn from the ICD or CPT or SNOMED codesets.
complaints       |array   |An array of codes or textual descriptions of patient complaints from this encounter.
diagnoses        |array   |An array of codes from the ICD or SNOMED codesets indicating why the encounter happened. This is usually a diagnosis (Pneumonia).
notes            |string  |Miscellaneous notes attached to this encounter.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "encounters",
        "schema": {
            "date": "string",
            "patient": "string",
            "provider": "string",
            "institution": "string",
            "type": "string",
            "complaints": "array",
            "diagnoses": "array",
            "notes": "string"
        }
    }
'
```

### Vitals

A Vital is a quantifiable measurement of a certain type, correlated with a patient and a point in time.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
patient          |string  |ID of the associated patient.
encounter        |string  |ID of the encounter at which this vital was measured.
code             |string  |Code (or name) of the vital being measured.
value            |double  |Value measured.
unit             |string  |Unit of measure (e.g. "mm").

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "vitals",
        "schema": {
            "patient": "string",
            "encounter": "string",
            "code": "string",
            "value": "double",
            "unit": "string"
        }
    }
'
```

### Problems

A Problem represents a diagnosed health issue with a patient.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
patient          |string  |ID of the associated patient.
code             |string  |Code (or name) of the diagnosed issue.
description      |string  |Description of the problem as it affects the patient.
resolved         |boolean |Does it still affect the patient?
onset            |string  |ISO 8601 date string representing the approximate time the problem started to affect the patient.
causeOfDeath     |boolean |Indicator noting if this was the cause of death for the patient.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "problems",
        "schema": {
            "patient": "string",
            "code": "string",
            "description": "string",
            "resolved": "boolean",
            "onset": "string",
            "causeOfDeath": "boolean"
        }
    }
'
```

### Procedures

A Procedure entry represents a procedure performed on a patient. This would include procedures such as X-Rays, office visits - essentially any kind of billable and hence coded work performed on the patient.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
date             |string  |ISO 8601 date string marking when this procedure was performed.
patient          |string  |ID of the patient on which the procedure was performed.
provider         |string  |ID of the provider that performed the procedure.
problem          |string  |ID of the problem that necessitated this procedure, if any.
code             |string  |The code (or name) for the procedure (from SNOMED or CPT).
site             |string  |Anatomical location upon which the procedure was performed.
performed        |boolean |Indicates whether or not the procedure has been performed.
specimens        |array   |Array of specimens collected as a result of the procedure. The structure of this array can include codes/values, or refer to a vital.
notes            |string  |Miscellaneous notes attached to the procedure.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "procedures",
        "schema": {
            "date": "string",
            "patient": "string",
            "provider": "string",
            "problem": "string",
            "code": "string",
            "site": "string",
            "performed": "boolean",
            "specimens": "array",
            "notes": "string"
        }
    }
'
```

### Results

A Result entry represents the outcome of a lab or test.

NAME             |TYPE    |DESCRIPTION
-----------------|--------|-----------
date             |string  |ISO 8601 date string marking when the lab/test was performed.
patient          |string  |ID of the patient for which the lab/test was performed.
provider         |string  |ID of the provider that performed the lab/test.
code             |string  |Code (or name) of the lab/test, from the LOINC codeset.
performed        |boolean |Indicates whether or not the lab/test has been performed.
specimens        |array   |Array of specimens collected as a result of the procedure. The structure of this array can include codes/values, or refer to a vital.
vitals           |array   |Array of IDs corresponding to vitals entries that were measured as part of this lab/test.
notes            |string  |Miscellaneous notes attached to the lab/test.

```
curl -X POST https://api.catalyze.io/v2/classes \
    -H "Content-Type: application/json" \
    -H "X-Api-Key: <api key>" \
    -H "Authorization: Bearer <session token>" -d '
    {
        "name": "results",
        "schema": {
            "date": "string",
            "patient": "string",
            "provider": "string",
            "code": "string",
            "site": "string",
            "performed": "boolean",
            "specimens": "array",
            "vitals": "array",
            "notes": "string"
        }
    }
'
```
