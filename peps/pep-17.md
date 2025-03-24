# PEP-17: Supporting Automated Reading of Date and Time Values by Common Programming Languages

- Author(s): Colin Smith  
- Contact: colin.smith@wisc.edu  
- Status: Draft  
- Type: Application  
- Created: 2025-03-24  
- Reviewed:  
- Final:  

## Introduction

This PEP proposes the creation of tools and resources to support automated reading and parsing of date and time values declared in EML metadata by common programming languages such as R and Python. The goal is to enhance the usability of EML-declared date/time formats for programmatic workflows and reduce the friction data consumers encounter when interpreting or transforming temporal data.

## Issue Statement

Date and time values in EML data packages are associated with format strings that describe how the values are encoded. While these format strings serve important roles in quality checking and human interpretation, they are often difficult to use directly in software pipelines.

Currently, there is no standardized mapping between EML format strings and the syntax used in common programming libraries like:

- `strptime()` / `strftime()` in Python’s `datetime` module or `pandas.to_datetime()`
- `as.POSIXct()` and `strptime()` in R’s `base` and `lubridate` packages

This creates barriers for automated processing:

- Data consumers must manually convert EML format strings into compatible equivalents for their toolchain.
- Small inconsistencies (e.g., use of slashes vs. dashes, presence of timezones, case sensitivity) can lead to failed parses or misinterpreted values.
- Data pipelines become more fragile and harder to generalize across data packages.

By creating a centralized and maintained set of mappings and tooling support, we can better serve developers and analysts working with EDI data.

## Proposed Solution

We propose to:

1. **Develop and maintain a mapping table** that links EML-compliant format strings with equivalent parsing strings for:
   - Python's `datetime` and `pandas` libraries
   - R’s `strptime()` and `lubridate` formats

2. **Package this mapping as a lightweight Python module and optional web-accessible resource**, such as a JSON or CSV lookup table served from a GitHub repository or web API.

3. **Publish documentation and examples** showing how to retrieve and apply these mappings in R and Python code.

4. **Coordinate with the proposed date/time checking library ([PEP-16](https://github.com/PASTAplus/PEP/blob/main/peps/pep-16.md))** so that the two systems share a canonical format list and avoid duplication.

This solution will help:

- Data consumers write reliable, programmatic parsing routines.
- Downstream applications (e.g., DEX or custom data processing pipelines) integrate seamlessly with EML-declared formats.
- Promote best practices for authors when selecting date and time formats that are easily machine-readable.

## Open issue(s)

1. **Mapping Ambiguity**: Some EML format strings may be ambiguous or too loosely defined to map precisely to a programming language syntax. We need rules to flag or handle these edge cases gracefully.
2. **Canonical Format Source**: The mapping must align with the authoritative list of supported formats defined in the date/time checking library (see related [PEP-16](https://github.com/PASTAplus/PEP/blob/main/peps/pep-16.md)), requiring coordination and versioning between the two.
3. **R Package Support**: Since this PEP is scoped to a Python implementation, we must decide whether R support will be limited to documentation/examples or whether we will also provide a corresponding R package.
4. **Community Education**: Guidance may be needed to ensure data authors declare format strings that are actually parseable in target languages.
5. **Future Language Support**: We may wish to support other environments (e.g., MATLAB) down the line. The architecture should support extensibility.

## References

- [PEP-16: Centralized Library for Date and Time Congruence Checking](https://github.com/PASTAplus/PEP/blob/main/peps/pep-16.md) 
- [Python strftime/strptime documentation](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)  
- [pandas.to_datetime() docs](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.to_datetime.html)  
- [R base strptime docs](https://stat.ethz.ch/R-manual/R-devel/library/base/html/strptime.html)  
- [lubridate cheat sheet (RStudio)](https://posit.co/resources/cheatsheets/)  

## Rejection
