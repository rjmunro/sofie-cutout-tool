# Sofie Cutout Tool - Experimental TensorFlow Branch

> **⚠️ Experimental Branch**: This is a complete reimplementation of the cutout tool using AI-powered automatic reframing. This branch (`feat/experimental-tensorflow`) is not compatible with the master branch.

## Overview

This experimental version uses machine learning (TensorFlow) to automatically detect subjects (faces, people, objects) in SDI video input and intelligently reframe/crop the video to a 1:1 aspect ratio suitable for social media or portrait displays. The reframed output is delivered via NDI.

### Key Features

- **AI-Powered Detection**: Uses TensorFlow.js with BlazeFace and COCO-SSD models for face, person, animal, and object detection
- **Intelligent Reframing**: Automatically crops video to 1:1 aspect ratio based on detected subjects
- **Smooth Motion**: Physics-based camera movement with acceleration/deceleration limits for professional-looking reframes
- **SDI Input**: Captures video from Blackmagic DeckLink cards via macadam
- **NDI Output**: Streams reframed video via NDI using grandiose
- **OSC Control**: Remote control via OSC protocol (port 10024) for manual overrides
- **Scene Change Detection**: Automatically resets framing on scene cuts
- **Audio Passthrough**: Maintains audio from SDI input to NDI output

## Architecture

This is a **Node.js/TypeScript streaming pipeline** (not Electron) built with:

- **Redioactive**: Reactive stream processing framework
- **Beamcoder**: FFmpeg bindings for video processing
- **Macadam**: Blackmagic DeckLink capture
- **Grandiose**: NDI output
- **TensorFlow.js**: Machine learning inference

### Pipeline Flow

```
SDI Input (macadam)
  ↓
Video → Format Conversion → Scene Detection → AI Analysis → Reframing → NDI Output
  ↓
Audio → Format Conversion → NDI Output
```

## Requirements

### Hardware
- Blackmagic DeckLink card (or compatible SDI input device)
- NDI-capable receiver/monitor

### Software
- Node.js >= 14.16
- Yarn 1.x
- FFmpeg libraries (via beamcoder)
- Blackmagic Desktop Video drivers

## Installation

```bash
yarn install
```

## Building

```bash
yarn build
```

This compiles TypeScript to the `dist/` directory.

## Usage

```bash
node dist/index.js
```

### OSC Control (Port 10024)

The tool listens for OSC messages to manually override crop parameters:

- `/oscControl/slider1` - Set crop X position (0-840)
- `/oscControl/slider2` - Set crop width (overrides auto-detection)
- Additional sliders available for testing

Send OSC messages to `localhost:10024` using tools like TouchOSC or custom controllers.

## Configuration

The `config/` directory contains JSON configuration files (not currently used in this experimental version - settings are hardcoded in source).

## Code Structure

- [src/index.ts](src/index.ts) - Main pipeline setup and connection
- [src/sdiProducer.ts](src/sdiProducer.ts) - SDI capture via macadam/DeckLink
- [src/ndiProducer.ts](src/ndiProducer.ts) - NDI input (alternative source)
- [src/analyse.ts](src/analyse.ts) - TensorFlow model loading and frame analysis
- [src/reframer.ts](src/reframer.ts) - Scene detection, AI analysis coordination, and intelligent cropping logic
- [src/ndiConsumer.ts](src/ndiConsumer.ts) - NDI output
- [src/redio.ts](src/redio.ts) - OSC server setup
- [src/trace.ts](src/trace.ts) - Performance tracing utilities

## How Reframing Works

1. **Scene Detection**: Uses FFmpeg's `scdet` filter to detect scene changes
2. **AI Analysis**: Every 12 frames (0.5s at 25fps), analyzes video using:
   - BlazeFace for face detection
   - COCO-SSD for person/object detection
3. **Target Calculation**: Determines optimal crop position based on detected subjects
4. **Smooth Movement**: Applies physics-based motion with:
   - Maximum acceleration: 2 pixels/frame²
   - Maximum deceleration: 3 pixels/frame²
   - Prevents jarring camera movements
5. **Buffering**: Maintains 6-frame buffer (2 seconds) for smooth output

## Differences from Master Branch

This experimental branch is a **complete ground-up rewrite**:

| Master Branch | Experimental Branch |
|--------------|---------------------|
| Electron app with UI | Node.js CLI tool |
| Manual cutout configuration | AI-powered automatic detection |
| CasparCG output | NDI output |
| Web-based control | OSC control |
| Configuration files active | Hardcoded settings |

## Development Notes

- Frame analysis is disabled by default (see line 61-79 in [src/reframer.ts](src/reframer.ts#L61-L79)) for performance testing
- The tool expects 1920x1080i50 input (HD interlaced 50Hz)
- Output is 1080x1080 square format
- Worker threads are set up for ML inference but may not be fully utilized

## Linting & Code Style

```bash
yarn lint          # Check code style
yarn lint-fix      # Auto-fix issues
```

Uses `@sofie-automation/code-standard-preset` for linting and formatting.

## License

MIT

## Related Projects

- Master branch: Traditional Electron-based cutout tool
- [Sofie TV Automation](https://github.com/nrkno/sofie-core)
- [Blackmagic macadam](https://github.com/Streampunk/macadam)
- [NDI grandiose](https://github.com/Streampunk/grandiose)
