                                ┌──────────────────────────────────────────┐
                                │              Docker Network              │
                                │                                          │
        ┌──────────┐    MJPEG   │  ┌───────────────────────────────────┐   │
        │  Webcam  │  ────────────>           Monitor Service          │   │
        └──────────┘            │  │         (core/monitor.py)         │   │
                                │  │                                   │   │
                                │  │  asyncio loop — every N seconds:  │   │
                                │  │   1. Pull frame from FrameSource  │   │
                                │  │   2. Send to VisionAnalyzer       │   │
                                │  │   3. Evaluate DetectionResult     │   │
                                │  │   4. Emit event to Queue          │   │
                                │  └────────────┬──────────────────────┘   │
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