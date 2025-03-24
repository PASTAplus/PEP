# PEP-16: Centralized Library for Date and Time Congruence Checking

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application
- Created: 2025-03-24
- Reviewed:
- Final:

## Introduction

This PEP proposes the development of a centralized Python library to handle all date and time congruence checks across EDI applications such as ECC, ezEML, and DEX. The primary motivation is to unify and simplify the date/time validation logic that is currently duplicated across these systems. A common library will improve maintainability, reduce code redundancy, and ensure consistent behavior across platforms.

## Issue Statement

Currently, each EDI application that performs quality checks on date and time values—such as ECC, ezEML, and DEX—implements its own validation logic. This fragmented approach introduces the following issues:

- **Inconsistencies in Behavior**: Applications may differ in how they interpret or validate format strings, leading to inconsistent user experiences.
- **Duplicated Effort**: Fixes and improvements must be replicated across multiple codebases, increasing the risk of bugs and technical debt.
- **Maintenance Overhead**: As standards evolve or new formats are supported, updates must be coordinated manually across multiple systems.

These challenges become more pronounced as the number of supported formats grows, and as EDI aims to enforce higher data quality standards for date and time values.

## Proposed Solution

We propose creating a standalone, reusable Python library for performing date and time congruence checks. The library will be designed to:

1. **Validate data values against declared EML format strings**, flagging mismatches in structure, padding, delimiters, or ordering.
2. **Support a shared, authoritative list of accepted date/time format strings**, including ISO 8601 and additional common formats identified by the community.
3. **Be modular and extensible**, allowing for additional validation logic to be introduced in the future (e.g., partial date components, timezone handling).
4. **Provide integration hooks for ECC, ezEML, and DEX**, replacing their internal date/time validation logic.
5. **Enable consistent error and warning messaging**, improving user understanding and support across applications.

While this PEP focuses solely on date and time validation, the library architecture should allow for future integration with other types of congruence checks, proposed in [PEP-8 (Unified congruency checker)](https://github.com/PASTAplus/PEP/blob/main/peps/pep-8.md).

## Open issue(s)

1. **Format Compatibility with Existing Applications**: DEX uses Pandas-based parsing, which may require translation logic between EML format strings and `strptime`/`to_datetime` syntax. This mapping will need to be documented and tested.
2. **Shared Format List Governance**: The list of supported formats must be centrally maintained and versioned. We will need to determine who is responsible for managing additions or changes.
3. **Backward Compatibility**: Legacy packages may use format strings or data styles not covered in the updated library. Decisions must be made on how to handle these cases (e.g., warnings, soft validation).
4. **Error/Warning Messaging Standardization**: To ensure consistent feedback to users, the library should implement a structured error/warning interface compatible with ECC and ezEML output formats.
5. **Performance Testing**: The validation logic must be performant enough for use within web-based tools (ezEML) and automated back-end checks (ECC, DEX).

## References

- [PEP-4: Expanding the List of Supported Date and Time Formats](https://github.com/PASTAplus/PEP/blob/main/peps/pep-4.md)
- [PEP-4: Proposal for Date and Time Congruence Library](https://github.com/PASTAplus/PEP/issues/7)
- [ISO 8601 Specification](https://en.wikipedia.org/wiki/ISO_8601)
- [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/info/rfc3339)
- [Python strptime/strftime Documentation](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)


## Rejection
