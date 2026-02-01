---
description: Generate structured study notes from raw PDF text files
allowed-tools: Read, Write, Glob
argument-hint: <input-file-path>
---

# Study Document Generator

<background_information>
- **Mission**: Transform raw educational text files (extracted from PDFs) into structured Markdown study documents
- **Success Criteria**:
  - Input file successfully parsed and analyzed
  - Structured study notes generated following specific format
  - Output saved with correct naming convention
  - Source content preserved without external knowledge added
</background_information>

<instructions>
## Core Task
Generate structured study notes from raw text file at **$ARGUMENTS**.

## Command Usage

### Invocation
```
/generate-study-doc <input-file-path>
```

**Examples**:
- `/generate-study-doc raw_chapter.txt`
- `/generate-study-doc path/to/chapter1.txt`
- `/generate-study-doc ~/documents/history_chapter_5.txt`

### Help
```
/generate-study-doc --help
/generate-study-doc -h
```

## Expected Input Format

**Input File Requirements**:
- Plain text file (`.txt` extension)
- Extracted from PDF educational materials
- Minimum 50 characters
- UTF-8 encoded
- Contains unformatted educational content with natural paragraph structure

**Typical Structure**:
- Subject and chapter information in header/opening lines
- Section headings and topic descriptions
- Definitions, key terms, names, events
- May include sidebar sections (TOME NOTA, CONEXÕES, Glossário)

## Output

**Output File**:
- Markdown format (`.md` extension)
- Naming: `<input-basename>_study_notes.md`
- Location: Same directory as input file
- Structure: Hierarchical bullet-point format with specific styling

**Output Format Example**:
```markdown
# Subject Name
## Cap. N

Main Topic \- Brief definition of the main topic
* Subtopic \- Definition or explanation
  * Nested concept \- Additional detail
* Another Subtopic \- Explanation
```

## Processing Notes

- **Language Preservation**: The tool automatically detects and preserves the source language (Portuguese, English, Spanish, etc.)
- **Diacritics**: All special characters (á, é, í, ó, ú, ç, ã, õ, etc.) are preserved exactly
- **Source Fidelity**: Only content from the input file is used - no external knowledge added
- **Glossary Filtering**: Sidebar sections (TOME NOTA, CONEXÕES, Glossário) are automatically excluded
- **Performance**: Small files (<10KB) process in ~10 seconds; larger files may take up to 60+ seconds

## Execution Steps

### Step 1: Argument Parsing and Validation

**Parse Arguments**:
- Extract file path from `$ARGUMENTS` variable
- Trim whitespace from the argument
- Check for help flags: `--help`, `-h`

**Validation Logic**:
```
IF $ARGUMENTS is empty OR $ARGUMENTS equals "--help" OR $ARGUMENTS equals "-h" THEN
  Display the Command Usage section above
  Stop execution
END IF

SET input_file_path = $ARGUMENTS (trimmed of leading/trailing whitespace)

IF input_file_path is empty THEN
  Display error: "Error: No input file path provided."
  Display: "Usage: /generate-study-doc <input-file-path>"
  Display: "Run '/generate-study-doc --help' for more information."
  Stop execution
END IF
```

**Handle Invalid Patterns**:
- Empty string or whitespace-only: Show usage hint and stop
- Help flags (`--help`, `-h`): Show full Command Usage documentation and stop
- Valid path: Continue to Step 2

### Step 2: File Validation and Reading

**Read Input File**:
Using the validated `input_file_path` from Step 1, attempt to read the file.

**File Reading Logic**:
```
TRY
  Use Read tool to load content from input_file_path
CATCH file_not_found
  Display error: "Input file not found at '<input_file_path>'. Please verify the file path and try again."
  Stop execution
CATCH permission_denied
  Display error: "Unable to read file '<input_file_path>'. Check file permissions."
  Stop execution
CATCH other_read_error
  Display error: "Unable to read file '<input_file_path>'. Expected plain text (.txt) file."
  Stop execution
END TRY

SET file_content = loaded content
SET file_size_chars = length of file_content
SET file_size_kb = file_size_chars / 1024
```

**File Size Validation**:
```
IF file_size_chars < 50 THEN
  Display warning: "⚠️ Warning: Input file contains fewer than 50 characters. Results may be incomplete."
  Display: "Minimum: 50 chars, Found: <file_size_chars> chars"
  Display: "Proceeding with processing..."
END IF

IF file_size_kb > 100 THEN
  Display error: "❌ Error: Input file too large (>100KB). Please split into smaller chapters."
  Display: "File size: <file_size_kb> KB"
  Stop execution
END IF

IF file_size_kb > 50 THEN
  Display info: "ℹ️ Note: Large file detected (<file_size_kb> KB). Processing may take 60+ seconds."
END IF
```

**Encoding Validation**:
```
TRY
  Verify file_content is valid UTF-8 text
CATCH encoding_error
  Display error: "Unable to read file '<input_file_path>'. File encoding must be UTF-8."
  Display: "Please convert the file to UTF-8 encoding and try again."
  Stop execution
END TRY
```

**Success Path**:
- File content successfully loaded into `file_content` variable
- File size validated (>= 50 chars, <= 100KB)
- UTF-8 encoding confirmed
- Continue to Step 3

### Step 3: AI Processing

Process the `file_content` from Step 2 to generate structured study notes using the following AI prompt.

**AI Prompt Structure**:

```
You are a study document generator. Your task is to transform raw educational text into structured study notes following a specific format.

## CRITICAL CONSTRAINTS

⚠️ **Source Fidelity**:
- Use ONLY the provided text—NO external knowledge
- Preserve unclear/ambiguous terminology EXACTLY as written
- Maintain specialized vocabulary character-for-character
- Do NOT correct factual errors in the source
- Preserve parenthetical clarifications exactly: "(ou Alternative)"
- Do NOT add context, background, or explanatory information not in the source

## CONTENT ANALYSIS INSTRUCTIONS

### 1. Identify Subject and Chapter
- Look at the document header (first few lines/paragraphs)
- Extract the subject name for the main heading
- Extract chapter number if present (look for "Cap.", "Capítulo", "Chapter", etc.)
- If no clear subject: use the most prominent topic as subject
- If no chapter: omit chapter heading

### 2. Extract Content Elements
Identify and extract: main topics, subtopics, definitions, key terms, names (people/places/organizations), dates, events, processes.

### 3. Filter Glossary Sections
**SKIP these sections entirely** (do not include in output):
- Sections labeled "TOME NOTA" or "Tome nota"
- Sections labeled "CONEXÕES" or "Conexões"
- Sections labeled "Glossário" or "GLOSSÁRIO"
- Sidebar content clearly marked as supplementary

### 4. Detect and Preserve Language
- Automatically detect the source language (Portuguese, English, Spanish, etc.)
- Preserve ALL diacritics exactly: á, â, ã, é, ê, í, ó, ô, õ, ú, ç, ñ, etc.
- Do NOT translate any content
- Maintain language-specific vocabulary and expressions

### 5. Identify Examples
Look for: "Ex:", "Exemplo:", "por exemplo", "for example", "por ejemplo". Format as sub-bullets with "Ex:" prefix.

### 6. Infer Hierarchical Relationships
Determine parent-child relationships using:
- **Context and proximity**: Related sentences/paragraphs likely belong together
- **Paragraph breaks**: Natural boundaries between concepts
- **Causal connectors**: "causando", "levando a", "resultando em", "causing", "leading to"
- **Sequential indicators**: "primeiro", "segundo", "first", "second", "then", "next"
- **Semantic grouping**: Group related facts under common parent topics
- **Source ordering**: Maintain the order concepts appear in the text

### 7. Content Organization Principles
- Extract concepts in the order they appear in the source
- Group related information under common topics
- Maintain logical flow from source document
- Infer structure from content, not from formatting (source has no formatting)

---

INPUT TEXT:
---
{file_content}
---

## CONDENSATION AND FORMATTING RULES

### 8. Summarize Verbose Content
Transform paragraph-length explanations into concise statements:
- Use format: **"Topic \- Definition/explanation"**
- Keep essential meaning, remove filler words
- Maintain technical accuracy
- Example transformation:
  - Source: "O Renascimento foi um período da história europeia que marcou a transição da Idade Média para a modernidade, caracterizado por um renovado interesse pelo conhecimento e pelos valores clássicos."
  - Output: "Renascimento \- Período de transição da Idade Média para a modernidade, caracterizado pelo interesse renovado na aprendizagem clássica"

### 9. Group Related Facts
When multiple related facts appear:
- Identify common parent topic
- Create parent topic line
- List facts as nested sub-bullets under parent
- Maintain logical connections

### 10. Preserve Key Information
**MUST preserve exactly**:
- All key terms and specialized vocabulary
- All proper names (people, places, organizations)
- All specific facts, numbers, statistics
- All technical terminology

### 11. Format Examples
When examples appear in source:
- Place as sub-bullet under the concept they illustrate
- Prefix with "Ex:" (standardized across all languages)
- Format: `  * Ex: <example content>`
- Keep examples concise but complete

### 12. Preserve Causal Relationships
Maintain cause-and-effect connections using connectors:
- Portuguese: "causando", "levando a", "resultando em"
- English: "causing", "leading to", "resulting in"
- Spanish: "causando", "llevando a", "resultando en"
Format: `Topic \- Explanation, causando consequence`

### 13. Preserve Comparative Structures
Maintain comparisons and contrasts:
- Portuguese: "diferente de", "similar a", "ao contrário de"
- English: "different from", "similar to", "unlike"
- Spanish: "diferente de", "similar a", "a diferencia de"
Format: `Topic \- Definition, diferente de other topic`

### 14. Format Sequential Items
When source presents steps, stages, or sequences:
- List as nested bullets under parent concept
- Maintain original order
- Keep numbering if present in source, otherwise use bullets

### 15. Avoid Redundancy
- Do NOT repeat information already stated
- Consolidate similar concepts under single entry
- If source repeats for emphasis, include once with strongest wording

[Format specification continues in task 3.3...]
```

### Step 4: Output File Generation

**Generate Output File Path**:
Extract the base name from the input file path and create the output file path.

**Output Path Logic**:
```
SET input_dir = directory path of input_file_path
SET input_basename = filename without extension from input_file_path
SET output_filename = input_basename + "_study_notes.md"
SET output_file_path = input_dir + "/" + output_filename
```

**Examples**:
- Input: `raw_chapter.txt` → Output: `raw_chapter_study_notes.md`
- Input: `path/to/chapter1.txt` → Output: `path/to/chapter1_study_notes.md`
- Input: `/home/user/history.txt` → Output: `/home/user/history_study_notes.md`

**Check for Existing Output File**:
```
Use Glob tool to check if output_file_path exists

IF output file exists THEN
  Display: "⚠️ Warning: Output file already exists at '<output_file_path>'"
  Prompt user: "Overwrite existing file? [y/N]: "
  Read user response

  IF response is NOT "y" AND NOT "yes" (case-insensitive) THEN
    Display: "Operation cancelled. No files modified."
    Stop execution
  END IF

  Display: "Overwriting existing file..."
END IF
```

**Write Output File**:
Using the `formatted_output` content from Step 3 (AI processing).

```
TRY
  Use Write tool to create output_file_path with formatted_output content
CATCH disk_space_error
  Display error: "Failed to write output file: Insufficient disk space."
  Display: "Check available disk space and try again."
  Stop execution (no partial output created)
CATCH permission_error
  Display error: "Failed to write output file: Permission denied."
  Display: "Output path: <output_file_path>"
  Display: "Check directory permissions and try again."
  Stop execution (no partial output created)
CATCH other_write_error
  Display error: "Failed to write output file: <error_details>"
  Display: "Output path: <output_file_path>"
  Stop execution (no partial output created)
END TRY
```

**Success Output**:
```
Display: "✅ Success! Study notes generated."
Display: ""
Display: "📄 Output file: <output_file_path>"
Display: ""
Display summary (from Step 3 AI processing):
  - Subject: <extracted_subject>
  - Chapter: <extracted_chapter> (if identified)
  - Main topics: <topic_count>
```

**Atomic Operation Guarantee**:
- Write tool creates file atomically (complete or not at all)
- If any error occurs during write, no partial file is created
- User sees clear error message indicating what went wrong

## Critical Constraints
- **Argument validation is mandatory**: Must check arguments before any file operations
- **Help flag priority**: Help flags take precedence over all other processing
- **No empty paths**: Must reject empty or whitespace-only file paths
- **File size limits**: Minimum 50 characters (warning), Maximum 100KB (hard stop)
- **UTF-8 encoding required**: Files must be UTF-8 encoded
- **Fail-fast validation**: Stop immediately on any validation error
- **Output naming convention**: `<input-basename>_study_notes.md` in same directory as input
- **Overwrite protection**: Must prompt user before overwriting existing output files
- **Atomic writes**: Output file must be written completely or not at all (no partial files)

</instructions>

## Tool Guidance

### Step-by-Step Tool Usage

**Step 1 - Argument Parsing**: No tools needed (string manipulation)

**Step 2 - File Reading and Validation**:
1. Use **Read tool** with `input_file_path` parameter
2. Capture file content or error
3. Calculate file size: `file_size_chars = length(content)`, `file_size_kb = file_size_chars / 1024`
4. Validate constraints:
   - If size < 50 chars: Display warning but continue
   - If size > 100KB: Display error and stop
   - If 50KB < size <= 100KB: Display performance note
5. Check UTF-8 encoding validity

**Step 3 - AI Processing**: (Tasks 3.1-3.4 - not yet implemented)

**Step 4 - Output File Operations**:
1. Generate output path: Extract directory and basename from `input_file_path`, append `_study_notes.md`
2. Use **Glob tool** with pattern matching output file path to check if file exists
3. If file exists: Prompt user for overwrite confirmation (accept "y" or "yes" case-insensitive)
4. If user declines: Display cancellation message and stop
5. Use **Write tool** to atomically create output file with `formatted_output` content
6. Handle Write tool errors (disk space, permissions) with specific error messages
7. On success: Display success message with output path and summary (subject, chapter, topic count)

## Output Description

After successful processing, displays:
1. **Subject**: Extracted from document
2. **Chapter**: Chapter number if identified
3. **Topics**: Count of main topics extracted
4. **Output Path**: Full path to generated file

**Format**: Concise summary (under 100 words)

## Safety & Fallback

### Validation Steps
1. Check if `$ARGUMENTS` is provided (show help if not)
2. Validate file path is non-empty string after trimming
3. Validate file exists and is readable (task 2.1)
4. Check file size (minimum 50 chars, maximum 100KB) (task 2.1)
5. Check if output file exists (prompt for confirmation) (task 2.2)
6. Ensure UTF-8 encoding compatibility (task 2.1)

### Error Handling
- **All errors are non-recoverable**: Display clear message and stop
- **No partial output**: If processing fails, do not create output file
- **Atomic operations**: File is written completely or not at all

think
