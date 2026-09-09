# CloudCam - Automated Sky Imaging System

A high-sensitivity, fully automated capture, overlay, and timelapse system for CFHT CloudCams that replaces the original DSLR system with a modern, Python-driven stack.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
  - [Core Functionality](#core-functionality)
- [Installation](#installation)
  - [Prerequisites](#prerequisites)
  - [Camera Configuration](#camera-configuration)
- [Project Structure](#project-structure)
- [Module Documentation](#module-documentation)
  - [`automated_process.py` - Main Daemon](#automated_processpy---main-daemon)
  - [`astrometry.py` - Celestial Mechanics](#astrometrypy---celestial-mechanics)
  - [`auto_brightness.py` - Exposure Control](#auto_brightnesspy---exposure-control)
  - [`take_image.py` - Camera Interface](#take_imagepy---camera-interface)
  - [`timelapse.py` - Video Generation](#timelapsepy---video-generation)
- [Usage](#usage)
  - [Basic Operation](#basic-operation)
  - [Web Interface](#web-interface)
- [License](#license)

## Overview

CloudCam is an intelligent astronomical imaging system that:
- Captures high-resolution sky frames only during astronomical night
- Self-adjusts exposure and gain to handle rapid dusk/dawn changes
- Recalibrates its WCS (World Coordinate System) solution whenever enough stars are visible
- Generates transparent overlays for Western and/or Hawaiian constellations, planets, and ISS
- Displays live frames in a real-time Matplotlib viewer
- Uploads current images and timelapses to the CFHT CloudCam website
- Creates nightly timelapse videos and lightweight 30-minute "recent activity" loops

> **Development history:** This repository was transferred from CFHT's private GitLab instance to GitHub with permission at the end of my internship. Most of the original version history remains in the company GitLab, so this repository begins as a snapshot of the project near the end of development.

## Features

### Core Functionality
- **Automated Night Detection**: Uses observatory telemetry to determine sunset/sunrise times
- **Adaptive Exposure Control**: Automatically adjusts camera settings for optimal sky brightness
- **Astrometric Calibration**: Real-time WCS solving and sky coordinate tracking
- **Cultural Star Overlays**: Support for both Western and Hawaiian constellation patterns
- **Planetary Tracking**: Real-time ephemeris calculation for planets, Sun, Moon, and ISS
- **Live Monitoring**: Real-time image display and status logging
- **Timelapse Generation**: Automated creation of nightly MP4 timelapses
- **Web Interface**: PHP frontend backed by FastAPI for live camera viewing, overlay selection, and timelapse generation

## Installation

### Prerequisites
Check the Dockerfile in the repository for complete dependencies. Key requirements include:
- Python 3.x
- OpenCV
- NumPy
- Matplotlib (TkAgg backend)
- Photutils
- Astropy
- Skyfield
- FastAPI (for web interface)

### Camera Configuration

The camera connection is configured with environment variables rather than a hardcoded observatory address:

```bash
export CLOUDCAM_HOST=<camera-host>
export CLOUDCAM_PORT=<camera-port>
```

`CLOUDCAM_HOST` defaults to `localhost` and `CLOUDCAM_PORT` defaults to `915` for local development.

## Project Structure

Below is an overview of the `CloudCam` repository's file and folder structure, along with brief descriptions of each component.

```
CloudCam-main/
├── .gitignore                  # Ignore rules for Git version control
├── .gitlab-ci.yml              # GitLab CI/CD pipeline configuration
├── Dockerfile                  # Docker image setup for deployment
├── README.md                   # Main project documentation
├── docker-compose.yml          # Docker multi-container setup
├── requirements.txt            # Python package dependencies

├── Conceptual_design/          # Project design and planning
│   ├── .gitkeep
│   ├── cloudcam_requirements_conceptual_design.md
│   └── initial.wcs

├── astrometry-net-indexes/     # Star calibration index files
│   ├── index-4118.fits
│   └── index-4119.fits

├── astrometrynet_files/        # Supporting files for calibration
│   ├── astrometry-ngc.png
│   └── initial_wcs_values.txt

├── data/                       # Astronomical data inputs
│   ├── de442s.bsp
│   ├── ephem.cat
│   ├── hawaiian_const.lines
│   ├── hawaiian_const.stars
│   └── tle.txt

├── fastapi/                    # Backend API service
│   ├── fastapi_astrometry.py   # API endpoint for astrometry operations
│   ├── fastapi_timelapse.py    # API endpoint for timelapse generation
│   ├── main.py                 # FastAPI app entry point
│   ├── requirements.txt        # Dependencies for the API service
│   └── timelapse_overlay.py    # Applies requested overlays to timelapse frames

├── src/                        # Core logic and automation modules
│   ├── astrometry.py           # Star pattern solving and celestial alignment
│   ├── auto_brightness.py      # Dynamic brightness and auto-exposure logic
│   ├── automated_process.py    # Main orchestration pipeline
│   ├── take_image.py           # Camera control and image acquisition
│   └── timelapse.py            # Timelapse creation logic

└── website/                    # PHP frontend for CloudCam viewing and controls
    ├── about.php               # CloudCam information page
    ├── index.php               # Live camera and overlay-selection interface
    └── timelapse2025.php       # Timelapse browsing and custom overlay generation
```

## Module Documentation

### `automated_process.py` - Main Daemon
The core process that orchestrates the entire system:
- Creates daily directory structures
- Manages nighttime imaging loops
- Handles WCS calibration triggers
- Generates overlays and captions
- Logs operational status to CFHT status server
- Saves images to the CFHT's NAS (CFHT's data storage unit)

**Key Functions:**
- `main()` - Main execution loop
- `image_capture()` - Core nighttime imaging loop
- `capture_with_timeout()` - Robust image capture with retry
- `visible_stars()` - Star detection for WCS triggers

### `astrometry.py` - Celestial Mechanics
Handles all astronomical calculations and overlay generation:
- WCS solving using astrometry.net
- Sky coordinate transformations
- Ephemeris calculations for celestial objects
- Constellation overlay rendering

**Key Functions:**
- `recalibrate_wcs()` - Initial WCS solution
- `repoint_wcs()` - Updates WCS for sky rotation
- `overlay()` - Complete overlay pipeline

### `auto_brightness.py` - Exposure Control
Maintains optimal image brightness through intelligent exposure management:
- Target mean brightness: 52 (configurable)
- Prioritizes exposure time adjustments over gain
- Fast mode for rapid twilight transitions

### `take_image.py` - Camera Interface
Manages TCP communication with the camera server:
- Reads the camera host and port from environment variables
- Retrieves current camera settings
- Captures and saves timestamped images
- Updates camera settings based on brightness analysis
- Saves images to the CFHT's NAS (CFHT's data storage unit)

### `timelapse.py` - Video Generation
Creates MP4 timelapses from image sequences:
- Supports variable frame rates (default: 10 fps)
- Automatic frame resizing
- H.264 encoding
- Saves timelapses to the CFHT's NAS (CFHT's data storage unit)

## Usage

### Basic Operation
1. The system automatically starts imaging at astronomical night
2. Exposure and gain are continuously adjusted for optimal brightness
3. When sufficient stars are detected, WCS calibration is performed
4. Constellation overlays are generated and applied to images
5. Every 30 minutes, a short timelapse preview is created
6. At dawn, a complete nightly timelapse is generated

### Web Interface

[Official CFHT CloudCams website](https://www.cfht.hawaii.edu/en/gallery/cloudcams/)

The web interface supports:
- Live camera feed viewing
- Custom overlay selection for the live camera feed
- Timelapse downloads
- Custom overlay generation for timelapses
- Hawaiian and Western constellation, star, and planet labeling options

## License

This project is licensed under the [MIT License](LICENSE).

You are free to use, modify, and distribute this software for any purpose, including commercial applications, as long as the original license and copyright notice are included.

© 2025 David Samtani
