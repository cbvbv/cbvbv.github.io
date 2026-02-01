# Research & Design Decisions

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `raw-text-study-doc-generator`
- **Discovery Scope**: New Feature (Claude Code Skill)
- **Key Findings**:
  - Claude Code skill pattern uses markdown files with frontmatter in `.claude/commands/` directory
  - Text processing requires pure analysis of plain text without external LLM knowledge injection
  - Pattern matching and structural inference from unformatted text is core challenge
  - Output must follow strict formatting rules (backslash-dash separators, no bold, nested bullets)

## Research Log

### Claude Code Skill Architecture Pattern
- **Context**: Need to understand how to create a new skill that integrates with Claude Code CLI
- **Sources Consulted**:
  - Existing SDD skills in `.claude/commands/sdd/`
  - spec-init.md structure and frontmatter pattern
- **Findings**:
  - Skills are markdown files with YAML frontmatter
  - Frontmatter includes: `description`, `allowed-tools`, `argument-hint`
  - Skills use `$ARGUMENTS` variable to access command-line arguments
  - Skills follow the pattern: `<background_information>` + `<instructions>` + tool guidance + output format
  - Skills are organized by namespace (e.g., `sdd/` for spec-driven development commands)
- **Implications**:
  - New skill should be created at `.claude/commands/generate-study-doc.md`
  - Must define clear argument structure for file path input
  - Should follow established prompt engineering patterns from SDD skills

### Plain Text Analysis Without External Knowledge
- **Context**: Requirement 5 mandates strict fidelity to source text without external knowledge
- **Challenge**: Claude's standard behavior includes using training knowledge to enhance responses
- **Findings**:
  - Must explicitly instruct to only use information from the provided text
  - Need clear boundary instructions: "Do not add context, explanations, or corrections"
  - Should process text as pure data extraction task, not knowledge synthesis
- **Implications**:
  - Prompt must emphasize source-only processing multiple times
  - Error handling must not "fix" or "improve" content
  - Language preservation is critical (no translation, no synonym substitution)

### Text Structure Inference Patterns
- **Context**: Raw text has no formatting - need to identify topics, hierarchies, and definitions
- **Sources Consulted**:
  - Example raw_chapter.txt (Rococó chapter)
  - Example super revisão.md output format
- **Findings**:
  - Topics often start paragraphs or follow clear section breaks
  - Definitions follow patterns like "X is...", "X - explanation"
  - Hierarchical relationships indicated by:
    - Indentation in lists
    - Contextual continuity (sub-concepts follow main concepts)
    - Temporal/causal connectors ("por isso", "depois", "além disso")
  - Glossary sections have clear markers ("Glossário:", "TOME NOTA")
- **Implications**:
  - Need heuristics for topic detection at paragraph boundaries
  - Must identify definition patterns in natural language
  - Should track contextual relationships between concepts
  - Glossary filtering requires pattern matching for section headers

### Output Formatting Requirements
- **Context**: Study notes must match exact format of super revisão.md
- **Findings**:
  - Main topics: Plain text + `\-` + definition (NO bold)
  - Subtopics: `* Topic \-` Definition
  - Nesting uses 2-space indentation per level
  - Parenthetical alternatives: `(ou Alternative Name)`
  - Examples: `Ex: specific example text`
  - No section headers beyond `# Subject` and `## Cap. N`
- **Implications**:
  - Must use exact character sequence `\-` (not `-` or ` - `)
  - Bold/italic formatting explicitly prohibited
  - Indentation must be precisely 2 spaces per level
  - Must preserve Portuguese diacritics exactly

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Single-Prompt Skill | All processing in one Claude prompt with file read | Simple, no intermediate state | May struggle with very long chapters | Best for MVP, aligns with existing skill patterns |
| Pipeline Architecture | Multi-step: extract → analyze → format → write | Clear separation of concerns, testable | More complex, requires state management | Overkill for this use case |
| Template-Based Generation | Extract data, fill template | Predictable output format | Less flexible for varied input | Template approach conflicts with fidelity requirement |

## Design Decisions

### Decision: Single-Prompt Processing Architecture
- **Context**: Need to choose between single-prompt vs multi-stage processing
- **Alternatives Considered**:
  1. Single unified prompt - read file, analyze, generate output in one pass
  2. Multi-stage pipeline - separate extraction, analysis, formatting phases
- **Selected Approach**: Single unified prompt with structured instructions
- **Rationale**:
  - Aligns with existing Claude Code skill patterns (spec-init, spec-requirements)
  - Reduces complexity and potential for intermediate state errors
  - Claude's context window is sufficient for typical chapter lengths
  - Maintains source text fidelity better (no data transformation between stages)
- **Trade-offs**:
  - Benefits: Simpler implementation, easier to maintain, matches established patterns
  - Compromises: Less modular, harder to optimize individual processing steps
- **Follow-up**: Monitor performance with large chapters (>100KB)

### Decision: Skill Namespace and Command Name
- **Context**: Need to determine where skill lives and how it's invoked
- **Alternatives Considered**:
  1. `/generate-study-doc` - top-level command
  2. `/study:generate` - study namespace
  3. `/doc:study-gen` - doc namespace
- **Selected Approach**: `/generate-study-doc` (top-level)
- **Rationale**:
  - Descriptive and discoverable
  - Follows verb-noun pattern common in CLI tools
  - Not part of SDD workflow, so shouldn't be in `sdd:` namespace
  - Clear purpose from command name alone
- **Trade-offs**:
  - Benefits: Self-documenting, easy to remember
  - Compromises: Slightly longer to type than abbreviated version
- **Follow-up**: None

### Decision: Error Handling Strategy
- **Context**: How to handle malformed input, missing files, invalid content
- **Alternatives Considered**:
  1. Fail fast with clear error messages
  2. Attempt recovery/correction with warnings
  3. Partial processing with degraded output
- **Selected Approach**: Fail fast with descriptive errors (no recovery attempts)
- **Rationale**:
  - Aligns with source fidelity requirement (no "fixing" content)
  - User can correct source and re-run
  - Prevents generation of incorrect study notes
  - Matches pattern from Requirement 6.7
- **Trade-offs**:
  - Benefits: Clear failure modes, maintains data integrity
  - Compromises: Less forgiving for minor input issues
- **Follow-up**: Ensure error messages guide users to solutions

### Decision: Language Detection Approach
- **Context**: Requirement 7 requires language preservation and detection
- **Alternatives Considered**:
  1. Explicit language parameter from user
  2. Automatic detection from text content
  3. Detect from file metadata/path
- **Selected Approach**: Automatic detection from content (first paragraphs)
- **Rationale**:
  - Reduces user friction (no extra parameters)
  - Reliable for educational texts (consistent vocabulary)
  - Can detect Portuguese vs English from diacritics and common words
- **Trade-offs**:
  - Benefits: User-friendly, works for mixed-language environments
  - Compromises: Could misdetect for very short texts
- **Follow-up**: Add override parameter if needed in future

## Risks & Mitigations
- **Risk: Long input files exceed context window** — Mitigation: Add file size check (Req 1.4), warn if >50KB, suggest splitting chapters
- **Risk: Ambiguous topic boundaries in plain text** — Mitigation: Use multiple heuristics (paragraph breaks, capitalization, context), document limitations
- **Risk: Glossary sections not properly filtered** — Mitigation: Compile comprehensive list of section markers (TOME NOTA, CONEXÕES, Glossário, EXPECTATIVAS, etc.)
- **Risk: Output format drift from template** — Mitigation: Explicit formatting instructions with examples, post-generation validation
- **Risk: External knowledge leakage into output** — Mitigation: Multiple emphatic instructions to use only source text, testing with deliberately incorrect source content

## References
- [Claude Code Skills Documentation](https://github.com/anthropics/claude-code) — Skill creation patterns
- [Markdown Specification](https://commonmark.org/) — Output format compliance
- [Unicode Normalization](https://unicode.org/reports/tr15/) — Diacritic preservation strategy
