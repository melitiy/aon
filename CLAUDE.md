# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ IMPORTANT PERMISSION RULE

**ALWAYS ask for explicit permission before implementing each numbered step (X.Y.Z) in the implementation plan. Wait for user confirmation before proceeding with any code changes.**

## 📊 PROGRESS TRACKING RULE

**After completing each phase or major section:**
1. Update the checkboxes in the implementation plan (mark completed items with [x])
2. Update the Progress Tracking section with current numbers
3. Add a brief note about what was completed in the Version History
4. Use this format for checkbox updates: `- [x]` for completed, `- [ ]` for pending

## Project Overview

This is a GAP Challenge Visual Pattern Puzzle Solver application. It's designed as a single HTML file with inline CSS/JavaScript that uses OpenCV.js for image processing and pattern detection.

## Development Commands

Since this is a single HTML file application with no build process:
- **Run**: Open the HTML file directly in a browser (Chrome 90+, Firefox 88+, Safari 14+, or Edge 90+)
- **Test**: Open in browser and test with sample puzzle images
- **No build/lint commands** - Pure HTML/CSS/JS with OpenCV.js from CDN

## Architecture

### Application Structure
The application will be implemented as a **single HTML file** containing:
- Inline CSS for all styling
- Inline JavaScript for all functionality
- OpenCV.js loaded from CDN (https://docs.opencv.org/4.5.0/opencv.js)
- No other external dependencies

### Core Components (PRD-Aligned)

1. **Template-Based Detection Pipeline**
   - Load user-provided 64x64 PNG templates from /templates/ folder
   - Simple processing: Grayscale → Resize (1000px max) → Template match → Grid map
   - cv.matchTemplate with TM_CCOEFF_NORMED method
   - 65% confidence threshold, 25px overlap filtering
   - Multiple scales: 0.8x, 1.0x, 1.2x of template size

2. **Large Image Handler**
   - `LargeImageHandler` class manages files >5MB
   - Automatic optimization for files up to 25MB
   - Progressive resizing based on file size:
     - <10MB: max 2000px
     - 10-20MB: max 1500px  
     - >20MB: max 1000px with compression

3. **Constraint Satisfaction Solver**
   - `PuzzleSolver` class implements backtracking algorithm
   - Validates row/column uniqueness constraints
   - Performance targets: <100ms for 4×4, <500ms for 5×5

4. **Shape Recognition**
   - Supports 5 shapes: Circle (1), Star (2), Cross (3), Triangle (4), Square (5)
   - Empty cells represented as 0
   - Templates loaded from user-provided 64x64 PNG files

### UI Layout
- Left panel: Original image preview
- Right panel: Interactive grid with detected/solved shapes
- Bottom: Control buttons (Solve, Clear, Reset)
- Input methods: Paste (Ctrl+V), drag-drop, file upload

## Key Implementation Details

- **Memory Management**: Always call `.delete()` on OpenCV Mat objects after use
- **Browser Compatibility**: Requires WebAssembly support for OpenCV.js
- **Performance**: Process images at max 1000px for optimal speed
- **Error Handling**: Graceful fallback to manual input if detection fails
- **Mobile Limits**: Enforce 5MB file size limit on mobile devices

## Development Workflow (PRD Section 5.2)

1. Load OpenCV.js and user templates
2. Convert image to grayscale (cv.cvtColor)
3. Resize to max 1000px (cv.resize)
4. Match templates at multiple scales (0.8x, 1.0x, 1.2x)
5. Apply 65% confidence threshold
6. Map detections to grid with snap-to-grid
7. Display in interactive grid for manual corrections
8. Solve using constraint satisfaction algorithm

## Testing Considerations

- Test with various image sizes (200px to 4000px)
- Verify large file handling (5MB, 10MB, 15MB, 20MB, 25MB)
- Check memory cleanup after processing
- Validate solver with known puzzle solutions
- Test all input methods across browsers

## Template Specifications

### Folder Structure
```
/project-root/
├── CLAUDE.md
├── AON-prd.md
├── puzzle-solver.html (main application)
└── templates/              (Shape templates folder - MUST BE CREATED)
    ├── circle.png         (64x64px - USER MUST PROVIDE)
    ├── star.png          (64x64px - USER MUST PROVIDE)
    ├── cross.png         (64x64px - USER MUST PROVIDE)
    ├── triangle.png      (64x64px - USER MUST PROVIDE)
    └── square.png        (64x64px - USER MUST PROVIDE)
```

### Template Requirements
- **Format**: PNG with transparency
- **Resolutions**: ALL templates must be exactly 64x64 pixels (single fixed size)
- **Color**: Black shape on transparent background
- **Padding**: 2-3 pixels transparent border
- **Style**: Clear, high-contrast silhouettes
- **IMPORTANT**: User must provide all 5 template files before starting

## Detailed Implementation Plan

### 📌 PRD Alignment Note
This implementation follows the PRD requirements:
- **No grid line detection** - Shapes determine grid structure
- **Template matching only** - No edge detection or Hough transforms
- **User selects grid size** - 4×4 or 5×5 chosen upfront
- **Minimal preprocessing** - Only grayscale and resize
- **Snap-to-grid** - Automatic position correction

### ⚠️ CRITICAL EXECUTION ORDER
1. **Phase 1**: Foundation & HTML Structure
2. **Phase 2**: Image Input System
3. **Phase 3.1**: OpenCV Initialization
4. **Phase 3.2**: Template Loading (MUST complete before 3.4)
5. **Phase 3.3**: Image Processing
6. **Phase 3.4**: Template Matching (requires 3.2 completed)
7. **Phase 3.5**: Grid Mapping
8. **Phase 4**: Manual Corrections
9. **Phase 5**: Constraint Solver
10. **Phase 6**: UI & Interaction
11. **Phase 7**: Testing

### PHASE 1.0.0: FOUNDATION & HTML STRUCTURE [Est: 4 hours]

#### 1.1.0 Create Base HTML File
- [x] 1.1.1 Create puzzle-solver.html with DOCTYPE and meta tags
- [x] 1.1.2 Add viewport meta for responsive design
- [x] 1.1.3 Set title "GAP Puzzle Solver" and favicon placeholder
- [x] 1.1.4 Add OpenCV.js CDN script tag with onload handler
- [x] 1.1.5 Create loading indicator for OpenCV initialization

#### 1.2.0 HTML Layout Structure
- [x] 1.2.1 Create main container div with id="app"
- [x] 1.2.2 Add header section with title and instructions
- [x] 1.2.3 Create left panel div for image preview (id="image-panel")
- [x] 1.2.4 Create right panel div for interactive grid (id="grid-panel")
- [x] 1.2.5 Add control panel section with buttons container
- [x] 1.2.6 Create status bar footer with progress indicator
- [x] 1.2.7 Add hidden file input element for uploads
- [x] 1.2.8 Create modal overlay for error messages

#### 1.3.0 CSS Framework Setup
- [x] 1.3.1 Define CSS variables for colors and spacing
- [x] 1.3.2 Create grid layout system using CSS Grid
- [x] 1.3.3 Style responsive breakpoints (mobile, tablet, desktop)
- [x] 1.3.4 Add base typography and font imports
- [x] 1.3.5 Create utility classes for common patterns
- [x] 1.3.6 Style loading states and animations
- [x] 1.3.7 Define hover and focus states for accessibility
- [x] 1.3.8 Add print styles for solution output

#### 1.4.0 JavaScript Foundation
- [x] 1.4.1 Create main app object namespace
- [x] 1.4.2 Set up event listener for DOMContentLoaded
- [x] 1.4.3 Initialize global variables and constants
- [x] 1.4.4 Create error handling wrapper functions
- [x] 1.4.5 Set up console logging system with levels
- [x] 1.4.6 Add performance monitoring utilities
- [x] 1.4.7 Create memory management helpers

#### 1.5.0 Template Folder Setup
- [x] 1.5.1 Create /templates/ directory structure
- [x] 1.5.2 Add template requirements documentation
- [x] 1.5.3 Create placeholder for template validation
- [x] 1.5.4 Set up template path constants
- [x] 1.5.5 Add template loading error handlers

### PHASE 2.0.0: IMAGE INPUT SYSTEM [Est: 6 hours]

#### 2.1.0 File Upload Handler
- [x] 2.1.1 Create FileUploadHandler class
- [x] 2.1.2 Add click event listener to file input
- [x] 2.1.3 Implement file type validation (PNG, JPEG, WebP)
- [x] 2.1.4 Add file size validation (max 25MB)
- [x] 2.1.5 Create file reading with FileReader API
- [x] 2.1.6 Convert file to base64 for processing
- [x] 2.1.7 Display preview in image panel
- [x] 2.1.8 Add upload progress indicator
- [x] 2.1.9 Implement error handling for invalid files
- [x] 2.1.10 Create success feedback animation

#### 2.2.0 Drag & Drop System
- [x] 2.2.1 Create DragDropHandler class
- [x] 2.2.2 Add dragenter event listener to drop zone
- [x] 2.2.3 Add dragover event with preventDefault
- [x] 2.2.4 Add dragleave event for visual feedback
- [x] 2.2.5 Implement drop event handler
- [x] 2.2.6 Add visual drop zone highlighting
- [x] 2.2.7 Create animated drop feedback
- [x] 2.2.8 Handle multiple file drops (reject)
- [x] 2.2.9 Add drag-in-progress overlay
- [x] 2.2.10 Implement mobile touch alternatives

#### 2.3.0 Clipboard Paste Handler
- [x] 2.3.1 Create ClipboardHandler class
- [x] 2.3.2 Add paste event listener to document
- [x] 2.3.3 Extract image from clipboard data
- [x] 2.3.4 Validate clipboard content type
- [x] 2.3.5 Convert clipboard image to blob
- [x] 2.3.6 Handle different clipboard formats
- [x] 2.3.7 Add paste success notification
- [x] 2.3.8 Implement fallback for older browsers
- [x] 2.3.9 Add keyboard shortcut display (Ctrl+V)

#### 2.4.0 Large File Optimization
- [x] 2.4.1 Create LargeImageHandler class
- [x] 2.4.2 Implement file size detection logic
- [x] 2.4.3 Add canvas-based image resizing
- [x] 2.4.4 Create quality reduction algorithm
- [x] 2.4.5 Implement progressive JPEG compression
- [x] 2.4.6 Add dimension calculation for different file sizes
- [x] 2.4.7 Create progress reporting for optimization
- [x] 2.4.8 Implement chunked processing for >20MB
- [x] 2.4.9 Add memory cleanup after optimization
- [x] 2.4.10 Create mobile-specific size limits
- [x] 2.4.11 Add optimization cancelled handler

### PHASE 3.0.0: OPENCV INTEGRATION & IMAGE PROCESSING [Est: 8 hours]

#### 3.1.0 OpenCV.js Initialization
- [x] 3.1.1 Create OpenCVManager class
- [x] 3.1.2 Implement OpenCV ready detection
- [x] 3.1.3 Add timeout handling for load failure
- [x] 3.1.4 Create retry mechanism for CDN issues
- [x] 3.1.5 Display loading progress to user
- [x] 3.1.6 Cache OpenCV status in sessionStorage
- [x] 3.1.7 Add fallback CDN URLs
- [x] 3.1.8 Implement version checking

#### 3.2.0 Template Loading System (PRIORITY - MUST BE COMPLETED BEFORE 3.4)
- [x] 3.2.1 Create TemplateLoader class
- [x] 3.2.2 Define template paths (/templates/*.png)
- [x] 3.2.3 Load user-provided 64x64 PNG templates
- [x] 3.2.4 Convert templates to grayscale cv.Mat
- [x] 3.2.5 Generate scaled versions (0.8x, 1.0x, 1.2x)
- [x] 3.2.6 Cache all template Mats in memory
- [x] 3.2.7 Validate template dimensions (64x64)
- [x] 3.2.8 Handle missing template files gracefully
- [x] 3.2.9 Clean up template Mats on completion

#### 3.3.0 Core Image Processing (PRD-Aligned)
- [x] 3.3.1 Create ImageProcessor class
- [x] 3.3.2 Implement canvas to cv.Mat conversion (cv.imread)
- [x] 3.3.3 Add grayscale conversion (cv.cvtColor RGBA2GRAY)
- [x] 3.3.4 Create aspect ratio preservation logic
- [x] 3.3.5 Implement resize to 1000px max (cv.resize)
- [x] 3.3.6 Implement Mat cleanup utility (.delete() calls)

#### 3.4.0 Template Matching (Shape Detection Only - REQUIRES 3.2 COMPLETED)
- [x] 3.4.1 Create ShapeDetector class
- [x] 3.4.2 Implement cv.matchTemplate with TM_CCOEFF_NORMED
- [x] 3.4.3 Process match results with cv.minMaxLoc
- [x] 3.4.4 Apply 65% confidence threshold filtering
- [x] 3.4.5 Remove overlapping detections (25px radius)
- [x] 3.4.6 Return list of detected shapes with positions
- [x] 3.4.7 Add confidence scores to detections
- [x] 3.4.8 Handle no detections gracefully

#### 3.5.0 Grid Mapping from Shape Positions
- [x] 3.5.1 Create GridMapper class
- [x] 3.5.2 Calculate cell spacing from shape positions
- [x] 3.5.3 Find grid origin point
- [x] 3.5.4 Implement snap-to-grid algorithm
- [x] 3.5.5 Map shapes to grid cells (0-indexed)
- [x] 3.5.6 Generate 2D array representation
- [x] 3.5.7 Validate grid consistency
- [x] 3.5.8 Return grid with shape numbers (1-5, 0 for empty)

### PHASE 4.0.0: MANUAL CORRECTION & VALIDATION [Est: 4 hours]

#### 4.1.0 Manual Grid Editor
- [ ] 4.1.1 Create GridEditor class
- [ ] 4.1.2 Add cell click event handlers
- [ ] 4.1.3 Implement shape cycling on click
- [ ] 4.1.4 Add right-click to clear cell
- [ ] 4.1.5 Create keyboard input (1-5 for shapes, 0 to clear)
- [ ] 4.1.6 Implement undo/redo functionality
- [ ] 4.1.7 Add visual feedback for edits
- [ ] 4.1.8 Save edit history

#### 4.2.0 Detection Validation
- [ ] 4.2.1 Create ValidationDisplay class
- [ ] 4.2.2 Show confidence scores per detection
- [ ] 4.2.3 Highlight low-confidence cells
- [ ] 4.2.4 Display detection statistics
- [ ] 4.2.5 Allow manual approval/rejection
- [ ] 4.2.6 Create fallback to full manual input

### PHASE 5.0.0: CONSTRAINT SOLVER ALGORITHM [Est: 8 hours]

#### 5.1.0 Solver Foundation
- [ ] 5.1.1 Create PuzzleSolver class
- [ ] 5.1.2 Define constraint rules structure
- [ ] 5.1.3 Implement grid state representation
- [ ] 5.1.4 Create domain tracking for cells
- [ ] 5.1.5 Add constraint graph construction
- [ ] 5.1.6 Implement solver configuration options
- [ ] 5.1.7 Create performance timer system

#### 5.2.0 Constraint Validation System
- [ ] 5.2.1 Create ConstraintValidator class
- [ ] 5.2.2 Implement row uniqueness check
- [ ] 5.2.3 Add column uniqueness check
- [ ] 5.2.4 Create constraint violation detection
- [ ] 5.2.5 Implement partial validation for efficiency
- [ ] 5.2.6 Add constraint propagation rules
- [ ] 5.2.7 Create conflict identification
- [ ] 5.2.8 Implement validation caching

#### 5.3.0 Backtracking Search Implementation
- [ ] 5.3.1 Create BacktrackingSearch class
- [ ] 5.3.2 Implement recursive search function
- [ ] 5.3.3 Add Most Constrained Variable heuristic
- [ ] 5.3.4 Create Least Constraining Value ordering
- [ ] 5.3.5 Implement forward checking
- [ ] 5.3.6 Add arc consistency (AC-3)
- [ ] 5.3.7 Create search tree pruning
- [ ] 5.3.8 Implement solution path tracking
- [ ] 5.3.9 Add timeout mechanism (5 seconds)
- [ ] 5.3.10 Create partial solution return

#### 5.4.0 Solution Optimization
- [ ] 5.4.1 Implement solution validation
- [ ] 5.4.2 Add solution uniqueness check
- [ ] 5.4.3 Create solution quality scoring
- [ ] 5.4.4 Implement alternative solution search
- [ ] 5.4.5 Add solution explanation generator
- [ ] 5.4.6 Create solution comparison utility

### PHASE 6.0.0: USER INTERFACE & INTERACTION [Est: 6 hours]

#### 6.1.0 Interactive Grid Display
- [ ] 6.1.1 Create GridDisplay class
- [ ] 6.1.2 Generate grid HTML dynamically
- [ ] 6.1.3 Add shape rendering with Unicode/SVG
- [ ] 6.1.4 Implement cell styling system
- [ ] 6.1.5 Create hover effects
- [ ] 6.1.6 Add focus indicators
- [ ] 6.1.7 Implement cell selection
- [ ] 6.1.8 Create grid size toggle (4x4/5x5)
- [ ] 6.1.9 Add grid lines and borders
- [ ] 6.1.10 Implement responsive sizing

#### 6.2.0 Control Panel Implementation
- [ ] 6.2.1 Create ControlPanel class
- [ ] 6.2.2 Implement Solve button handler
- [ ] 6.2.3 Add Clear button functionality
- [ ] 6.2.4 Create Reset button handler
- [ ] 6.2.5 Implement Export solution button
- [ ] 6.2.6 Add Settings toggle
- [ ] 6.2.7 Create Help modal trigger
- [ ] 6.2.8 Implement keyboard shortcuts display

#### 6.3.0 Solution Display System
- [ ] 6.3.1 Create SolutionDisplay class
- [ ] 6.3.2 Implement solved cell highlighting
- [ ] 6.3.3 Add solution animation
- [ ] 6.3.4 Create before/after comparison
- [ ] 6.3.5 Implement solution stats display
- [ ] 6.3.6 Add copy solution to clipboard
- [ ] 6.3.7 Create printable solution view

#### 6.4.0 Error Handling & Feedback
- [ ] 6.4.1 Create ErrorHandler class
- [ ] 6.4.2 Implement user-friendly error messages
- [ ] 6.4.3 Add error recovery suggestions
- [ ] 6.4.4 Create toast notification system
- [ ] 6.4.5 Implement progress indicators
- [ ] 6.4.6 Add success animations
- [ ] 6.4.7 Create help tooltips
- [ ] 6.4.8 Implement accessibility announcements

### PHASE 7.0.0: TESTING & OPTIMIZATION [Est: 4 hours]

#### 7.1.0 Unit Testing Setup
- [ ] 7.1.1 Create test data fixtures
- [ ] 7.1.2 Implement shape detection tests
- [ ] 7.1.3 Add solver algorithm tests
- [ ] 7.1.4 Create grid mapping tests
- [ ] 7.1.5 Implement file handling tests

#### 7.2.0 Performance Optimization
- [ ] 7.2.1 Profile OpenCV operations
- [ ] 7.2.2 Optimize template matching
- [ ] 7.2.3 Implement lazy loading
- [ ] 7.2.4 Add request animation frame
- [ ] 7.2.5 Optimize memory usage
- [ ] 7.2.6 Create performance benchmarks

#### 7.3.0 Cross-browser Testing
- [ ] 7.3.1 Test Chrome 90+
- [ ] 7.3.2 Test Firefox 88+
- [ ] 7.3.3 Test Safari 14+
- [ ] 7.3.4 Test Edge 90+
- [ ] 7.3.5 Test mobile browsers
- [ ] 7.3.6 Document compatibility issues

## Critical Implementation Notes

### ⚠️ Prerequisites Before Starting
1. **Create /templates/ folder** in project root
2. **Add 5 template PNG files** (64x64px each):
   - circle.png
   - star.png
   - cross.png
   - triangle.png
   - square.png
3. **Test OpenCV.js CDN** is accessible
4. **Verify browser compatibility** (WebAssembly support required)

### 🚫 What NOT to Implement (Per PRD)
- NO edge detection for grid finding
- NO Hough line detection
- NO perspective correction
- NO advanced image filters (noise reduction, histogram equalization)
- NO rotation correction
- NO brightness/contrast adjustment
- NO grid line detection
- NO complex image preprocessing beyond grayscale

### ✅ What to Implement (Per PRD)
- Template-based shape detection only
- User selects grid size (4×4 or 5×5) upfront
- Simple grayscale conversion
- Aspect-preserving resize to 1000px max
- Snap-to-grid position correction
- Shape positions determine grid structure
- Manual correction interface for misdetections

## Progress Tracking

**Total Steps**: 167
**Completed**: 89
**In Progress**: 0
**Remaining**: 78

**Estimated Total Time**: ~40 hours

## Development Tips

1. **Start with Phase 1** - Get the HTML structure right first
2. **Test after each phase** - Don't wait until the end
3. **Templates are critical** - Ensure they're loaded before matching
4. **Memory management** - Always delete OpenCV Mats
5. **Keep it simple** - Follow PRD, avoid feature creep
6. **User feedback** - Show progress for long operations
7. **Graceful degradation** - Always have manual fallback

## Version History

### Version 1.2 (Current)
- Completed Phase 3: OpenCV Integration & Image Processing (38 items)
  - OpenCV.js initialization with fallback CDNs
  - Template loading system with multi-scale support
  - Image processing pipeline (grayscale, resize)
  - Template matching with 65% confidence threshold
  - Grid mapping with snap-to-grid algorithm
- Added "Detect Shapes" button to trigger shape detection
- Integrated complete detection pipeline from image to grid

### Version 1.1
- Completed Phase 1: Foundation & HTML Structure (28 items)
- Completed Phase 2: Image Input System (23 items)
- Fixed cross symbol to display as "+" instead of "✕"
- Added progress tracking rule for better documentation

### Version 1.0
- Fresh start implementation plan
- Aligned with PRD requirements
- Corrected execution order
- Removed advanced features not in MVP
- Template-based detection focus