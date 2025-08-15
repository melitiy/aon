# Product Requirements Document
## Visual Pattern Puzzle Solver Application

### 1. Executive Summary

**Product Name:** GAP Challenge Visual Pattern Solver  
**Version:** 1.0  
**Date:** August 2025  
**Implementation:** Claude Code with OpenCV.js  
**Document Status:** Final  

This PRD defines a web-based application that automatically detects and solves GAP Challenge visual pattern puzzles using OpenCV.js for professional image processing and template matching. The application will be built as a single HTML file with inline CSS/JavaScript, requiring only OpenCV.js as an external dependency loaded from CDN.

### 2. Problem Statement

#### Current Challenge
Users solving GAP Challenge puzzles face several pain points:
- Manual pattern identification from screenshots is tedious and error-prone
- Solving larger grids (5×5) requires significant mental effort and time
- No quick way to verify solutions or identify mistakes
- Limited practice resources with instant feedback
- Difficulty tracking which constraints are violated when stuck

#### Target Users
- Cognitive assessment test participants
- Puzzle enthusiasts practicing pattern recognition
- Educational institutions using visual puzzles for training
- Researchers studying problem-solving strategies

### 3. Solution Overview

A single-page web application that:
1. Accepts puzzle screenshots via paste, drag-drop, or file upload
2. Automatically detects the grid and identifies shapes using pattern matching
3. Populates an interactive grid with detected shapes
4. Allows manual correction of detection errors
5. Solves incomplete puzzles using constraint satisfaction algorithms
6. Provides visual feedback showing the solution

### 4. Detailed Functional Requirements

#### 4.1 Image Input & Processing

**Input Methods:**
- **Clipboard Paste**: Ctrl/Cmd+V to paste screenshots directly
- **Drag & Drop**: Drag image files onto designated drop zone
- **File Upload**: Click to browse and select image files
- **Supported Formats**: PNG, JPEG, JPG, WebP
- **Max File Size**: 25MB (with automatic optimization for files >5MB)

**Image Processing Pipeline:**
1. Display uploaded image in preview pane (left side)
2. **Large Image Handling (>5MB):**
   - Automatic pre-processing resize if dimensions exceed 2000px
   - Progressive quality reduction for files >10MB
   - Client-side compression using Canvas API
   - Memory-efficient processing with cleanup after each step
3. Detect grid boundaries automatically
4. Identify grid size (4×4 or 5×5)
5. Segment grid into individual cells
6. Classify shape in each cell (or mark as empty)
7. Handle various resolutions (min 200px, max 4000px per dimension)

**Large Image Optimization Strategy:**
```javascript
function optimizeLargeImage(file, callback) {
    const MAX_DIMENSION = 2000;
    const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
    
    if (file.size <= MAX_FILE_SIZE) {
        return callback(file); // No optimization needed
    }
    
    const img = new Image();
    img.onload = () => {
        // Calculate optimal dimensions
        const scale = Math.min(
            MAX_DIMENSION / img.width,
            MAX_DIMENSION / img.height,
            1
        );
        
        // Create canvas for resizing
        const canvas = document.createElement('canvas');
        canvas.width = img.width * scale;
        canvas.height = img.height * scale;
        
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        
        // Convert to blob with compression
        canvas.toBlob((blob) => {
            callback(blob);
            // Clean up
            canvas.remove();
        }, 'image/jpeg', 0.9); // 90% quality
    };
    
    img.src = URL.createObjectURL(file);
}
```

#### 4.2 Shape Recognition System

**Supported Shapes:**
- Circle (represented as 1)
- Star (represented as 2)
- Cross/X (represented as 3)
- Triangle (represented as 4)
- Square (represented as 5)
- Empty cell (represented as 0)

**Recognition Features:**
- Template matching with multiple scales
- Rotation invariance for symmetric shapes
- Color-agnostic detection (works with any color scheme)
- Confidence scoring for each detection
- Fallback to manual input on low confidence

#### 4.3 Interactive Grid Interface

**Grid Display:**
- Dynamic sizing based on detected/selected dimensions
- Visual representation using Unicode symbols or SVG icons
- Clear distinction between pre-filled and solved cells
- Hover effects for better interactivity

**Manual Input Controls:**
- Click any cell to cycle through shapes
- Right-click to clear a cell
- Keyboard shortcuts (1-5 for shapes, 0 or Delete for clear)
- Undo/Redo functionality (Ctrl+Z/Ctrl+Y)

**Grid States:**
- Initial: Shows detected shapes from image
- Editing: User corrections to detection
- Solving: Processing animation
- Solved: Complete solution with highlighted changes
- Error: Invalid configurations highlighted in red

#### 4.4 Constraint Satisfaction Solver

**Algorithm Requirements:**
- Implement backtracking with constraint propagation
- Arc consistency checking (AC-3 algorithm)
- Most Constrained Variable (MCV) heuristic
- Forward checking to prune search space

**Solving Process:**
1. Validate initial configuration
2. Identify empty cells
3. Calculate possible values for each cell
4. Apply constraint propagation
5. Use backtracking for ambiguous cases
6. Return first valid solution

**Performance Targets:**
- 4×4 grid: < 100ms
- 5×5 grid: < 500ms
- Timeout after 5 seconds with partial solution

#### 4.5 User Interface Layout

**Layout Structure:**
```
+--------------------------------------------------+
|                    Header                        |
+----------------------+---------------------------+
|                      |                           |
|   Image Preview      |    Interactive Grid       |
|     (Original)       |      (Detected/Solved)    |
|                      |                           |
|                      +---------------------------+
|                      |     Control Panel         |
|                      | [Solve] [Clear] [Reset]   |
+----------------------+---------------------------+
|              Status Bar / Instructions           |
|         Progress Bar (for large files)           |
+--------------------------------------------------+
```

**Progressive Loading UI:**
- Show file size indicator when image selected
- Display "Optimizing..." message for files > 5MB
- Progress bar showing optimization steps:
  1. Reading file (20%)
  2. Resizing image (40%)
  3. Compressing (60%)
  4. Processing (80%)
  5. Detecting shapes (100%)
- Estimated time remaining for large files
- Cancel button during processing

**Responsive Design:**
- Desktop: Side-by-side layout (min width: 1024px)
- Tablet: Stacked layout with tabs (768px - 1023px)
- Mobile: Single column with accordion sections (< 768px)

### 5. Technical Implementation (Claude Code Specific)

#### 5.1 Architecture

**Single HTML File Structure:**
- Inline CSS for styling
- Inline JavaScript for functionality
- OpenCV.js via CDN for image processing
- No other external dependencies

**Required Libraries (CDN):**
- OpenCV.js (https://docs.opencv.org/4.5.0/opencv.js) - Core image processing and template matching
- All other functionality in pure JavaScript

#### 5.2 Image Processing Pipeline (OpenCV-Based)

**Complete Processing Flow:**
```
1. Load & Initialize OpenCV.js
   ↓
2. Convert to Grayscale (cv.cvtColor)
   ↓
3. Aspect-Preserving Resize (cv.resize)
   ↓
4. Dynamic Template Generation
   ↓
5. Template Matching (cv.matchTemplate)
   ↓
6. Grid Mapping & Snap to Grid (Pure JS)
   ↓
Output: 2D Grid Array
```

**OpenCV Implementation Strategy:**
```javascript
class OpenCVPuzzleDetector {
  - initialize() // Load OpenCV and prepare templates
  - processScreenshot(canvas) // Main processing pipeline
  - resizePreservingAspect(mat, maxSize) // OpenCV resize
  - performTemplateMatching(image, template) // OpenCV matching
  - mapToGrid(detections) // Pure JS grid mapping
  - snapToGrid(detection, gridParams) // Position correction
}
```

**Shape Detection Method:**
1. Generate templates at multiple scales (0.8×, 1.0×, 1.2× of estimated size)
2. Use OpenCV's cv.matchTemplate with TM_CCOEFF_NORMED
3. Apply confidence threshold (65%)
4. Filter overlapping detections
5. Map to grid coordinates with snap-to-grid correction

#### 5.3 Solver Algorithm

```javascript
// Constraint Satisfaction Problem solver structure
class PuzzleSolver {
  - validateConstraints(grid)
  - getEmptyCells(grid)
  - getPossibleValues(cell, grid)
  - propagateConstraints(grid)
  - backtrackSearch(grid)
  - isComplete(grid)
}
```

#### 5.4 Large Image Processing Architecture

**Chunked Processing for Very Large Images (>20MB):**
```javascript
class ChunkedImageProcessor {
    async processInChunks(largeImage) {
        // Split processing into manageable chunks
        const chunks = [];
        
        // Step 1: Resize in chunks to prevent memory overflow
        const tempCanvas = document.createElement('canvas');
        const ctx = tempCanvas.getContext('2d');
        
        // Process in segments
        for (let y = 0; y < largeImage.height; y += 500) {
            for (let x = 0; x < largeImage.width; x += 500) {
                const chunk = this.extractChunk(largeImage, x, y, 500, 500);
                chunks.push(chunk);
                
                // Allow browser to breathe
                await new Promise(resolve => setTimeout(resolve, 10));
            }
        }
        
        // Reassemble and process
        return this.reassembleAndProcess(chunks);
    }
}
```

**Memory-Efficient Processing Pipeline:**
1. Check available memory before processing
2. Use OffscreenCanvas when available (Chrome 69+)
3. Process in Web Worker for images > 15MB (if supported)
4. Fallback to chunked processing for very large files
5. Aggressive garbage collection between steps

### 6. User Experience Flow

#### 6.1 Happy Path

1. User takes screenshot of puzzle
2. Pastes image using Ctrl+V
3. System detects grid and shapes automatically
4. User reviews detection (makes corrections if needed)
5. Clicks "Solve" button
6. Solution appears with solved cells highlighted
7. User can copy or screenshot the solution

#### 6.2 Error Handling

**Detection Failures:**
- Message: "Unable to detect grid. Please ensure image is clear and well-lit."
- Action: Enable manual grid setup mode

**Invalid Puzzle Configuration:**
- Message: "This puzzle violates constraints. Check row X / column Y."
- Action: Highlight conflicting cells in red

**No Solution Found:**
- Message: "No valid solution exists for this configuration."
- Action: Suggest checking initial input

**Large File Handling:**
- Message: "Optimizing large image... This may take a few seconds."
- Action: Show progress bar during optimization
- For files > 25MB: "File too large. Please use an image under 25MB."
- For mobile > 5MB: "File too large for mobile. Please use an image under 5MB."

**Memory Issues:**
- Message: "Low memory detected. Optimizing processing..."
- Action: Automatic quality reduction and cleanup

### 7. Performance Optimization

**Large Image Handling (Files >5MB):**
- Pre-process images client-side before OpenCV processing
- Resize images exceeding 2000px in any dimension
- Use JPEG compression (90% quality) for oversized files
- Implement progressive loading with user feedback

**Memory Management for Large Files:**
```javascript
class LargeImageProcessor {
    async processLargeImage(file) {
        // Step 1: Check file size
        if (file.size > 25 * 1024 * 1024) {
            throw new Error('File too large. Maximum 25MB.');
        }
        
        // Step 2: Pre-optimize if needed
        let processedFile = file;
        if (file.size > 5 * 1024 * 1024) {
            processedFile = await this.optimizeImage(file);
        }
        
        // Step 3: Process with OpenCV
        const result = await this.processWithOpenCV(processedFile);
        
        // Step 4: Aggressive cleanup
        this.cleanup();
        
        return result;
    }
    
    cleanup() {
        // Force garbage collection hints
        if (window.gc) window.gc();
        
        // Clear any cached images
        URL.revokeObjectURL(this.lastObjectURL);
        
        // Clear canvas contexts
        const canvases = document.querySelectorAll('canvas');
        canvases.forEach(canvas => {
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        });
    }
}
```

**OpenCV.js Optimization:**
- One-time library loading (3-5 seconds)
- Reuse pre-generated templates across detections
- Proper memory management (delete OpenCV Mats immediately after use)
- Use INTER_AREA for downscaling, INTER_LINEAR for upscaling
- Process images in chunks if memory usage exceeds 200MB

**Image Processing:**
- Resize images to max 1000px before OpenCV processing
- For images >10MB: resize to max 1500px
- For images >20MB: resize to max 1000px and reduce quality
- Process in grayscale to reduce computational load
- Generate only 3 template scales per shape

**Template Matching:**
- Use TM_CCOEFF_NORMED for best accuracy
- Apply 65% confidence threshold
- Filter duplicate detections within 25px radius

**Grid Mapping:**
- Pure JavaScript for fast execution
- Snap-to-grid correction for position errors
- Single pass through detections

### 8. Accessibility Requirements

- **Keyboard Navigation**: Full functionality via keyboard
- **Screen Reader Support**: ARIA labels for all interactive elements
- **Color Blind Mode**: Shape labels (1-5) in addition to symbols
- **High Contrast Mode**: Toggle for better visibility
- **Focus Indicators**: Clear visual focus states

### 9. Browser Compatibility

**Minimum Requirements:**
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

**Required APIs:**
- Canvas API
- Clipboard API
- File API
- Drag and Drop API
- WebAssembly (for OpenCV.js)

**External Dependencies:**
- OpenCV.js v4.5.0+ (loaded from CDN)
- ~8MB download (cached after first load)

### 10. Testing Strategy

#### 10.1 Test Cases

**Image Processing:**
- Various image resolutions (200px to 4000px)
- Different color schemes
- Rotated/skewed images
- Partial grids
- Low quality/blurry images

**Large File Testing:**
- 5MB image (should process with minimal optimization)
- 10MB image (should auto-resize to 2000px)
- 15MB image (should auto-resize to 1500px + compression)
- 20MB image (should auto-resize to 1000px + aggressive compression)
- 25MB image (should process at limit)
- 30MB image (should reject gracefully)
- Mobile device with 6MB file (should reject on mobile)

**Memory Testing:**
- Process multiple large files sequentially
- Monitor memory usage during processing
- Verify cleanup after processing
- Test browser tab recovery after high memory usage

**Solver Validation:**
- Empty grid
- Single empty cell
- Multiple solutions
- No solution scenarios
- All grid sizes

**UI Testing:**
- All input methods
- Browser compatibility
- Responsive layouts
- Accessibility features

#### 10.2 Test Data

Create test suite with:
- 10 sample puzzles per grid size
- Known solutions for validation
- Edge cases (all same shape, alternating patterns)
- Performance benchmarks

### 11. MVP Scope (Phase 1)

**Included:**
- OpenCV.js integration for image processing
- Grayscale conversion and aspect-preserving resize
- Template matching for 5 shapes (circle, star, cross, triangle, square)
- Snap-to-grid position correction
- Support for 4×4 and 5×5 grids
- Multiple input methods (paste, drag-drop, file upload)
- Basic solver algorithm
- Simple UI with detection preview

**Excluded (Future Phases):**
- Manual shape correction interface
- Multiple puzzle formats
- Solution explanation
- Puzzle generation
- User accounts/history
- Advanced image enhancement (noise reduction, rotation correction)

### 12. Success Metrics

**Technical Metrics:**
- Detection accuracy: > 90% on clear images
- Solve success rate: 100% for valid puzzles
- Response time: < 1 second for 5×5 grids
- Browser crash rate: < 0.1%

**User Metrics:**
- Task completion rate: > 95%
- Average time to solution: < 30 seconds
- Error recovery success: > 80%
- User satisfaction: > 4.5/5

### 13. Implementation Timeline

**Week 1: OpenCV Integration & Core Structure**
- Set up HTML layout with inline CSS
- Integrate OpenCV.js from CDN
- Implement image input methods (paste, drag-drop, file)
- Create basic UI components

**Week 2: Image Processing Pipeline**
- Implement grayscale conversion
- Add aspect-preserving resize
- Create shape templates in OpenCV
- Test template generation at multiple scales

**Week 3: Detection & Grid Mapping**
- Implement template matching with OpenCV
- Add confidence thresholding
- Create snap-to-grid algorithm
- Build grid parameter calculation

**Week 4: Solver & Polish**
- Implement constraint satisfaction solver
- Add solution display
- Optimize performance
- Cross-browser testing
- Final UI improvements

### 14. Risk Mitigation

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Large files (>10MB) causing browser crash | High | High | Pre-process resize, progressive loading, memory cleanup |
| OpenCV.js load time (8MB) | High | Medium | Cache after first load, show loading progress |
| Memory leaks from OpenCV Mats | Medium | High | Strict cleanup with .delete() calls, memory monitoring |
| Out of memory with 25MB images | Medium | High | Automatic downscaling, quality reduction, chunked processing |
| Template matching accuracy | Medium | High | Multiple scales, 65% threshold, snap-to-grid correction |
| Browser compatibility with WebAssembly | Low | High | Fallback to manual input mode |
| Different image resolutions | High | Medium | Aspect-preserving resize to 1000px max |
| Poor image quality | Medium | Medium | Grayscale conversion, confidence thresholds |
| Mobile device memory limitations | High | High | Detect mobile and enforce stricter size limits (5MB) |

### 15. Appendix

#### A. Complete OpenCV Processing Pipeline

```javascript
class OpenCVPuzzleDetector {
    constructor(gridSize) {
        this.gridSize = gridSize;
        this.templates = {};
        this.isReady = false;
    }
    
    async initialize() {
        // Wait for OpenCV to load
        await this.waitForOpenCV();
        // Generate all template sizes
        this.generateAllTemplates();
        this.isReady = true;
    }
    
    processScreenshot(canvas) {
        let src = cv.imread(canvas);
        
        // Step 1: Grayscale conversion
        let gray = new cv.Mat();
        cv.cvtColor(src, gray, cv.COLOR_RGBA2GRAY);
        
        // Step 2: Aspect-preserving resize
        let resized = this.resizePreservingAspect(gray, 1000);
        
        // Step 3: Template matching
        const detections = this.detectAllShapes(resized);
        
        // Step 4: Grid mapping
        const grid = this.mapToGrid(detections);
        
        // Cleanup
        src.delete();
        gray.delete();
        resized.delete();
        
        return grid;
    }
}
```

#### B. Large Image Handler

```javascript
class LargeImageHandler {
    constructor() {
        this.MAX_FILE_SIZE = 25 * 1024 * 1024; // 25MB
        this.OPTIMIZATION_THRESHOLD = 5 * 1024 * 1024; // 5MB
    }
    
    async handleImageUpload(file) {
        // Validate size
        if (file.size > this.MAX_FILE_SIZE) {
            throw new Error(`File too large. Maximum size is 25MB, got ${(file.size / 1024 / 1024).toFixed(1)}MB`);
        }
        
        // Check if optimization needed
        if (file.size > this.OPTIMIZATION_THRESHOLD) {
            console.log(`Optimizing large image: ${(file.size / 1024 / 1024).toFixed(1)}MB`);
            return await this.optimizeImage(file);
        }
        
        return file;
    }
    
    async optimizeImage(file) {
        return new Promise((resolve) => {
            const img = new Image();
            
            img.onload = () => {
                // Determine target dimensions based on file size
                let maxDimension;
                const fileSizeMB = file.size / (1024 * 1024);
                
                if (fileSizeMB < 10) {
                    maxDimension = 2000;
                } else if (fileSizeMB < 20) {
                    maxDimension = 1500;
                } else {
                    maxDimension = 1000;
                }
                
                // Calculate scale
                const scale = Math.min(
                    maxDimension / img.width,
                    maxDimension / img.height,
                    1
                );
                
                // Create resizing canvas
                const canvas = document.createElement('canvas');
                canvas.width = Math.round(img.width * scale);
                canvas.height = Math.round(img.height * scale);
                
                const ctx = canvas.getContext('2d');
                ctx.imageSmoothingEnabled = true;
                ctx.imageSmoothingQuality = 'high';
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                
                // Convert to blob with compression
                const quality = fileSizeMB > 15 ? 0.8 : 0.9;
                
                canvas.toBlob((blob) => {
                    console.log(`Optimized from ${(file.size / 1024 / 1024).toFixed(1)}MB to ${(blob.size / 1024 / 1024).toFixed(1)}MB`);
                    
                    // Cleanup
                    URL.revokeObjectURL(img.src);
                    canvas.remove();
                    
                    resolve(blob);
                }, 'image/jpeg', quality);
            };
            
            img.onerror = () => {
                resolve(file); // Return original if optimization fails
            };
            
            img.src = URL.createObjectURL(file);
        });
    }
    
    // Mobile device detection and limits
    getMobileSettings() {
        const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
        
        if (isMobile) {
            return {
                maxFileSize: 5 * 1024 * 1024, // 5MB for mobile
                maxDimension: 1000,
                compressionQuality: 0.8
            };
        }
        
        return {
            maxFileSize: this.MAX_FILE_SIZE,
            maxDimension: 2000,
            compressionQuality: 0.9
        };
    }
}
```

#### C. Shape Templates Generation

```javascript
// OpenCV template creation for each shape
createCircleTemplate(size) {
    let template = new cv.Mat(size, size, cv.CV_8UC1, new cv.Scalar(0));
    const center = new cv.Point(size/2, size/2);
    const radius = (size/2) - 3;
    cv.circle(template, center, radius, new cv.Scalar(255), -1);
    return template;
}

createStarTemplate(size) {
    // 5-pointed star with inner/outer radius ratio
    let template = new cv.Mat(size, size, cv.CV_8UC1, new cv.Scalar(0));
    const points = generateStarPoints(size);
    cv.fillPoly(template, points, new cv.Scalar(255));
    return template;
}

// Similar for cross, triangle, square
```

#### D. Grid Mapping Algorithm

```javascript
mapToGrid(detections) {
    // Calculate grid parameters from shape positions
    const gridParams = calculateGridParameters(detections);
    
    // Snap each detection to nearest grid cell
    const grid = Array(this.gridSize).fill()
        .map(() => Array(this.gridSize).fill(0));
    
    detections.forEach(detection => {
        const pos = snapToGrid(detection, gridParams);
        if (isValidPosition(pos)) {
            grid[pos.row][pos.col] = detection.shapeNumber;
        }
    });
    
    return grid;
}

snapToGrid(detection, params) {
    const col = Math.round((detection.x - params.origin.x) / params.cellWidth);
    const row = Math.round((detection.y - params.origin.y) / params.cellHeight);
    return { row, col };
}
```

#### E. Performance Metrics

| Operation | Target Time | Actual (Typical) | Large Files (>10MB) |
|-----------|------------|------------------|---------------------|
| OpenCV.js Load | 5s | 3-5s | Same |
| Image Optimization | - | - | 1-2s |
| Grayscale Conversion | 50ms | 20-30ms | 50-100ms |
| Image Resize | 100ms | 50-80ms | 200-500ms |
| Template Matching (all shapes) | 1s | 600-800ms | 800-1200ms |
| Grid Mapping | 100ms | 20-50ms | Same |
| **Total Processing** | **< 2s** | **1-1.5s** | **2-4s** |

### 16. Conclusion

This PRD provides a comprehensive blueprint for building the GAP Challenge Visual Pattern Solver using Claude Code. The focus on browser-based implementation without external dependencies ensures the application can be developed and deployed quickly while maintaining functionality and user experience. The modular approach allows for iterative development and easy troubleshooting within Claude Code's constraints.