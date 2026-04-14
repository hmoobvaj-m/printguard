```bash
                                ┌──────────────────────────────────────────┐
                                │              Docker Network              │
                                │                                          │
        ┌──────────┐    MJPEG   │  ┌───────────────────────────────────┐   │
        │  Webcam  │  ────────────>│           Monitor Service         │   │
        └──────────┘            │  │         (core/monitor.py)         │   │
                                │  │                                   │   │
                                │  │  asyncio loop — every N seconds:  │   │
                                │  │   1. Pull frame from FrameSource  │   │
                                │  │   2. Send to VisionAnalyzer       │   │
                                │  │   3. Evaluate DetectionResult     │   │
                                │  │   4. Emit event to Queue          │   │
                                │  └────────────────┬──────────────────┘   │
                                │                   │ asyncio.Queue        │
                                │           ┌───────┴─────────┐            │
                                │           ▼                 ▼            │
                                │       ┌─────────┐     ┌───────────┐      │
                                │       │Moonraker│     │    DB     │      │
                                │       │ Client  │     │Repository │      │
                                │       └────┬────┘     └─────┬─────┘      │
                                │            │                │            │
                                │            ▼                ▼            │
                                │       ┌─────────┐      ┌──────────┐      │
                                │       │ Klipper │      │ SQLite   │      │
                                │       │Moonraker│      │   DB     │      │
                                │       └─────────┘      └──────────┘      │
                                │            │                │            │
                                │  ┌───────────────────────────────────┐   │
                                │  │          FastAPI Backend          │   │
                                │  │  /api/v1/  + WebSocket            │   │
                                │  └───────────────────────────────────┘   │
                                │                    │                     │
                                └────────────────────│─────────────────────┘
                                                     │ HTTP / WebSocket
                                                ┌────▼─────┐
                                                │  React   │
                                                │ Frontend │
                                                └──────────┘
```
## Overview
Description of the system architecture that combines video processing with 3D printing services via Klipper/Moonraker

## Tech Stack
- **Frontend**: React + Tailwind
- **Backend**: FastAPI (Python)
- **Vision**: Llama.cpp Server (swappable via Protocol)
- **Database**: SQLite
- **Containerization**: Docker

## Core Components

### 1. Monitor Service (`core/monitor.py`)
- Runs async loop every N seconds
- Pulls frames from video source
- Processes frames through VisionAnalyzer
- Evaluates DetectionResults
- Emits events to asyncio.Queue

### 2. Vision Processing
- **Vision Analyzer**: Processes video frames for object detection
- **Detection Results**: Structured output from vision processing
- **Protocol Interface**: Swappable vision backend (currently llama.cpp)

### 3. Data Flow Components
- **Event Queue**: async queue for communication between components
- **Moonraker Client**: Communicates with Klipper/Moonraker for printer status
- **Database Repository**: Stores processed events and system state

### 4. API Layer
- **FastAPI Backend**: Provides RESTful API endpoints and WebSocket connections
- **WebSocket Interface**: Real-time communication with frontend

### 5. Frontend
- **React Application**: User interface for monitoring and control
- **Tailwind CSS**: Styling framework


## Data Flow Description
1. **Video Input**: Webcam provides MJPEG stream to Monitor Service
2. **Processing Pipeline**: 
   - Monitor Service pulls frames from FrameSource
   - Frames processed by VisionAnalyzer
   - DetectionResults evaluated and emitted to Queue
3. **System Integration**:
   - Events sent through asyncio.Queue to Moonraker Client
   - Moonraker Client communicates with Klipper for printer status
4. **User Interface**: 
   - FastAPI Backend provides WebSocket connection
   - React Frontend displays real-time monitoring data

## Deployment Architecture
- All components containerized in Docker
- Services communicate over Docker network
- SQLite database persists local state
- Moonraker integration enables 3D printer monitoring