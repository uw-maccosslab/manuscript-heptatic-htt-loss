# Notebook Entry Formatting Guide

## Overview

This document describes the formatting conventions for notebook entries in this repository. Notebook entries serve as detailed technical design documents that capture experimental approaches, decision rationale, and implementation details for collaborative projects.

---

## Document Structure

### Required Sections

Every notebook entry should include the following sections:

```
1. Title (H1)
2. Overview
3. Core Technical Content (varies by topic)
4. Open Questions and Discussion Points
5. References
6. Version History
```

### Optional Sections (as needed)

- Protocol Outlines
- Reagent/Materials Lists
- Comparison Tables (for evaluating alternatives)
- Design Constraints

---

## Formatting Conventions

### Headings

Use Markdown heading levels consistently:

| Level | Usage | Example |
|-------|-------|---------|
| H1 (`#`) | Document title only | `# PEA-MS Assay Design` |
| H2 (`##`) | Major sections | `## Overview`, `## References` |
| H3 (`###`) | Subsections | `### Primer Design` |
| H4 (`####`) | Sub-subsections | `#### Pros`, `#### Cons` |

### Section Separators

Use horizontal rules (`---`) to visually separate major sections:

```markdown
## Section One

Content here...

---

## Section Two

More content...
```

### Tables

Use tables for structured data, specifications, and comparisons:

```markdown
| Column A | Column B | Column C |
|----------|----------|----------|
| Data 1   | Data 2   | Data 3   |
```

**Best practices:**
- Include a header row
- Align columns for readability in source
- Keep tables focused (don't overload with too many columns)

### Code Blocks

Use fenced code blocks for:
- DNA/protein sequences
- Command-line instructions
- Workflow diagrams (ASCII art)
- Configuration examples

```markdown
​```
5'-ATGAGTAAAGGAGAAGAACTTTTC-3'
​```
```

For syntax highlighting, specify the language when applicable:

```markdown
​```python
def calculate_mass(sequence):
    return sum(aa_masses[aa] for aa in sequence)
​```
```

### Sequence Diagrams

Use ASCII diagrams to illustrate molecular structures or workflows:

```
N-term                                   C-term
  |                                         |
[GFP]-Gly-Gly-Ser-Lys-[barcode]-Arg-[6xHis]
                   ↑              ↑
              Lys-C site     Trypsin site
```

### Lists

**Bulleted lists** for unordered items:
```markdown
- Item one
- Item two
- Item three
```

**Numbered lists** for sequential steps or ranked items:
```markdown
1. First step
2. Second step
3. Third step
```

**Nested lists** for hierarchical information:
```markdown
1. Major step
   - Sub-detail A
   - Sub-detail B
2. Next major step
```

### Inline Formatting

| Format | Usage | Syntax |
|--------|-------|--------|
| **Bold** | Emphasis, key terms | `**text**` |
| *Italic* | Titles, species names, notes | `*text*` |
| `Code` | Gene names, commands, file names | `` `text` `` |

---

## Special Sections

### Workflow Summary

Include a high-level workflow summary near the top of the document:

```markdown
## Workflow Summary

​```
1. Step one
2. Step two
3. Step three → leads to step four
4. Final step
​```
```

### Pros and Cons

When comparing approaches, use structured pros/cons sections:

```markdown
### Approach A

#### Pros

1. **Advantage one** — explanation
2. **Advantage two** — explanation

#### Cons

1. **Disadvantage one** — explanation
2. **Disadvantage two** — explanation
```

### Open Questions

Include a section for unresolved questions and discussion points:

```markdown
## Open Questions and Discussion Points

### 1. Question Title
- Context or background
- Specific question to resolve
- Possible options to consider

### 2. Another Question
- Details here...
```

### References

Format references with:
- Numbered entries
- Author list with "et al." for >3 authors
- Year in parentheses
- Full title in sentence case
- Journal name in italics
- DOI link

```markdown
## References

1. **Author A, et al.** (2024). Title of the paper in sentence case. *Journal Name* Volume:Pages. https://doi.org/xxxxx
   - *Brief note about relevance to this document.*

2. **Author B, et al.** (2023). Another paper title. *Another Journal* Volume:Pages. https://doi.org/xxxxx
```

### Version History

Always include a version history table at the end:

```markdown
## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2025-01-01 | Initial draft |
| 0.2 | 2025-01-15 | Added protocol section |
| 0.3 | 2025-02-01 | Incorporated reviewer feedback |
```

**Version numbering:**
- 0.x — Draft versions
- 1.0 — First stable/reviewed version
- 1.x — Minor updates to stable version

---

## Content Guidelines

### Writing Style

1. **Be specific** — Include concrete details (concentrations, temperatures, catalog numbers)
2. **Be concise** — Use clear, direct language
3. **Be complete** — Document rationale for decisions, not just the decisions themselves
4. **Be honest** — Note uncertainties and open questions

### Technical Detail Level

- Include enough detail that someone unfamiliar with the project could understand the approach
- Link to external resources (papers, protocols) rather than reproducing them entirely
- Use tables and diagrams to convey complex information efficiently

### Collaboration Notes

- Use discussion sections to capture input from collaborators
- Attribute ideas and suggestions where appropriate
- Flag items that need input or approval from specific team members

---

## File Naming

Notebook entries should be named descriptively in lowercase with underscores:

```
<topic>_<subtopic>.md
```

Examples:
- `pea_ms_assay_design.md`
- `barcode_optimization_v2.md`
- `antibody_conjugation_protocol.md`
- `invention_disclosure.md`

---

## Template

A minimal notebook entry template:

```markdown
# [Title]

## Overview

Brief description of the topic and purpose of this document.

---

## [Main Technical Content]

### Subsection 1

Content...

### Subsection 2

Content...

---

## Open Questions and Discussion Points

### 1. [Question]
- Details...

---

## References

1. **Author, et al.** (Year). Title. *Journal*. https://doi.org/xxxxx

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | YYYY-MM-DD | Initial draft |
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1 | 2026-01-03 | Initial formatting guide |
