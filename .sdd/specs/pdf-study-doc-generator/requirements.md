# Requirements Document

## Project Description (Input)
lets make a claude code command that takes in a text file (converted from PDF by pdf-to-text command) and a specific subject and chapter in it and from that (and only that, no external knowledge or resources), generates a study docs in the same style as the examples in super revisão.md.

## Introduction

This specification defines a Claude Code skill/command that processes text files (converted from PDF textbooks) to generate structured study documents. The command extracts content from a specified chapter of a text file on a given subject, then generates study materials formatted in the same hierarchical, bullet-point style as demonstrated in `super revisão.md`. The generator operates exclusively on the text file content without external knowledge or web searches.

The input text file format includes page markers (e.g., `--- Page 42 ---`) to preserve document structure from the original PDF.

The target output style features:
- Chapter-based organization (e.g., "## Cap. 2")
- Plain text topic lines in format "Topic Name (alternative) - Definition"
- Sometimes just topic names alone without definitions
- Plain text continuation lines that add context after topic lines
- Hierarchical bullet-point lists with `*` for supporting details
- 2-space indentation for nested bullet points
- Sub-topics within bullets using "Name - Definition" pattern
- NO bold formatting anywhere in the content

## Requirements

### Requirement 1: Command Interface
**Objective:** As a developer, I want a Claude Code skill command that accepts text file input and parameters, so that I can generate study documents from converted textbook content.

#### Acceptance Criteria
1. The command shall accept a file path to a text file as input
2. The command shall accept a subject name as a parameter (e.g., "Artes", "História", "Biologia")
3. The command shall accept a chapter identifier as a parameter (e.g., "2", "Cap. 3", "Chapter 5")
4. When the command is invoked without required parameters, the command shall display usage instructions with examples
5. The command shall validate that the text file exists before processing
6. If the text file does not exist or is not readable, then the command shall display a clear error message indicating the file path issue

### Requirement 2: Text Content Extraction
**Objective:** As a user, I want the system to extract text content from specific chapters of text files, so that I can generate study materials from that content.

#### Acceptance Criteria
1. The system shall read text files and load content into memory
2. When reading text content, the system shall recognize page markers in the format `--- Page N ---`
3. The system shall identify chapter boundaries based on the provided chapter identifier
4. If the specified chapter cannot be found in the text, then the system shall prompt the user to confirm the chapter exists or provide a list of detected chapters
5. The system shall extract only the content from the specified chapter (from chapter start to next chapter or end of document)
6. The system shall handle multi-page chapters correctly by extracting content across page markers
7. The system shall preserve page marker information for context when analyzing content

### Requirement 3: Content Analysis and Structure Recognition
**Objective:** As a system, I want to analyze extracted text content to identify key concepts and hierarchical relationships, so that I can generate well-structured study documents.

#### Acceptance Criteria
1. The system shall identify main topics, subtopics, and supporting details from the extracted chapter content
2. The system shall recognize key terms, definitions, important dates, historical figures, and significant concepts
3. The system shall identify relationships between concepts (cause-effect, chronological, categorical, hierarchical)
4. The system shall detect contextual information that provides background or explanation for main concepts
5. When analyzing content, the system shall not use external knowledge or web searches
6. The system shall base all analysis exclusively on the content present in the specified text file chapter

### Requirement 4: Study Document Generation
**Objective:** As a user, I want the system to generate study documents in a specific markdown format, so that the output matches the style of existing study materials.

#### Acceptance Criteria
1. The system shall generate output in Markdown format
2. The system shall structure content with a main subject heading (e.g., "# História")
3. The system shall include a chapter heading matching the input chapter (e.g., "## Cap. 2")
4. When presenting main topics, the system shall format them as plain text lines (not bold) with the pattern "Topic Name (alternative if any) - Definition"
5. When a topic has no immediate definition, the system shall present the topic name alone as a plain text line
6. The system shall include plain text continuation lines after topic lines when additional context is needed
7. The system shall organize supporting information using hierarchical bullet-point lists (using `*` for bullets)
8. The system shall use 2-space indentation for nested bullet points
9. When sub-topics appear within bullets, the system shall format them as "Name - Definition" in plain text
10. The system shall NOT use bold formatting anywhere in the generated content
11. The system shall include contextual details inline with concepts (dates, locations, people, significance)
12. The system shall maintain concise, clear language throughout the document

### Requirement 5: Content Fidelity and Constraints
**Objective:** As a user, I want the generated study document to accurately reflect only the text file content, so that I have faithful study materials without external information.

#### Acceptance Criteria
1. The system shall generate content based exclusively on the extracted text chapter
2. The system shall not incorporate external knowledge, web searches, or information from other sources
3. The system shall preserve factual accuracy from the source material
4. When the text content is unclear or ambiguous, the system shall represent it as accurately as possible without adding interpretations
5. The system shall not add examples, analogies, or explanations not present in the source chapter

### Requirement 6: Output Formatting Style
**Objective:** As a user, I want the study document to match the formatting patterns in the reference document, so that all study materials have consistent presentation.

#### Acceptance Criteria
1. The system shall use `#` for the main subject heading
2. The system shall use `##` for chapter headings (e.g., "## Cap. 2")
3. The system shall format main topic lines as plain text (not bold) with the pattern "Topic Name (alternatives) - Definition"
4. The system shall format topic-only lines as plain text without definitions when appropriate
5. The system shall add plain text continuation lines after topics when additional context exists
6. The system shall use unordered lists with `*` bullet points for all supporting details
7. The system shall use 2-space indentation for each nested level in bullet-point hierarchies
8. The system shall format sub-topics within bullets as plain text "Name - Definition"
9. The system shall NOT apply bold, italic, or other text formatting to any content
10. The system shall separate distinct topics with blank lines
11. The system shall maintain consistent spacing and formatting throughout the document

### Requirement 7: Command Output and File Handling
**Objective:** As a user, I want the generated study document to be saved to a file and displayed, so that I can review and use the study materials.

#### Acceptance Criteria
1. When generation completes successfully, the system shall save the study document to a markdown file
2. The system shall use a filename derived from the subject and chapter (e.g., "artes-cap2-study.md")
3. The system shall display the file path where the study document was saved
4. The system shall provide a preview of the generated content in the command output
5. If a file with the same name exists, then the system shall prompt the user to overwrite or choose a different filename
6. The system shall report generation completion status and any warnings encountered during processing

### Requirement 8: Error Handling and User Feedback
**Objective:** As a user, I want clear feedback during processing and helpful error messages, so that I can resolve issues and understand the system's status.

#### Acceptance Criteria
1. When processing begins, the system shall display a status message indicating text file reading has started
2. While extracting content, the system shall provide progress feedback for large documents
3. If text extraction fails, then the system shall display a detailed error message explaining the failure reason
4. If chapter identification fails, then the system shall suggest possible chapter identifiers found in the document
5. When content analysis completes, the system shall report the amount of content extracted (e.g., page count, word count)
6. If generation produces warnings (e.g., minimal content found, unclear structure), then the system shall display these warnings to the user
7. The system shall complete processing within a reasonable time (under 2 minutes for typical textbook chapters)

