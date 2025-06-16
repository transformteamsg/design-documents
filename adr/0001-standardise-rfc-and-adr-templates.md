# 0001 Standardise RFC and ADR Templates

**Author(s):**

- Yi Ming Peh / [@yimingiscold](https://github.com/yimingiscold)
- Alex Ng / [@axxng](https://github.com/axxng)
- Elieen Kang / [@kmye](https://github.com/kmye)
- Chadin Anuwattanaporn / [@chadinwork](https://github.com/chadinwork)

**Status:** Accepted

**Discussion:** [RFC #1](https://github.com/transformgovsg/design-documents/issues/1)

## Context

We are initiating an RFC process to encourage collaborative design discussions and better record architectural decisions. To ensure clarity and consistency from the outset, we needed to define and standardise both the RFC template (for GitHub Issues) and the ADR template (for accepted decisions), along with recommended file structure and naming conventions.

## Decision

We will adopt standard RFC and ADR templates with the following key decisions:

1. **Workflow:**

   - RFCs are created as GitHub Issues using the standard template
   - Discussion and feedback happen in the GitHub Issue
   - Accepted RFCs are converted to ADRs and committed to the repository
   - Rejected RFCs are closed but can be reopened if new arguments arise

2. **Naming Convention:**

   - RFC Titles: Short, descriptive titles in sentence case
   - ADR Filenames: 4-digit number followed by kebab-case description (e.g., `0001-standardise-rfc-and-adr-template.md`)

3. **File Structure:**

   - Engineering-wide ADRs: `/adr/`
   - Product-specific ADRs: `/product-foo/`

4. **ADR Status Values:**

   - `Accepted`: Decision is currently active and in use
   - `Superseded`: Decision has been replaced by a newer ADR
   - `Deprecated`: Decision is no longer recommended

5. **Standard Templates:**

   a. **RFC Template (for GitHub Issues):**

   ```markdown
   ## Author(s)

    <!--
    Who is proposing this RFC? List names and (optionally) GitHub handle.
    -->

   ## Discussion & Voting Timeline

    <!--
    Specify the timeline for the RFC process, including when discussion ends and voting begins and ends. Voting starts when the author posts a comment to initiate it, typically asking for 👍 or 👎 reactions.
   
    Example:
    - Discussion open until 2025-06-13
    - Voting starts when the author calls for it
    - Voting ends on 2025-06-23 or when consensus is reached
    -->

   ## Summary

    <!--
    Briefly summarise what this RFC proposes. (1-2 sentences)
    -->

   ## Motivation

    <!--
    Why are we creating this RFC? What problem or opportunity does it address?
    -->

   ## Detailed Design

    <!--
    Describe the proposed solution or design in detail. Include examples, diagrams, or code if relevant.
    -->

   ## Alternatives Considered

    <!--
    What other solutions or approaches did you consider? Why were they not chosen?
    -->

   ## Drawbacks

    <!--
    What are the potential risks, costs, or downsides of this proposal?
    -->

   ## Impact

    <!--
    How will this change affect current systems, workflows, or users?
    -->

   ## Open Questions

    <!--
    (Optional) Are there unresolved issues or aspects that require further discussion?
    Remove this section if there are none.
    -->

   ## References

    <!--
    (Optional) Link to related issues, discussions, documents, or relevant standards.
    Remove this section if there are none.
    -->
   ```

   b. **ADR Template (for Accepted RFCs):**

   ```markdown
   # [XXXX-title]

    <!--
    A short, descriptive title for this decision. Use a 4-digit number for sorting and referencing, e.g., 0001.
    -->

   **Author(s):**

    <!--
    The proposer plus anyone who made substantial contributions to the decision, design, or the ADR itself — such as co-authors, significant reviewers, or people whose input shaped the outcome.
    -->

   **Status:**

    <!--
    The current status of this ADR. Supported values are:
    - Accepted
    - Superseded (if this ADR has been replaced by another)
    - Deprecated
   
    Note that when creating a new ADR that replaces one or more existing ADRs, you should set this ADR’s status accordingly and fill out the `Supersedes` field with links to the replaced ADR(s). Conversely, when updating an existing ADR to reflect that it has been superseded (changing its status from `Accepted` to `Superseded`), you should update its `Superseded by` field to reference the new ADR(s) that supersede it.
   
    -->

   **Discussion:**

    <!--
    Link to the original RFC issue, e.g. [RFC #123](https://github.com/transformgovsg/design-documents/issues/123).
    -->

   **Supersedes:**

    <!--
    (Optional, required only when this ADR replaces one or more existing ADRs.)
    Provide the link(s) to the ADR(s) this new ADR supersedes, e.g.
    [ADR 0001](https://github.com/transformgovsg/design-documents/adr/0001-standardise-rfc-and-adr-template.md)
    -->

   **Superseded by:**

    <!--
    (Optional, required only when this ADR has been replaced by another ADR.)
    Update this field when changing the status of this ADR from Accepted to Superseded to reference the new ADR(s) that supersede it, e.g.
    [ADR 0002](https://github.com/transformgovsg/design-documents/adr/0002-new-standardise-rfc-and-adr-template.md)
    -->

   ## Context

    <!--
    What is the background or context for this decision? Describe the problem statement, relevant factors, and references to the RFC and its discussion.
    -->

   ## Decision

    <!--
    What architectural or technical decision was made? Clearly state the outcome.
    -->

   ## Consequences

    <!--
    What are the consequences, positive or negative, of this decision? Include any follow-ups or next steps.
    -->

   ## References

    <!--
    (Optional) Link to related issues, discussions, documents, or relevant standards.
    Remove this section if there are none.
   -->
   ```

## Consequences

### Positive

- Establishes a clear and consistent process for submitting and recording architectural decisions
- Improves traceability of decisions through standardised documentation
- Provides clear structure for both RFC discussions and ADR documentation
- Separates engineering-wide and product-specific decisions for better organization

### Negative

- Contributors will need time to learn and adapt to the new process
- Requires consistent adherence to naming and folder conventions
