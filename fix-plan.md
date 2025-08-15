# Fix Plan for Shape Detection Issues

## Priority 1: Memory Management Error (Critical)

### Issue: "Mat instance already deleted" 
**Location:** Lines around ShapeDetector and GridMapper cleanup

**Fix Steps:**
1. Review all `.delete()` calls in the detection pipeline
2. Ensure Mats are not deleted until completely done with them
3. Check for duplicate deletion calls
4. Add try-catch blocks around Mat operations
5. Implement proper cleanup sequence

**Code areas to check:**
- `ShapeDetector.detectShapes()` - template cleanup
- `GridMapper.mapToGrid()` - any Mat operations
- `ImageProcessor.processImage()` - grayscale and resized Mat cleanup

## Priority 2: Grid Mapping Algorithm (High)

### Issue: Shapes not placed in correct grid positions

**Root Cause Analysis:**
- Grid origin calculation may be incorrect
- Cell spacing calculation not accounting for image padding
- Snap-to-grid algorithm not working properly

**Fix Steps:**
1. Add debug logging for:
   - Detected shape coordinates (x, y)
   - Calculated grid parameters (origin, cell width/height)
   - Final mapped grid positions
2. Implement better grid detection:
   - Find bounding box of all detections
   - Calculate uniform grid spacing
   - Account for image borders/padding
3. Improve snap-to-grid:
   - Use clustering to group nearby detections
   - Calculate grid from cluster centers
   - Handle edge cases (partial grids)

**Proposed Algorithm:**
```javascript
// 1. Find bounding box of all detections
const minX = Math.min(...detections.map(d => d.x));
const maxX = Math.max(...detections.map(d => d.x));
const minY = Math.min(...detections.map(d => d.y));
const maxY = Math.max(...detections.map(d => d.y));

// 2. Calculate cell size
const cellWidth = (maxX - minX) / (gridSize - 1);
const cellHeight = (maxY - minY) / (gridSize - 1);

// 3. Map each detection to nearest grid cell
detections.forEach(detection => {
    const col = Math.round((detection.x - minX) / cellWidth);
    const row = Math.round((detection.y - minY) / cellHeight);
    // Validate and place in grid
});
```

## Priority 3: Detection Threshold Tuning (Medium)

### Issue: Inconsistent shape detection (especially squares)

**Current Settings:**
- Confidence threshold: 65%
- Template scales: 0.8x, 1.0x, 1.2x
- Method: TM_CCOEFF_NORMED

**Fix Steps:**
1. Lower confidence threshold to 50% initially
2. Add more template scales: [0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4]
3. Test different matching methods:
   - TM_CCORR_NORMED
   - TM_SQDIFF_NORMED (with inverted threshold)
4. Implement adaptive thresholding per shape type
5. Add non-maximum suppression with larger radius (35px instead of 25px)

## Priority 4: Template Quality (Medium)

### Issue: Templates may not match actual puzzle shapes well

**Fix Steps:**
1. Generate new templates from actual puzzle images
2. Create multiple template variations per shape:
   - Different colors converted to grayscale
   - Different sizes
   - Different styles (filled vs outlined)
3. Use template averaging for better matching
4. Consider edge detection on templates for better matching

## Priority 5: Enhanced Debugging (Low)

### Add comprehensive logging:

1. **Detection phase:**
   - Log each template match with confidence score
   - Log filtered vs unfiltered detections
   - Save detection overlay image for debugging

2. **Grid mapping phase:**
   - Log grid parameters calculation
   - Log each shape's mapping decision
   - Create visual grid overlay

3. **Add debug mode:**
   - Draw detection rectangles on image
   - Show confidence scores
   - Display grid lines
   - Color-code detection quality

## Implementation Order

1. **Immediate fixes (Phase 1):**
   - Fix Mat deletion error
   - Add basic debug logging
   - Lower detection threshold to 50%

2. **Grid mapping improvements (Phase 2):**
   - Implement new grid calculation algorithm
   - Add clustering for grid detection
   - Test with all sample images

3. **Detection improvements (Phase 3):**
   - Add more template scales
   - Test different matching methods
   - Implement per-shape thresholds

4. **Long-term improvements (Phase 4):**
   - Generate better templates
   - Add visual debugging tools
   - Implement machine learning approach (future)

## Testing Strategy

1. Create unit tests for:
   - Grid mapping with known coordinates
   - Template matching with synthetic images
   - Memory management with Mat lifecycle

2. Integration tests with:
   - All 8 test images in /test/4/
   - Various image sizes and qualities
   - Different grid configurations

3. Performance tests:
   - Memory usage monitoring
   - Detection speed benchmarks
   - Accuracy metrics tracking

## Success Metrics

- **Memory:** No Mat deletion errors
- **Detection rate:** >90% of visible shapes detected
- **Position accuracy:** >95% correct grid placement
- **Processing time:** <2 seconds per image
- **All test images:** 100% successful processing