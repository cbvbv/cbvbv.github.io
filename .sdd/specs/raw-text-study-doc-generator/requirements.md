# Requirements Document

## Project Description (Input)
lets make a claude code command that takes in a text file (converted from PDF, with raw text) of a specific subject and chapter in and from that (and only that, no external knowledge or resources), generates a study docs in the same style as the examples in super revisão.md. Example raw file in at raw_chapter.txt

## Introduction
This specification defines a Claude Code command skill that transforms raw text files (extracted from PDF educational materials) into structured study documents. The command will analyze plain text chapter content and generate concise study notes following the hierarchical, bullet-point style exemplified in "super revisão.md". The input text has no formatting (no bold, markdown, or styling) and is purely plain text with natural paragraph structure. The system must operate exclusively from the input text without using external knowledge sources, ensuring fidelity to the source material.

## Requirements

### Requirement 1: Command Interface
**Objective:** As a student, I want to invoke a simple command with a raw text file path, so that I can quickly generate study notes from my educational materials.

#### Acceptance Criteria
1. The Study Doc Generator shall accept a file path parameter pointing to a raw text file
2. When the user invokes the command without parameters, the Study Doc Generator shall display usage instructions including expected file format and output location
3. If the specified file does not exist, then the Study Doc Generator shall display an error message indicating the file path could not be found
4. If the file is empty or contains fewer than 50 characters, then the Study Doc Generator shall display a warning that the input appears too short for meaningful processing
5. The Study Doc Generator shall execute as a Claude Code skill command (invocable via `/` prefix)

### Requirement 2: Content Extraction and Analysis
**Objective:** As a student, I want the system to identify key concepts and structure from raw chapter text, so that important information is preserved in the study notes.

#### Acceptance Criteria
1. When processing raw text input, the Study Doc Generator shall identify the subject and chapter information from the document header
2. The Study Doc Generator shall extract main topics and subtopics by analyzing section headings and structural patterns in the plain text
3. The Study Doc Generator shall identify definitions, key terms, and their explanations from the source text
4. When encountering glossary sections or sidebar information (e.g., "TOME NOTA", "CONEXÕES", "Glossário"), the Study Doc Generator shall skip these sections and not include them in the output
5. The Study Doc Generator shall extract key dates, names, events, and processes mentioned in the text
6. The Study Doc Generator shall not add information, facts, or explanations beyond what is explicitly stated in the input text
7. The Study Doc Generator shall process plain text input without assuming any pre-existing formatting (no bold, italic, or markdown in the source)

### Requirement 3: Study Document Structure and Formatting
**Objective:** As a student, I want the generated study document to follow the hierarchical format of "super revisão.md", so that the notes are consistent with my existing study materials.

#### Acceptance Criteria
1. The Study Doc Generator shall use Markdown format for all output documents
2. The Study Doc Generator shall organize content with a top-level subject heading (# Subject)
3. When a chapter is identified, the Study Doc Generator shall create a second-level heading (## Cap. N) for the chapter
4. The Study Doc Generator shall represent main topics as plain text (no bold) followed by a backslash-dash (\-) and a brief definition on the same line (e.g., "Rococó \- Estilo artístico que se iniciou...")
5. The Study Doc Generator shall represent subtopics as bullet points using asterisk (*) followed by the topic name, backslash-dash (\-), and definition (e.g., "* Contexto Social \- Valorização do racionalismo...")
6. The Study Doc Generator shall use nested bullet points with proper indentation (two spaces per level) to represent hierarchical relationships between concepts
7. When sub-concepts are present, the Study Doc Generator shall indent them under parent topics using proper Markdown list syntax with consistent spacing
8. The Study Doc Generator shall use parenthetical alternatives for terminology (e.g., "Dadaísmo (ou Dadá)", "Terra Primitiva (ou Pré-Biótica)")
9. The Study Doc Generator shall not use bold, italic, or other text formatting for topic names, maintaining plain text throughout
10. The Study Doc Generator shall avoid adding section headers beyond subject and chapter

### Requirement 4: Content Condensation and Clarity
**Objective:** As a student, I want information condensed into concise study notes, so that I can review material efficiently without losing key information.

#### Acceptance Criteria
1. The Study Doc Generator shall summarize paragraph-length explanations into concise statements using the format "Topic \- Definition/explanation"
2. When multiple related facts are present, the Study Doc Generator shall group them under a common topic using nested bullet points
3. The Study Doc Generator shall preserve all key terms, names, dates, and specific facts from the source text
4. The Study Doc Generator shall convert verbose explanations into clear, direct statements while retaining technical accuracy
5. When examples are provided in the source text, the Study Doc Generator shall include them as sub-bullets with "Ex:" prefix (e.g., "Ex: Água quando deixada em ambiente aberto eventualmente desaparece")
6. The Study Doc Generator shall maintain causal relationships and contextual information using connectors from the source language (e.g., "causando", "levando a" for Portuguese)
7. The Study Doc Generator shall avoid redundant information by consolidating repeated concepts
8. The Study Doc Generator shall preserve comparative structures (e.g., "diferente de", "similar a") when contrasting concepts
9. When listing sequential items or steps, the Study Doc Generator shall present them as nested bullet points under the parent concept

### Requirement 5: Source Text Fidelity
**Objective:** As a student, I want the study document to reflect only the content from my source material, so that I study accurate information from my curriculum.

#### Acceptance Criteria
1. The Study Doc Generator shall not incorporate external knowledge or information beyond the input text
2. If terminology or concepts are unclear in the source text, the Study Doc Generator shall preserve the original wording rather than clarifying with external knowledge
3. When specialized vocabulary is used in the source text, the Study Doc Generator shall maintain that vocabulary exactly as written
4. The Study Doc Generator shall not correct factual errors present in the source material
5. If the source text includes parenthetical clarifications or alternative names, the Study Doc Generator shall preserve them
6. The Study Doc Generator shall not add contextual explanations or background information not present in the input text

### Requirement 6: Output Generation and Delivery
**Objective:** As a student, I want the generated study document saved in a predictable location with clear naming, so that I can easily find and organize my study materials.

#### Acceptance Criteria
1. The Study Doc Generator shall create output files in Markdown format (.md extension)
2. When the input file is named `[filename].txt`, the Study Doc Generator shall name the output file `[filename]_study_notes.md`
3. The Study Doc Generator shall save the output file in the same directory as the input file by default
4. When output file generation is complete, the Study Doc Generator shall display the full path to the generated file
5. If an output file already exists at the target location, then the Study Doc Generator shall ask for user confirmation before overwriting
6. The Study Doc Generator shall display a summary showing the subject, chapter, and number of main topics extracted
7. If processing fails for any reason, then the Study Doc Generator shall display a descriptive error message and not create a partial output file

### Requirement 7: Multi-Language Support
**Objective:** As a student studying in different languages, I want the system to preserve the original language of the source material, so that terminology and concepts remain in their proper linguistic context.

#### Acceptance Criteria
1. The Study Doc Generator shall detect and preserve the language of the input text in the output document
2. The Study Doc Generator shall not translate content from one language to another
3. When processing Portuguese text, the Study Doc Generator shall maintain Portuguese-specific characters and diacritics (á, â, ã, é, ê, í, ó, ô, õ, ú, ç)
4. The Study Doc Generator shall preserve language-specific formatting conventions present in the source text

### Requirement 8: Skill Integration
**Objective:** As a developer using Claude Code, I want the study doc generator to integrate as a standard skill, so that it follows the established patterns for command invocation and help documentation.

#### Acceptance Criteria
1. The Study Doc Generator shall be invocable using the `/generate-study-doc` command syntax
2. The Study Doc Generator shall provide a description that appears in skill listings
3. When invoked with `--help` or `-h` flag, the Study Doc Generator shall display usage instructions and examples
4. The Study Doc Generator shall follow Claude Code skill naming conventions and file structure
5. The Study Doc Generator shall be listed in the available skills when users query for help
