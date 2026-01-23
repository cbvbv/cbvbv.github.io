# Implementation Plan

## Overview
This document outlines the implementation tasks for the Obsidian HTML Viewer feature. Tasks are organized to build incrementally from foundational infrastructure through to complete integration. Parallel-capable tasks are marked with `(P)`.

## Tasks

- [x] 1. Set up project structure and dependencies
- [x] 1.1 (P) Create HTML file with basic structure
  - Create `./2o/index.html` with viewport meta tags and semantic HTML structure
  - Include container elements for title, content, and error display
  - Add basic CSS reset and typography styles
  - _Requirements: 5.1, 5.2_

- [x] 1.2 (P) Integrate marked.js library
  - Add marked.js v17.0.1 via CDN or npm bundle
  - Configure library initialization with GFM and breaks enabled
  - Verify library loads and basic markdown parsing works
  - _Requirements: 1.1_

- [x] 2. Implement configuration management
- [x] 2.1 Create configuration object and validation
  - Define inline JavaScript configuration object with workspaceDir and startFile properties
  - Implement validation function to check configuration properties are non-empty strings
  - Display error overlay with actionable message if configuration is invalid
  - _Requirements: 3.1, 3.2, 3.4, 3.5, 3.6_

- [ ] 3. Build file loading infrastructure
- [x] 3.1 Implement file loader with fetch API
  - Create loadFile function that uses fetch with relative paths from workspace directory
  - Handle response status codes and map to appropriate error types (not_found, access_denied, network_error)
  - Return result object with success/failure status and data or error
  - Support both .md and .markdown file extensions
  - _Requirements: 1.2, 4.1, 4.4, 4.5_

- [x] 3.2 Implement path resolver for wiki-links
  - Create resolvePath function that converts wiki-link text to candidate file paths
  - Handle subdirectory paths with forward slash separators
  - Generate array of candidate paths with different extensions to try sequentially
  - Normalize paths for case-insensitive comparison while preserving display format
  - _Requirements: 2.7, 4.2, 4.3, 4.4_

- [x] 3.3 Add file existence checking with caching
  - Implement fileExists function that checks if file is accessible via fetch HEAD request
  - Create Map-based cache to store file existence results
  - Provide getCachedValidity function for synchronous access to cached results
  - Lazy validation strategy: only check files referenced in current document
  - _Requirements: 2.4, 4.1_

- [ ] 4. Implement wiki-link processing
- [x] 4.1 Create wiki-link parser and transformer
  - Implement regex-based parser to identify all wiki-link patterns including basic and alias syntax
  - Extract target filename and optional display text from matches
  - Transform wiki-links to standard markdown link syntax before rendering
  - Handle subdirectory paths and special characters in link targets
  - _Requirements: 2.1, 2.2, 2.6, 2.7_

- [ ] 4.2 Integrate link validation with styling
  - Call file existence checker for each extracted wiki-link target
  - Generate markdown links with special CSS class for broken links
  - Preserve original wiki-link text when target file does not exist
  - Store link metadata for tooltip generation
  - _Requirements: 2.2, 2.4, 2.5_

- [ ] 5. Build markdown rendering layer
- [ ] 5.1 Create markdown renderer with marked.js
  - Implement render function that calls marked.parse with preprocessed markdown
  - Configure marked options for GFM support, line breaks, and header IDs
  - Wrap rendering in try-catch to handle parse errors gracefully
  - Return HTML string or error object with message
  - _Requirements: 1.1, 1.3, 1.4_

- [ ] 5.2* Add rendering validation tests
  - Test standard markdown syntax rendering for headings, lists, code blocks, emphasis, links, images
  - Verify nested structures and multi-line blocks preserve formatting
  - Test error handling for malformed markdown
  - _Requirements: 1.1, 1.4_

- [ ] 6. Implement UI rendering and interactions
- [ ] 6.1 Create UI renderer for content display
  - Implement renderContent function that updates DOM with HTML and title
  - Use innerHTML for content insertion and textContent for title to prevent XSS
  - Apply appropriate CSS classes for styling and scrolling behavior
  - Add scrollToTop function to reset viewport on navigation
  - _Requirements: 6.1, 6.3, 6.5_

- [ ] 6.2 Add hover effects and visual feedback for links
  - Apply CSS hover styles for clickable wiki-links
  - Implement tooltip display for broken links showing "File not found" message
  - Use data attributes to store link metadata for tooltip content
  - Ensure visual distinction between valid and broken links
  - _Requirements: 2.5, 6.2_

- [ ] 6.3 Implement error display system
  - Create showError function that displays error messages with title, message, and optional actions
  - Style error overlay with appropriate severity levels (warning, error, info)
  - Implement clearErrors function to dismiss error messages
  - Map technical error types to user-friendly messages
  - _Requirements: 3.5, 3.6, 6.4_

- [ ] 7. Build URL routing system
- [ ] 7.1 Implement URL parameter routing
  - Create getCurrentFile function using URLSearchParams to parse query string
  - Implement navigateToFile function that updates URL using history.pushState
  - Handle URI encoding/decoding for filenames with special characters
  - Fall back to configured start file when no URL parameter present
  - _Requirements: 2.8, 2.9, 3.3_

- [ ] 7.2 Add browser navigation support
  - Implement popstate event listener for browser back/forward buttons
  - Register route change handler that reloads content when URL changes
  - Ensure URL stays in sync with displayed content
  - Test bookmark and refresh behavior
  - _Requirements: 2.8_

- [ ] 7.3 Handle invalid URL parameters
  - Detect when URL parameter specifies non-existent file
  - Display clear error message with file name
  - Provide action button to navigate back to start file
  - _Requirements: 2.10_

- [ ] 8. Implement application initialization flow
- [ ] 8.1 Create initialization sequence
  - Load and validate configuration on application start
  - Parse URL query parameter to determine initial file to load
  - Attempt to load file and handle both success and failure cases
  - Process content through wiki-link transformer and markdown renderer
  - Display rendered content or error message based on outcome
  - _Requirements: 1.2, 2.9, 3.3_

- [ ] 8.2 Wire up click handlers for wiki-link navigation
  - Add event delegation for clicks on wiki-links in rendered content
  - Extract target filename from link href attribute
  - Call navigateToFile to trigger URL update and content reload
  - Prevent default link behavior to avoid page refresh
  - _Requirements: 2.3_

- [ ] 9. Integration and end-to-end testing
- [ ] 9.1 Create test workspace with sample markdown files
  - Set up example workspace directory with nested structure
  - Create sample markdown files with various wiki-link patterns
  - Include files with subdirectories, aliases, and broken links
  - Configure application to use test workspace
  - _Requirements: 4.2_

- [ ] 9.2 Test complete navigation flow
  - Load application with default start file and verify rendering
  - Click various wiki-links and verify navigation works
  - Confirm URL parameter updates correctly
  - Test browser back/forward navigation
  - Verify broken links display with correct styling and tooltips
  - _Requirements: 2.3, 2.4, 2.5, 2.8_

- [ ] 9.3 Test error scenarios
  - Test invalid configuration handling
  - Test missing file error display
  - Test invalid URL parameter handling
  - Verify all error messages are user-friendly and actionable
  - _Requirements: 3.5, 3.6, 2.10, 6.4_

- [ ] 9.4* Browser compatibility testing
  - Test file:// protocol access in Chrome, Firefox, Safari
  - Verify fetch API behavior with relative paths
  - Test History API and query parameter routing
  - Document any browser-specific limitations
  - _Requirements: 5.2, 5.3, 5.4_

- [ ] 10. Polish and final integration
- [ ] 10.1 Add CSS styling for complete UI
  - Style title header with clear typography
  - Style content area with readable font, line height, and spacing
  - Style wiki-links with distinct colors for valid vs broken links
  - Style error messages with appropriate visual hierarchy
  - Add responsive scrolling for long content
  - _Requirements: 6.1, 6.2, 6.3, 6.5_

- [ ] 10.2 Add loading states and transitions
  - Show loading indicator during file fetch operations
  - Add smooth transitions for content updates
  - Prevent multiple simultaneous navigation requests
  - Handle rapid navigation clicks gracefully
  - _Requirements: 6.3_

- [ ] 10.3 Final integration verification
  - Test complete workflow from configuration to navigation
  - Verify all components integrate correctly
  - Confirm all requirements are satisfied
  - Test with example workspace (carrinhos folder)
  - _Requirements: All_
