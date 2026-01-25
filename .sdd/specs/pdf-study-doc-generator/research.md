# Research & Design Decisions

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `pdf-study-doc-generator`
- **Discovery Scope**: New Feature (greenfield Claude Code skill)
- **Key Findings**:
  - Claude Code skills are markdown files with frontmatter defining tool access and parameters
  - PDF text extraction requires native tool support (Read tool in Claude Code can process PDFs)
  - Output must strictly match reference format: plain text topics, hierarchical bullets, no bold formatting
  - Content analysis must be LLM-based (no external knowledge/web access during generation)

## Research Log

### Claude Code Skill Structure
- **Context**: Need to understand how Claude Code skills are implemented
- **Sources Consulted**: Examined existing `.claude/commands/sdd/*.md` files
- **Findings**:
  - Skills are markdown files with YAML frontmatter
  - Frontmatter specifies: description, allowed-tools, argument-hint
  - Body contains instructions structured with background_information, instructions, tool guidance, output description, safety & fallback
  - Skills receive arguments via `$ARGUMENTS` variable
  - Skills are invoked with `/skill-name` syntax
- **Implications**:
  - Generator must be implemented as a markdown skill file in `.claude/commands/`
  - Must specify Read tool for PDF access, Write tool for output file
  - Instructions must be clear and structured for Claude to execute autonomously

### PDF Processing in Claude Code
- **Context**: Determine how to extract text from PDF files
- **Sources Consulted**: Claude Code documentation, Read tool capabilities
- **Findings**:
  - Claude Code's Read tool can process PDF files directly
  - PDFs are processed page by page, extracting both text and visual content
  - Text extraction preserves document structure reasonably well
  - No external PDF libraries needed - native tool support exists
- **Implications**:
  - Use Read tool with PDF file path for extraction
  - No need for pypdf2, pdfplumber, or similar libraries
  - Chapter identification must be done via text pattern matching
  - May need multiple Read invocations if chapter spans many pages

### Target Output Format Analysis
- **Context**: Must generate markdown matching `super revisão.md` style exactly
- **Sources Consulted**: Analyzed `super revisão.md` structure
- **Findings**:
  - Main structure: `# Subject` → `## Cap. N` → topics → bullets
  - Topics are plain text lines: `Topic Name (alternative) - Definition`
  - Sometimes topics are just names without definitions
  - Plain text continuation lines follow topics for context
  - Bullets use `*` with 2-space indentation for nesting
  - Sub-topics within bullets: `Name - Definition` (plain text)
  - **Critical**: NO bold, italic, or other formatting in content body
  - Images: `![][imageN]Description`
- **Implications**:
  - Must suppress Claude's natural tendency to bold key terms
  - Formatting instructions must be explicit and repeated
  - Need pattern recognition to identify topics vs. bullets vs. continuations
  - Image detection must create numbered placeholders

### Content Analysis Constraints
- **Context**: Requirement states "no external knowledge or resources"
- **Sources Consulted**: Requirements 3, 5
- **Findings**:
  - Must analyze PDF content exclusively
  - Cannot use web search or external APIs during generation
  - Must extract concepts, relationships, hierarchy from PDF text alone
  - LLM capabilities sufficient for structure recognition and summarization
- **Implications**:
  - Analysis phase uses Claude's reading comprehension only
  - No tool calls during content generation except Write for output
  - Must instruct Claude explicitly to use only extracted content
  - Quality depends on PDF text clarity and structure

### Chapter Identification Strategy
- **Context**: Need to extract specific chapter from multi-chapter PDF
- **Sources Consulted**: Requirements 2.3, 2.4, 2.5
- **Findings**:
  - Chapter identifiers vary: "2", "Cap. 3", "Chapter 5"
  - Chapters typically marked by heading patterns
  - May need flexible regex matching for detection
  - Chapter ends at next chapter or document end
- **Implications**:
  - Must search extracted text for chapter markers
  - Should suggest found chapters if specified chapter not found
  - Extraction window: from chapter start to next chapter start
  - Need robust pattern matching with multiple identifier formats

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Single Skill File | All logic in one skill markdown file | Simple, self-contained, matches existing patterns | Complex instructions may be lengthy | Recommended - aligns with SDD skill pattern |
| Multi-Component | Separate skill + helper scripts | Cleaner separation of concerns | Added complexity, multiple files to maintain | Overkill for this feature |
| Pipeline Architecture | Separate extract → analyze → generate phases | Clear phase separation, testable | Over-engineered for single-command use case | Not needed for this scope |

## Design Decisions

### Decision: Single Markdown Skill File Architecture
- **Context**: Need to structure the PDF study document generator for Claude Code
- **Alternatives Considered**:
  1. Single markdown skill file with all logic in instructions
  2. Multi-file approach with separate components
  3. Pipeline architecture with explicit phase separation
- **Selected Approach**: Single markdown skill file (`.claude/commands/generate-study-doc.md`)
- **Rationale**:
  - Matches existing SDD skill pattern
  - Self-contained and easy to maintain
  - All logic can be expressed in structured instructions
  - Claude can execute multi-phase logic within single skill invocation
- **Trade-offs**:
  - Benefits: Simple, consistent with project patterns, no dependencies
  - Compromises: Instructions may be lengthy, harder to unit test individual phases
- **Follow-up**: Monitor instruction complexity; refactor if exceeds ~500 lines

### Decision: Sequential Processing Flow
- **Context**: How to structure the generation workflow
- **Alternatives Considered**:
  1. Sequential: Extract → Identify Chapter → Analyze → Generate → Write
  2. Parallel: Extract all content, then process in parallel
  3. Streaming: Process content incrementally as extracted
- **Selected Approach**: Sequential processing with explicit phases
- **Rationale**:
  - Clear phase separation ensures correct execution order
  - Each phase's output feeds into next phase
  - Easier to provide progress feedback
  - Matches user expectations from requirements
- **Trade-offs**:
  - Benefits: Predictable, debuggable, clear error handling
  - Compromises: Slightly slower than parallel (but acceptable for 2-minute target)
- **Follow-up**: Add progress status messages between phases

### Decision: LLM-Based Content Analysis
- **Context**: How to identify topics, hierarchies, and relationships in extracted text
- **Alternatives Considered**:
  1. LLM-based: Use Claude's comprehension to analyze structure
  2. Rule-based: Use regex patterns and heuristics
  3. Hybrid: Combine LLM analysis with structural rules
- **Selected Approach**: LLM-based analysis with strict formatting instructions
- **Rationale**:
  - Claude excels at understanding semantic relationships and hierarchies
  - Requirement 3 needs concept recognition, not just pattern matching
  - Can adapt to different subject matter and writing styles
  - No code/dependencies needed
- **Trade-offs**:
  - Benefits: Flexible, high-quality analysis, handles variety
  - Compromises: Less deterministic than rule-based, dependent on prompt quality
- **Follow-up**: Include detailed examples in skill instructions to guide analysis

### Decision: Plain Text Formatting Enforcement
- **Context**: Must generate markdown without bold/italic formatting (Requirement 4.10, 6.9)
- **Alternatives Considered**:
  1. Explicit formatting instructions + examples
  2. Post-processing regex to strip formatting
  3. Formatting validation before write
- **Selected Approach**: Explicit instructions + examples + validation reminder
- **Rationale**:
  - Claude follows explicit formatting rules when clearly stated
  - Examples from `super revisão.md` provide clear reference
  - Reminder in generation phase reinforces constraint
- **Trade-offs**:
  - Benefits: Instructive, leverages Claude's capabilities, no post-processing
  - Compromises: Requires vigilance in prompt design
- **Follow-up**: Test with various subjects to ensure formatting consistency

## Risks & Mitigations

- **Risk**: PDF extraction may fail on scanned/image-based PDFs
  - **Mitigation**: Error handling with clear message; suggest OCR or text-based PDF

- **Risk**: Chapter identification may fail with non-standard formats
  - **Mitigation**: Flexible pattern matching; suggest found chapters if target not found

- **Risk**: Generated content may include bold formatting despite instructions
  - **Mitigation**: Explicit anti-formatting instructions; examples showing plain text style

- **Risk**: Large PDFs may cause performance issues
  - **Mitigation**: Extract only specified chapter; progress feedback for user awareness

- **Risk**: Complex textbook content may produce poor hierarchical structure
  - **Mitigation**: Detailed analysis instructions; examples of hierarchy patterns

## References
- Claude Code Skills: `.claude/commands/sdd/*.md` (existing skill implementations)
- Target Format: `super revisão.md` (reference output style)
- Requirements: `.sdd/specs/pdf-study-doc-generator/requirements.md`
