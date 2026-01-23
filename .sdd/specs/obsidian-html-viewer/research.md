# Research & Design Decisions

---
**Purpose**: Capture discovery findings, architectural investigations, and rationale that inform the technical design.

**Usage**:
- Log research activities and outcomes during the discovery phase.
- Document design decision trade-offs that are too detailed for `design.md`.
- Provide references and evidence for future audits or reuse.
---

## Summary
- **Feature**: `obsidian-html-viewer`
- **Discovery Scope**: New Feature (Greenfield)
- **Key Findings**:
  - Marked.js v17.0.1 provides robust markdown parsing with extension API for custom wiki-link handling
  - Browser File API with fetch() for local file access is feasible for static site deployment
  - Client-side architecture with URL-based routing eliminates need for backend server
  - Custom renderer extension required to transform wiki-links before markdown processing

## Research Log

### Markdown Parsing Library Selection
- **Context**: Need to render standard markdown with ability to extend parser for wiki-link syntax
- **Sources Consulted**:
  - [marked.js official documentation](https://marked.js.org)
  - [marked.js GitHub repository](https://github.com/markedjs/marked) - v17.0.1 (Nov 2025)
- **Findings**:
  - Marked.js is lightweight (~40KB), fast, and widely adopted
  - Supports 100% Markdown 1.0, 98% CommonMark 0.31, 97% GFM 0.29
  - Provides `marked.use()` extension API for custom tokenizers and renderers
  - No built-in sanitization - requires DOMPurify for user-generated content (not needed for trusted local files)
  - Browser-compatible via CDN or npm/ESM imports
- **Implications**:
  - Marked.js selected as primary markdown parser
  - Extension API enables pre-processing step to convert `[[wiki-links]]` to standard markdown links
  - Performance adequate for typical Obsidian workspace sizes (hundreds of notes)

### Browser File System Access
- **Context**: Static HTML site needs to load markdown files from local filesystem without web server
- **Sources Consulted**:
  - MDN Web Docs on File API
  - Browser compatibility research for local file access
- **Findings**:
  - Modern browsers restrict `file://` protocol access via fetch() due to CORS security
  - Three viable approaches for static deployment:
    1. **FileReader API** with file input selector - requires user to select files manually (not suitable)
    2. **File System Access API** (`showDirectoryPicker`) - Chrome/Edge only, requires user permission (not suitable for seamless experience)
    3. **Fetch API with relative paths** - works when served via web server or `file://` protocol if files are in same directory tree
  - Browser security allows fetch() of relative paths from the origin (including `file://` origin in same directory)
- **Implications**:
  - Use fetch() with relative paths from configured workspace directory
  - Files must be organized in accessible directory structure relative to HTML file
  - Example folder structure: `./2o/index.html` and `./2o/workspace/` for markdown files

### Wiki-Link Syntax Parsing
- **Context**: Need to identify and convert `[[Title]]` and `[[Target|Display]]` syntax to clickable links
- **Sources Consulted**:
  - Obsidian documentation on wiki-link syntax
  - remark-wiki-link plugin analysis
- **Findings**:
  - Wiki-link patterns:
    - Basic: `[[Page Name]]`
    - With alias: `[[Target|Display Text]]`
    - With path: `[[folder/note]]`
    - With heading: `[[Page#Heading]]` (optional for future)
  - Regex pattern for basic + alias: `/\[\[([^|\]]+)(?:\|([^\]]+))?\]\]/g`
  - Can be implemented as pre-processing step before markdown parsing or as marked.js extension
- **Implications**:
  - Implement custom preprocessor function to transform wiki-links to markdown links before parsing
  - Convert `[[Title]]` → `[Title](workspace/Title.md)`
  - Convert `[[Target|Display]]` → `[Display](workspace/Target.md)`
  - Handle broken links by checking file existence and applying disabled styling

### URL-Based Navigation Strategy
- **Context**: Single-page application needs routing without backend server
- **Sources Consulted**:
  - HTML5 History API patterns
  - Query parameter routing for static sites
- **Findings**:
  - URL query parameters (`?file=note-name`) work with static sites and `file://` protocol
  - `window.location.search` provides query parameter access
  - `history.pushState()` updates URL without page reload
  - Bookmark-friendly: users can share/bookmark specific notes
- **Implications**:
  - Use query parameter routing: `index.html?file=note-name`
  - On page load, check for `?file=` parameter and load specified file
  - On wiki-link click, update URL parameter and load new content without page refresh
  - Fall back to configured start file when no parameter present

## Architecture Pattern Evaluation

| Option | Description | Strengths | Risks / Limitations | Notes |
|--------|-------------|-----------|---------------------|-------|
| Pure Client-Side SPA | All logic in browser, no backend, fetch() for file loading | Simple deployment, no server needed, works offline | Limited to same-origin files, no server-side search | Selected - aligns with static site requirement |
| Static Site Generator | Pre-build all pages with links resolved | Fast, pre-rendered HTML, no runtime processing | Requires build step, loses dynamic workspace switching | Rejected - conflicts with "easily adjustable" requirement |
| Minimal Node Server | Express/Fastify serving files + API | Full filesystem access, server-side search capability | Requires server deployment, not static | Rejected - requirement specifies static HTML site |

## Design Decisions

### Decision: Client-Side Single-Page Application Architecture
- **Context**: Requirement specifies static HTML site in `./2o` directory with easy configuration changes
- **Alternatives Considered**:
  1. Static site generator (Jekyll, Hugo) - requires build step
  2. Node.js server - requires server deployment
  3. Pure client-side SPA - selected
- **Selected Approach**: Single HTML file with vanilla JavaScript loading markdown files via fetch()
- **Rationale**:
  - Meets "no server required" constraint (Req 5.3)
  - Enables "easily adjustable" configuration via JavaScript config object (Req 3.4)
  - Simple deployment and maintenance
  - Works with `file://` protocol for local viewing
- **Trade-offs**:
  - Benefits: Zero deployment complexity, instant configuration changes, offline-capable
  - Compromises: Limited to same-origin file access, no backend search/indexing
- **Follow-up**: Verify fetch() behavior with relative paths in target deployment environment

### Decision: Wiki-Link Preprocessing Before Markdown Parsing
- **Context**: Standard markdown parsers don't recognize `[[wiki-link]]` syntax
- **Alternatives Considered**:
  1. Custom marked.js tokenizer extension - complex, deep integration
  2. Regex preprocessing before markdown parsing - selected
  3. Post-processing HTML output - fragile, loses semantic meaning
- **Selected Approach**: Preprocessing function that transforms wiki-links to markdown link syntax before passing to marked.js
- **Rationale**:
  - Simple regex-based transformation
  - Preserves separation of concerns (wiki-link handling vs markdown rendering)
  - Allows file existence checking before rendering
  - Easier to test and maintain
- **Trade-offs**:
  - Benefits: Simple implementation, clear separation, testable
  - Compromises: Requires two-pass processing (preprocessing + markdown parsing)
- **Follow-up**: Implement file existence cache to optimize broken link detection

### Decision: URL Query Parameter Navigation
- **Context**: Need client-side routing for note navigation without page refresh
- **Alternatives Considered**:
  1. Hash-based routing (`#/note`) - works but less clean URLs
  2. Query parameter routing (`?file=note`) - selected
  3. History API with path routing - doesn't work with `file://` protocol
- **Selected Approach**: Use `?file=note-name` query parameter with History API
- **Rationale**:
  - Compatible with both `file://` and `http://` protocols
  - Bookmark-friendly and shareable URLs
  - Simple URL parsing with URLSearchParams
  - Supports browser back/forward navigation
- **Trade-offs**:
  - Benefits: Universal compatibility, clean implementation, bookmark support
  - Compromises: Slightly less aesthetic than path-based routing
- **Follow-up**: Handle edge cases like special characters in filenames

### Decision: Inline Configuration Object
- **Context**: Requirement 3.4 specifies "easily modifiable configuration"
- **Alternatives Considered**:
  1. External JSON file - requires additional fetch, async complexity
  2. Inline JavaScript config object - selected
  3. HTML data attributes - less structured
- **Selected Approach**: JavaScript configuration object at top of main script
- **Rationale**:
  - Single-file modification for configuration changes
  - No async loading overhead
  - Familiar JavaScript syntax
  - Easy to document with comments
- **Trade-offs**:
  - Benefits: Zero latency, simple editing, no external dependencies
  - Compromises: Requires editing HTML file (but acceptable for single-file app)
- **Follow-up**: Add validation for required config properties

## Risks & Mitigations

- **Risk**: Browser security policies may block local file access in some browsers
  - **Mitigation**: Document requirement to open via local web server (Python `http.server`, VS Code Live Server) or ensure files are in same directory tree

- **Risk**: Large workspaces (1000+ files) may impact performance for file existence checking
  - **Mitigation**: Implement lazy file index with caching, only scan on first access

- **Risk**: File name conflicts (case sensitivity) across operating systems
  - **Mitigation**: Normalize filenames to lowercase for comparison, document case-sensitive behavior

- **Risk**: Special characters in filenames may break URL routing
  - **Mitigation**: Use `encodeURIComponent()` for URL parameters and `decodeURIComponent()` when resolving files

## References

- [Marked.js Documentation](https://marked.js.org) - Official API and usage guide
- [Marked.js GitHub](https://github.com/markedjs/marked) - Source code, current version v17.0.1
- [MDN File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API) - Browser file access standards
- [MDN History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) - URL management for client-side routing
- [Obsidian Help - Internal Links](https://help.obsidian.md/Linking+notes+and+files/Internal+links) - Wiki-link syntax specification
