# Requirements Document

## Project Description (Input)
make a html site on ./2o that is able to show a obsidian workspace (directory with md files, example folder carrinhos, unrelated with eventual use), and be able to click the Obsidian file links [[Title]], which is not supported in regular markdown. make the workspace directory and start file easily adjustable.

## Introduction
This specification defines requirements for an HTML-based viewer application that renders Obsidian-style markdown files with support for wiki-link navigation. The application will be deployed in the `./2o` directory and enable users to browse and navigate through a workspace of markdown files using clickable `[[Title]]` links, with configurable workspace directory and entry point settings.

## Requirements

### Requirement 1: Markdown File Rendering
**Objective:** As a user, I want the viewer to display markdown files from an Obsidian workspace, so that I can read formatted content in a web browser.

#### Acceptance Criteria
1. When a markdown file is selected, the viewer shall parse and render standard markdown syntax including headings, lists, code blocks, emphasis, links, and images
2. When the viewer initializes, the viewer shall load markdown files from the configured workspace directory
3. When a markdown file is loaded, the viewer shall display its rendered HTML content
4. When rendering markdown content, the viewer shall preserve formatting including nested structures and multi-line blocks

### Requirement 2: Wiki-Link Navigation
**Objective:** As a user, I want to click on Obsidian wiki-links `[[Title]]`, so that I can navigate between interconnected notes.

#### Acceptance Criteria
1. When the viewer parses markdown content, the viewer shall identify all wiki-link patterns matching `[[Title]]` syntax
2. When a wiki-link references an existing file, the viewer shall convert it to a clickable HTML element
3. When a user clicks a wiki-link to an existing file, the viewer shall navigate to and display the corresponding markdown file
4. If a wiki-link target file does not exist, the viewer shall render the link as grayed out and non-clickable
5. When a user hovers over a non-existent wiki-link, the viewer shall display a tooltip indicating the file does not exist
6. When the viewer encounters `[[Target|Display Text]]` syntax, the viewer shall render the display text while linking to the target file
7. When the viewer encounters wiki-links with path notation (e.g., `[[folder/note]]`), the viewer shall resolve the target file from subdirectories
8. When a user navigates to a file, the viewer shall update the URL with a file parameter (e.g., `?file=title`)
9. When the viewer loads with a file parameter in the URL, the viewer shall display the specified markdown file
10. If the URL parameter specifies a non-existent file, the viewer shall display an error message indicating the file was not found

### Requirement 3: Workspace Configuration
**Objective:** As a user, I want to easily configure the workspace directory and start file, so that I can point the viewer to different collections of markdown files without code changes.

#### Acceptance Criteria
1. The viewer shall provide a configuration mechanism for specifying the workspace directory path
2. The viewer shall provide a configuration mechanism for specifying the initial start file to display
3. When the viewer initializes without a URL file parameter, the viewer shall load the configured start file from the configured workspace directory
4. The viewer shall enable configuration modification through a JavaScript configuration object or JSON file
5. If the configured workspace directory does not exist, the viewer shall display an error message indicating the workspace directory was not found
6. If the configured start file does not exist, the viewer shall display an error message indicating the start file was not found

### Requirement 4: File System Integration
**Objective:** As a user, I want the viewer to work with a directory of markdown files, so that I can use it with any Obsidian-compatible workspace structure.

#### Acceptance Criteria
1. When resolving file paths, the viewer shall read markdown files from the local filesystem relative to the configured workspace directory
2. When scanning the workspace, the viewer shall support nested directory structures within the workspace
3. When resolving wiki-links, the viewer shall search for matching files within the workspace directory tree
4. When processing file names, the viewer shall handle file names with spaces and special characters
5. When identifying markdown files, the viewer shall support common markdown file extensions (.md, .markdown)

### Requirement 5: Deployment and Accessibility
**Objective:** As a user, I want the HTML site deployed in the `./2o` directory, so that I can access it through a web browser.

#### Acceptance Criteria
1. The viewer application shall be located in the `./2o` directory of the repository
2. When a user opens the main HTML file in a web browser, the viewer shall be accessible and functional
3. The viewer shall function without requiring a web server for basic static file operation
4. When loading resources, the viewer shall use relative file paths compatible with local file access

### Requirement 6: User Interface
**Objective:** As a user, I want a clear and functional interface, so that I can easily navigate and read markdown content.

#### Acceptance Criteria
1. When a note is displayed, the viewer shall show the current note's title or filename
2. When a user hovers over a clickable wiki-link, the viewer shall provide visual feedback indicating interactivity
3. When rendering markdown content, the viewer shall display it in a readable format with appropriate spacing and typography
4. If a file fails to load, the viewer shall display an error message to the user describing the failure
5. When displaying content exceeding viewport height, the viewer shall provide vertical scrolling capability
