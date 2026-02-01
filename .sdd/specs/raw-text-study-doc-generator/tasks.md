# Implementation Tasks

## Overview
Implementation tasks for the raw-text-study-doc-generator feature following the single-prompt AI architecture. All tasks build toward a Claude Code skill that transforms raw educational text into structured study notes using AI processing.

---

## Tasks

- [ ] 1. Create skill command structure and interface
- [x] 1.1 (P) Create skill markdown file with frontmatter and help documentation
  - Define skill at `.claude/commands/generate-study-doc.md`
  - Add YAML frontmatter with description, allowed-tools (Read, Write, Glob), and argument-hint
  - Write help documentation explaining command usage, expected input format, and output location
  - Include example invocation pattern showing file path argument
  - Document error scenarios (file not found, file too small, no arguments provided)
  - _Requirements: 1.1, 1.2, 1.5, 8.1, 8.2, 8.3, 8.4, 8.5_

- [x] 1.2 (P) Implement argument parsing and validation logic
  - Extract file path from `$ARGUMENTS` variable
  - Handle missing arguments case (display help text)
  - Validate file path is non-empty string
  - Set up error messaging for invalid input patterns
  - _Requirements: 1.1, 1.2_

- [ ] 2. Implement file I/O operations and validation
- [x] 2.1 Implement input file reading with validation
  - Use Read tool to load file content from provided path
  - Check file existence and handle "file not found" error with clear message including path
  - Validate file size (minimum 50 characters) and display warning if below threshold
  - Display file size warning at 50KB threshold (performance consideration)
  - Reject files over 100KB with context limit error message
  - Preserve UTF-8 encoding throughout read operation
  - _Requirements: 1.3, 1.4, 6.7_

- [x] 2.2 Implement output file generation and overwrite protection
  - Generate output file path using convention: `<input-basename>_study_notes.md`
  - Check if output file already exists using Glob tool
  - Prompt user for confirmation before overwriting existing files
  - Use Write tool to atomically create output file
  - Handle write failures with descriptive error messages (disk space, permissions)
  - Ensure no partial output on any failure condition
  - _Requirements: 6.1, 6.2, 6.3, 6.5, 6.7_

- [ ] 3. Design and implement AI processing prompt
- [x] 3.1 Create comprehensive prompt with content analysis instructions
  - Define role and task context ("You are a study document generator")
  - Add critical constraints section (no external knowledge, preserve source fidelity)
  - Include processing steps for identifying subject and chapter from header
  - Add instructions to filter glossary sections ("TOME NOTA", "CONEXÕES", "Glossário")
  - Define topic extraction patterns (definitions, key terms, dates, names, events)
  - Add example identification rules (look for "Ex:", "Exemplo:", "por exemplo")
  - Include hierarchy inference guidance using context and proximity signals
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 7.1_

- [x] 3.2 Add condensation and formatting rules to prompt
  - Define condensation rules for "Topic \- Definition" format
  - Add grouping instructions for related facts under common topics
  - Include preservation rules for key terms, names, dates, specific facts
  - Add example formatting instructions ("Ex:" prefix for sub-bullets)
  - Define causal relationship preservation (connectors like "causando", "levando a")
  - Include comparative structure preservation ("diferente de", "similar a")
  - Add sequential item formatting as nested bullets
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9_

- [ ] 3.3 Define exact format specification in prompt
  - Specify document structure (# Subject, ## Cap. N)
  - Define main topic format: plain text (no asterisk), "Topic \- Definition"
  - Define subtopic format: start with `* `, "* Topic \- Definition"
  - Specify indentation rules (exactly 2 spaces per nesting level)
  - Mandate separator usage (always `\-`, backslash-dash)
  - Prohibit bold (`**text**`), italic (`*text*`), other formatting
  - Add parenthetical preservation rule: "(ou Alternative)"
  - Include diacritic preservation list: á, â, ã, é, ê, í, ó, ô, õ, ú, ç
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10, 7.2, 7.3, 7.4_

- [ ] 3.4 Add validation checklist to prompt
  - Include pre-output validation steps for AI to follow
  - Check all topics use `\-` separator
  - Verify no bold or italic formatting present
  - Validate indentation consistency (2-space increments)
  - Confirm main topics have no leading `*`
  - Verify subtopics all start with `* `
  - Add instruction to return only formatted Markdown (no preamble)
  - _Requirements: 6.7_

- [ ] 4. Implement prompt invocation and output processing
- [ ] 4.1 Integrate AI processing into skill workflow
  - Embed complete prompt template in skill markdown
  - Inject raw text content into prompt at designated location
  - Invoke AI processing with full prompt and input text
  - Capture formatted Markdown output from AI response
  - Handle AI processing failures with appropriate error messages
  - Implement fallback for ambiguous or complex text structures
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9, 3.10, 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.8, 4.9, 5.1, 5.2, 5.3, 5.4, 5.5, 5.6, 7.1, 7.2, 7.3, 7.4_

- [ ] 4.2 Generate user summary and success feedback
  - Extract subject from AI output (parse # heading)
  - Extract chapter number from AI output (parse ## Cap. N)
  - Count main topics (lines without leading `*`)
  - Display summary with subject, chapter, and topic count
  - Show full output file path on successful write
  - Format success message for terminal display
  - _Requirements: 6.4, 6.6_

- [ ] 5. Implement error handling and edge cases
- [ ] 5.1 Add comprehensive error handling for all failure modes
  - Handle file not found with descriptive path-specific message
  - Display warning for files under 50 characters with actual count
  - Show error for files over 100KB with splitting suggestion
  - Handle read failures with permission/system error details
  - Handle write failures with disk space/permission guidance
  - Implement "no subject found" error with header guidance
  - Convert "no chapter found" to warning (allow formatting without chapter)
  - Add "empty after filtering" error for glossary-only documents
  - Add AI processing failure error with simplification suggestion
  - Ensure no partial output created on any failure path
  - _Requirements: 1.3, 1.4, 6.7_

- [ ] 6. Testing and validation
- [ ] 6.1 Test skill with various input formats and content types
  - Test with clear subject/chapter headers → verify extraction
  - Test with various definition patterns → verify topic extraction
  - Test with nested topic structures → verify proper indentation
  - Test with "TOME NOTA", "CONEXÕES", "Glossário" sections → verify filtering
  - Test with Portuguese text containing diacritics → verify preservation
  - Test with parentheticals "(ou Alternative)" → verify exact reproduction
  - Test with examples ("Ex:", "Exemplo:") → verify "Ex:" prefix formatting
  - Test with errors/unclear terms → verify no external knowledge added
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.7, 3.4, 3.5, 3.6, 3.8, 4.5, 5.1, 5.2, 5.3, 5.4, 5.5, 7.3_

- [ ] 6.2 Test file I/O operations and error scenarios
  - Test with missing file → verify error message with path
  - Test with empty file → verify warning with character count
  - Test with file under 50 chars → verify warning
  - Test with file over 50KB → verify warning but processing continues
  - Test with file over 100KB → verify graceful failure
  - Test with existing output file → verify overwrite confirmation prompt
  - Test output file naming: `chapter1.txt` → `chapter1_study_notes.md`
  - _Requirements: 1.3, 1.4, 6.2, 6.3, 6.5_

- [ ] 6.3 Validate output format compliance
  - Verify output is valid Markdown (# heading, ## heading, * lists)
  - Check all nested bullets use exactly 2 spaces per level
  - Verify all topics use `\-` separator (not `-` or other variants)
  - Scan for prohibited formatting: `**bold**`, `*italic*`, `__text__`
  - Compare output structure against reference examples
  - Verify main topics have no leading `*`
  - Verify subtopics all start with `* `
  - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.9_

- [ ] 6.4 Test command interface and help documentation
  - Test invocation without arguments → verify help text displayed
  - Test invocation with `--help` flag → verify help text displayed
  - Test invocation with `-h` flag → verify help text displayed
  - Verify help text includes usage instructions and examples
  - Verify skill appears in Claude Code skill listings
  - Test command with valid file path → verify successful execution
  - _Requirements: 1.2, 8.2, 8.3, 8.5_

---

## Requirements Coverage Summary

All 8 requirements with 48 acceptance criteria are covered by the tasks above:

- **Requirement 1** (Command Interface): Tasks 1.1, 1.2, 2.1, 5.1, 6.2, 6.4
- **Requirement 2** (Content Extraction): Tasks 3.1, 4.1, 6.1
- **Requirement 3** (Output Structure): Tasks 3.3, 4.1, 6.3
- **Requirement 4** (Content Condensation): Tasks 3.2, 4.1, 6.1
- **Requirement 5** (Source Fidelity): Tasks 3.1, 4.1, 6.1
- **Requirement 6** (Output Generation): Tasks 2.1, 2.2, 4.2, 5.1, 6.2
- **Requirement 7** (Multi-Language): Tasks 3.1, 3.3, 4.1, 6.1
- **Requirement 8** (Skill Integration): Tasks 1.1, 6.4

---

## Task Execution Notes

- Tasks marked with `(P)` can be executed in parallel with other parallel tasks
- Tasks 1.1 and 1.2 are parallel-capable (independent setup work)
- Task 2 depends on Task 1 completion (needs skill structure)
- Tasks 3.1-3.4 build the prompt incrementally and should be done sequentially
- Task 4 depends on Tasks 2 and 3 (needs I/O and prompt ready)
- Task 5 can begin after Task 2 (error handling for file operations)
- Task 6 (testing) should be done after all implementation tasks complete
