# PEP-4: Expanding the List of Supported Date and Time Formats for ECC and ezEML Congruence Checks



* Author(s): Colin Smith, Jon Ide
* Contact: colin.smith@wisc.edu
* Status: Draft
* Type: Policy
* Created: 2024-05-09
* Reviewed:
* Final:


## Introduction

The EML Congruence Checker (ECC) and the ezEML Check Data Tables are services that verify the accuracy of data packages before they are published in the EDI Data Repository. Currently, date and time format checks rely on a predefined list of acceptable formats. Any format not on the list is not validated and is simply flagged with a warning. This PEP recommends expanding the scope of these checks by adding common, unambiguous date and time formats (definitions below). This will improve the overall quality of date and time data published in the repository.


### Unambiguous Date and Time Format

A date and time format is considered unambiguous if it has a single, clear interpretation and is defined using a specific format string within the data package metadata. Here are the key characteristics of unambiguous formats:



* **Year Representation:** Four-digit years (YYYY) are always used to avoid potential confusion arising from double-digit years.
* **Month and Day:**  Months (MM) and days (DD) are represented with two digits each.
* **Delimiters:**  Year, month, and day components can be optionally separated by common delimiters such as hyphens (-) or forward slashes (/).
* **Order Flexibility:**  The order of year, month, and day components can vary (e.g., MM/DD/YYYY and DD/MM/YYYY are both acceptable).
* **Time Format:**  The 24-hour format (e.g. hh:mm:ss) is used to represent time, with the specific level of precision dictated by the data itself.
* **Timezone Offset:**  Whenever possible, the time format should include a timezone offset, but is not required


### Common Date and Time Formats

Common date and time formats are those frequently encountered within data packages stored in the repository. These will be identified through an analysis. Additionally, "common" means the format can be easily parsed by programming languages commonly used by data consumers (e.g. R, Python).


## Issue Statement

The quality checking services currently adhere to a limited subset of the internationally recognized [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard for unambiguous date and time formats, and adopt [RFC 3339](https://www.rfc-editor.org/info/rfc3339),  allowing separation of date and time parts with a space, not just a "T". _Hereafter we refer to these combined conventions as simply "ISO 8601"_. This subset of formats was curated by the ECC working group in 2017 based on its ability to unambiguously define components of dates and times useful to data consumers, its widespread adoption, and its broad support across programming languages. The checker functionality is as follows:



1. **Data Package with Declared Format String:** If a data package being evaluated has a format string specified within its metadata, and the format is recognized by the checker, regular expression matching is run on the corresponding data values. Any discrepancies between the declared format and the data itself are flagged as errors. This allows users to rectify these issues before uploading the data package to the repository.


2. **Data Package with Unrecognized Format:** However, if the data package uses a date and time format not included in the preferred list, the entire data package is bypassed for evaluation, and only a warning is issued regarding the unsupported format. This approach leads to a few issues:

   - **Inaccurate/Illegible Data Uploads:** Data packages with unsupported date and time formats can still be uploaded to the repository, potentially containing imprecise or unreadable date and time values. Warnings issued by the checker do not prevent upload.
   
   - **Limited Support for Common Formats:** User communities that prefer commonly used date and time formats not included in the current list are not adequately supported by the repository's data checks. This is an issue because these formats constitute a significant portion of data submissions.
   
   - **Unnecessary Reformatting Efforts:** Data curators, or data submitters, may expend time and effort reformatting data from one unambiguous format (non-ISO 8601) to another (ISO 8601) for compliance with the checker, which is a challenge during high submission loads.


## Proposed Solution

To address the limitations identified above, we propose the following enhancements to the EML congruence checker services:


1. **Expansion of Supported Formats:** The authoritative list of unambiguous date and time formats used by the ECC and ezEML congruence checks will be augmented to include unambiguous and commonly used formats. This will improve the coverage of date and time format validation.


2. **Continued Warnings for Unsupported Formats:** Data packages containing date and time formats not included in the expanded list will continue to trigger warnings during congruence checks. This will help raise awareness of non-standard formats and encourage users to consider adopting more widely accepted formats.


3. **Promoting ISO 8601 Adoption:** Educational materials and notes within relevant EDI resources (e.g. ECC and ezEML quality reports) will be added to emphasize the benefits of using ISO 8601 compliant date and time formats. This will guide data submitters towards improved data quality practices.

We propose a phased rollout of this enhancement, prioritizing ezEML. This allows us to validate date and time data currently published to the repository (outside ECC checks) without locking us into the expanded list. This approach increases the overall congruence of published data today.

It is important to note that implementing this solution does not hinder EDI from progressively strengthening data quality checks in the future to align with evolving organizational or scientific community requirements.


## Open issue(s)

Adoption of the proposed changes may impact a few EDI applications:

1. **ECC:** Two checks may be affected:
    1. `dateTimeFormatString`: This check verifies if the declared format is included in the preferred list.
    2. `dateFormatMatches`: This check ensures the data complies with the declared format.
2. **ezEML:** Two mirroring checks may be affected:
   1. `data_check6`: This check mirrors ECC's `dateTimeFormatString`.
   2. `data_check7`: This check mirrors ECC's `dateFormatMatches`.
3. **Data Explorer (DEX):** Date and time parsing operations that rely on the declared formats may require adjustments.

_Note, distinguishing between ISO 8601 formats and the expanded list, to enable feedback to the data submitter on date and time best practices, will require modifications to the code that processes this list as well as the list itself. Therefore, the scope of this PEP is not purely "Policy" but "Application" as well. Which applications will be affected, and how they will be modified to accommodate the proposed changes in this PEP are to be determined._


## References



* [ISO 8601, the worldwide exchange and communication of date and time-related data](https://en.wikipedia.org/wiki/ISO_8601)
* [RFC 3339, Date and Time on the Internet: Timestamps](https://www.rfc-editor.org/info/rfc3339)
* [ECC Quality Check notes for dates and time](https://github.com/EDIorg/ECC/tree/master/practices/dateTimeFormatString)