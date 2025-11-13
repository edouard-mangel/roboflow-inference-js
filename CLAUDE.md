# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a browser-based demo application for running Roboflow computer vision models. It's a vanilla JavaScript project that demonstrates object detection using webcam, uploaded images, or predefined sample images. The demo was originally used on the Roboflow homepage and has been generalized for template use.

## Development Commands

### Run Development Server
```bash
npm run dev
```
Starts Vite dev server for local development with hot reload.

### Install Dependencies
```bash
npm install
```

## Architecture

### Core JavaScript Files

The project has three main JavaScript implementations (variations of the same functionality):

- **script.js**: Original implementation using `inferencejs.InferenceEngine` for model inference
- **newhome.js**: Alternative implementation using the older `roboflow` client library
- **new.js**: Latest implementation combining features from both approaches

**Important**: `index.html` currently loads `newhome.js` (line 84). When modifying inference logic, check which JS file is actually being used by the HTML.

### Inference Approaches

The codebase supports two different Roboflow inference methods:

1. **InferenceEngine API** (script.js, new.js):
   - Uses `inferencejs.InferenceEngine()`
   - Worker-based model loading: `inferEngine.startWorker(modelName, version, apiKey)`
   - Inference: `inferEngine.infer(workerId, new inferencejs.CVImage(element))`
   - Requires managing worker IDs

2. **Legacy Roboflow API** (newhome.js):
   - Uses `roboflow.auth().load()` pattern
   - Direct model methods: `model.detect(video)`
   - Simpler API but older approach

3. **Detect API** (all files):
   - Direct HTTP POST to `https://detect.roboflow.com/{model}/{version}?api_key={key}`
   - Used for static image inference
   - Returns predictions in different format than InferenceEngine

### Key Architecture Patterns

**Canvas Management**: The application uses multiple canvas elements:
- `picture_canvas`: For static image inference display
- `video_canvas`: For real-time webcam inference overlay
- Both canvases are 640x480 by default (responsive on mobile)

**Model Configuration**: Models are defined in `available_models` object with:
- `name`: Display name
- `version`: Model version number
- `confidence`: Detection threshold (0.3-0.6)
- `imageGrid`: Array of 4 sample image URLs
- `video`: Demo video URL (optional)

**Coordinate System**: The application handles aspect ratio differences between source images/video and the fixed canvas size:
- `getCoordinates(img)`: Calculates crop/scale coordinates to fit media into canvas
- Returns `[sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight, scalingRatio]`
- Bounding boxes are adjusted using these coordinates

**Prediction Format Mapping**: Different APIs return different prediction formats:
- **InferenceEngine**: Returns predictions with `bbox` object containing `{x, y, width, height}`
- **Detect API**: Returns flat predictions with `{x, y, width, height, class, confidence}`
- `imageInference()` and `processDrop()` map Detect API format to InferenceEngine format

## Configuration

### API Keys

The application requires Roboflow API keys defined as constants:
- `API_KEY`: Publishable key for InferenceEngine/roboflow client (currently set in code)
- `DETECT_API_KEY`: API key for direct Detect API calls (currently set in code)

**Security Note**: These keys are currently hardcoded but should be moved to environment variables or a config file for production use.

### Available Models

Default models configured:
- `microsoft-coco`: General object detection (80 COCO classes), confidence 0.6
- `construction-site-safety`: Worksite safety detection, confidence 0.3
- `containers-detection-db0c2`: Logistics/container detection, confidence 0.3
- `sku-110k`: Retail product detection, confidence 0.3

To add a new model, add an entry to the `available_models` object following the existing pattern.

## Common Workflows

### Adding a New Model

1. Add model configuration to `available_models` object in the JS file
2. Include model name, version, confidence threshold
3. Provide 4 sample images in `imageGrid` array
4. Model will automatically appear in the dropdown selector

### Modifying Inference Logic

1. Identify which JS file is loaded in `index.html`
2. Key functions to modify:
   - `webcamInference()`: Real-time webcam detection loop
   - `imageInference(e)`: Static image detection from sample grid
   - `processDrop(e)`: Drag-and-drop image detection
   - `drawBoundingBoxes()`: Visualization of detection results

### Bounding Box Rendering

The `drawBoundingBoxes()` function:
- Assigns random colors from `color_choices` to each class
- Caches colors in `bounding_box_colors` for consistency
- Draws 4px stroke rectangles around detections
- Renders class name + confidence % as labels with colored backgrounds

## Mobile Support

The application detects mobile screens with `window.matchMedia("(max-width: 500px)")` and adjusts canvas dimensions to fill the screen. Mobile users can access the "Try on Mobile" option which displays a QR code.

## Dependencies

- **Vite** (^5.3.5): Development server and build tool
- **inferencejs** (1.0.11): Loaded from CDN in `index.html` for browser-based ML inference
- **roboflow**: Also available via CDN for legacy API support

The project loads external dependencies via CDN rather than npm packages for browser compatibility.
