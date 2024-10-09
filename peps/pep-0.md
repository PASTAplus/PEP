# PEP-0: PEP Purpose and Guidelines

- Author(s): Mark Servilla
- Contact: servilla@edirepository.org
- Status: Implemented
- Type: Process
- Created: 2024-05-20
- Reviewed: 2024-06-11
- Final: 2024-10-09

## What is a PEP?

PEP stands for **PASTA Enhancement Proposal** and borrows extensively from the Python community's *Python Enhancement Proposal* process (see [here](https://peps.python.org/pep-0001)). We are fortunate (LOL) that we can reuse the PEP acronym to our advantage. While the first "P" in PEP is for "PASTA", we will extend the PEP process to all technical proposals related to projects of the Environmental Data Initiative (EDI).

A PEP is an open and transparent document for EDI and its community that serves as a blueprint of change within the technical operations of EDI. Its purpose is to systematically record information that proposes *significant* additions or modifications (including removal) to the design or implementation of EDI developed software, processes, and technical policies. PEPs will be reviewed and decided upon by the EDI Technical Steering Council (ETSC; pronounced "et-c"). A PEP is owned collaboratively between the author(s) and ETSC, allowing both parties to modify and refine the PEP accordingly. PEPs are stored as text files using Markdown formatting in a version-controlled repository ([GitHub](https://github.com/PASTAplus/PEP)) allowing their history to be preserved. This means anyone can review how a PEP evolved over time, either through git commands or by browsing the history on GitHub.

We emphasize that a PEP should only be written when considering a major technical change within EDI, while avoiding its use for bug fixes or minor enhancements such as cosmetic improvements to UI components. When in doubt, ask the ETSC if a PEP would be required.

## Types of PEPs

A PEP can be one of three types:

1. **Application (software)**: A PEP that proposes a significant change to an existing software application or the development of a new software application that affects one or more EDI services.
2. **Process (operational)**: A PEP that proposes a significant change to an existing operational process or the development of a new operational process. An operational process is defined as a series of non-software tasks that are performed by EDI to achieve a specific outcome. Examples include data publication workflows, curation workflows involving ezEML and the EDI data repository, and the PEP review process.
3. **Policy**: A PEP that proposes a significant change to an existing policy or the development of a new policy that affects EDI operations. Policies are defined as a set of rules or guidelines that govern the behavior of EDI staff, users, or software. Examples include data policies, privacy policies, and the EDI code of conduct.

## PEP Workflow

In general, the PEP process includes the following steps:

1. **Proposal Submission**: Anyone can propose a change to EDI's technical operations by writing a PEP that outlines the issue to be addressed, the proposed solution, and its potential impact on EDI's operations. However, the PEP must have a champion (normally the author) that will be the single point of contact for the ETSC to address correspondence. Each PEP is identified by a unique number (e.g., PEP-1, PEP-2, etc.) that is assigned by the ETSC. The champion should request a PEP number from the ETSC before submitting any formal documentation by writing a brief email describing the issue and solution to etsc@edirepository.org (a PEP number will be released only if the issue warrants a PEP). PEPs must be written using the EDI [PEP template](https://github.com/PASTAplus/PEP/blob/main/pep-template.md) and be submitted as a pull request to the [PEP repository](https://github.com/PASTAplus/PEP), at which point the PEP will be in the "Draft" status.
2. **Review**: The new PEP will change status from "Draft" to "Review" once the ETSC begins its review process. At this time, a PEP GitHub issue is created as the official means to convey feedback between ETSC, the PEP authors, and any community member wishing to comment on the PEP. During this process, the ETSC may ask the PEP authors for additional information, including modifications to the PEP, or to seek feedback on the PEP from the user community most affected by any proposed change. The PEP will remain in the "Review" status until the ETSC is satisfied that the proposal is ready to be accepted, at which point the status changes from "Review" to "Accepted". The ETSC may also decide to reject the PEP at this stage if it is deemed unnecessary or not feasible by adding a section to the PEP that details why it was rejected. If rejected, the status will change from "Review" to "Rejected" and the date of rejection will be entered into the "Final" section of the Preamble.
3. **Implementation**: If a PEP is accepted, the ETSC will identify the appropriate EDI technical lead who will be in charge of implementing the PEP. The technical lead will be responsible for ensuring that all required steps (e.g., GitHub stories, use-cases, and tasks) are in place before work begins on PEP. The PEP will remain in the "Accepted" status until the implementation is complete, at which point the status will change to "Implemented" and the date when implementation was completed will be entered into the "Final" section of the Preamble.

## What goes into a PEP?

A PEP should be about a single issue and its proposed solution. More specifically, the PEP should include a detailed description of the issue being addressed and a technical description of the proposed solution. The solution should be well justified (i.e., have a valid business case) and, if applicable, describe alternatives that were considered but not recommended. Any negative impacts to EDI operations (e.g., an increased load on a specific server) or the user community should also be described and justified. The more focused a PEP the better chances of it being accepted and accomplished. PEPs that are too vague or composed of multiple ideas will be rejected.

Required sections of a PEP include:

1. **PEP number and title**: The PEP number should be requested from the ETSC. The title should be short, concise, and relevant to the PEP (see this PEP).
2. **Preamble**: The preamble or header information (see this PEP).
    - **Author(s)**: List of PEP authors or champions
    - **Contact**: Email of primary contact for PEP
    - **Status**: One of *Draft*, *Review*, *Accepted* (or *Rejected*), or *Implemented*
    - **Type**: One of *Application (software)*, *Process (operational)*, or *Policy*
    - **Created**: Date PEP was submitted
    - **Reviewed**: Date PEP began review process
    - **Final**: Date PEP entered final status
3. **Introduction**: A brief statement about the PEP issue.
4. **Issue Statement**: A detailed statement of the issue the PEP will address, why the PEP should be considered, and the benefits it will deliver to EDI and/or the user community.
5. **Proposed Solution**: A detailed explanation of the proposed solution and the business case for its consideration is required. Where relevant, the explanation should include design decisions, security implications, use cases, example implementations,any potential for impact that negatively affects EDI operations or the user community, and an estimation of the proposed effort. Alternative solutions, if any, that were considered but not recommended should also be described in this section.
6. **Open issue(s)**: A list of open issues that require resolution before the PEP can be adopted and how they will be mitigated. This may include additional software dependencies that need to be resolved, potential security vulnerabilities, funding or resource limitations, or other issues that may affect the implementation of the PEP.
7. **References**: A list of references that support the PEP.
8. **Rejection (optional)**: If the PEP is rejected, a section should be added to the PEP by ETSC that details why it was rejected.

A PEP may deviate from the above form with prior approval of the ETSC. However, the PEP must adhere to the [GitHub flavored Markdown formatting guidelines](https://github.github.com/gfm/).