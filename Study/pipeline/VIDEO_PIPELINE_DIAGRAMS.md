# Video Pipeline - Architecture Diagrams & Visual Reference

## Complete Video Processing Pipeline Flow

### System-Level Block Diagram

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                        INNOFUSION VIDEO MEDIASERVER                    ┃
┃                         Integrated Video Pipeline                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

    ┌─────────────────────────────────────────┐
    │        IMAGE SENSOR INPUT               │
    │  (Bayer 3840×2160 @ 30fps, ~12-bit)    │
    └────────────────┬────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │   VIF (Video Input    │  Captures raw sensor data
         │   Front End) [Dev: 0] │  into system buffers
         └───────────┬───────────┘
                     │
    ┌────────────────▼────────────────┐
    │  ISP (Image Signal Processing)   │  Bayer-to-RGB conversion
    │  [Device: 0, Channel: 0]        │  Auto exposure/white balance
    │                                  │  3D Noise reduction
    │  ◇ AE (Auto Exposure)            │  Gamma, color matrix
    │  ◇ AWB (White Balance)           │  LDC (Lens Distortion Correct)
    │  ◇ 3DNR (Noise Reduction)        │
    │  ◇ Gamma Correction              │
    │  ◇ Color Enhancement             │  Output: YUV420 planar
    └────────────────┬────────────────┘
                     │
       ┌─────────────▼─────────────┐
       │   Multiple Output Ports   │
       │    (Optional Overlays)    │
       │                           │
       │  ┌──────────────────────┐ │
       │  │  OSD/Region Manager  │ │  Optional: Text, logo overlay
       │  │      (RGN)           │ │
       │  └──────────────────────┘ │
       │                           │
       └─────────────┬─────────────┘
                     │
        ┌────────────▼────────────┐
        │   SCL Device 0          │  Primary multi-resolution scaler
        │   [Scaler Channel: 0]   │
        │                          │
        │ Three Output Ports:      │
        │  • Port 0: Full Res      │
        │  • Port 1: Sub Res       │
        │  • Port 2: JPEG Res      │
        └─┬──┬──────────┬──────────┘
          │  │          │
    ┌─────▼──▼──────┐   │
    │                │   │
    │ SCL Device 1   │   │  Secondary Scaler (Optional)
    │ [Optional AI]  │   │  For low-res AI/VDF processing
    │                │   │
    │ Channels:      │   │
    │ • IPU (AI)     │   │
    │ • VDF (Motion) │   │
    └────────────────┘   │
         │               │
         │      ┌────────▼────────┐
         │      │                 │
    ┌────▼──────▼───────────────┐ │
    │   VENC Encoder Block      │ │
    │                            │ │
    │  ┌──────────────────────┐  │ │
    │  │  VENC DEV 0 (Main)   │  │ │
    │  │  ┌────────────────┐  │  │ │
    │  │  │ Channel 0      │  │  │ │  H265 Main Stream
    │  │  │ 3840×2160      │──┼──┼──→ RTSP Stream
    │  │  │ 2 Mbps H265    │  │  │ │
    │  │  └────────────────┘  │  │ │
    │  │  ┌────────────────┐  │  │ │
    │  │  │ Channel 1      │  │  │ │  H264 Sub Stream
    │  │  │ 1920×1080      │──┼──┼──→ RTSP Stream / Preview
    │  │  │ 1 Mbps H264    │  │  │ │
    │  │  └────────────────┘  │  │ │
    │  │  ┌────────────────┐  │  │ │
    │  │  │ Channel 3-4    │  │  │ │  Optional AI/VDF
    │  │  │ 640×360        │  │  │ │
    │  │  └────────────────┘  │  │ │
    │  └──────────────────────┘  │ │
    │                            │ │
    │  ┌──────────────────────┐  │ │
    │  │  VENC DEV 8 (JPEG)   │  │ │
    │  │  ┌────────────────┐  │  │ │
    │  │  │ Channel 2      │  │  │ │  JPEG Snapshots
    │  │  │ 640×360        │──┼──▼──→ Web Interface
    │  │  │ JPEG 75% Qual  │  │    │
    │  │  └────────────────┘  │    │
    │  └──────────────────────┘    │
    └────────────────────────────────┘
         │                    │
    ┌────▼───────────────────▼────┐
    │   OUTPUT MANAGEMENT LAYER    │
    │                               │
    │  ┌──────────────────────────┐ │
    │  │   Prebuffer Manager      │ │
    │  │   (100-frame circular)   │ │
    │  │                          │ │
    │  │   Enables:               │ │
    │  │   • Pre-event recording  │ │
    │  │   • Smooth playback      │ │
    │  │   • Frame synchronization│ │
    │  └──────────────────────────┘ │
    │                               │
    │  ┌──────────────────────────┐ │
    │  │   NAL Unit Extraction    │ │
    │  │   & Packetization        │ │
    │  └──────────────────────────┘ │
    │                               │
    │  ┌──────────────────────────┐ │
    │  │   RTSP Server Interface  │ │
    │  └──────────────────────────┘ │
    └────┬──────────────────────────┘
         │
         ├──────────────▬──────────────┬────────────────┐
         │              │              │                │
    ┌────▼──┐   ┌──────▼──┐   ┌──────▼──┐      ┌─────▼─────┐
    │ RTSP   │   │ Debug   │   │ File    │      │ Network   │
    │ Client1│   │ File    │   │ Dump    │      │ Client    │
    │(1080p) │   │ Output  │   │(H264)   │      │ (4K)      │
    └────────┘   └─────────┘   └─────────┘      └───────────┘
```

---

## Detailed Codec & Bitrate Flow Diagram

```
RESOLUTION HIERARCHY & DATA FLOW:

Sensor → ISP → SCL Multi-Output Port Structure

                    ISP Output (3840×2160)
                            │
            ┌───────────────┼───────────────┐
            │               │               │
        ┌───▼───┐       ┌───▼───┐      ┌───▼───┐
        │ Port 0│       │ Port 1│      │ Port 2│
        └───┬───┘       └───┬───┘      └───┬───┘
            │               │             │
            │               │             │
        ┌───▼─────┐    ┌────▼────┐   ┌───▼───┐
        │3840×2160│    │1920×1080│   │640×360│
        └───┬─────┘    └────┬────┘   └───┬───┘
            │               │            │
        ┌───▼──────────────────────────────┐
        │ VENC Module Selection            │
        │                                  │
    ┌───┴─────────────┬───────────────┬───┴────┐
    │                 │               │        │
┌───▼─────┐    ┌─────▼──────┐  ┌────▼────┐ ┌─▼────────┐
│ VENC Ch0│    │ VENC Ch1   │  │VENC Ch2 │ │ VENC Ch3 │
│ H265    │    │ H264       │  │ JPEG    │ │ Optional │
│ 2 Mbps  │    │ 1 Mbps     │  │ 75% Qua │ │ IPU/VDF  │
│ Main    │    │ Sub        │  │ Snapshot│ │ Stream   │
└────┬────┘    └─────┬──────┘  └────┬────┘ └──────────┘
     │              │              │
     │ HW_RING      │ HW_RING      │ FRAME_BASE
     │ Mode         │ Mode         │ Mode
     │              │              │
     │              │              │
 ┌───▼──────────────▼──────────────▼────┐
 │     Network Streaming Output          │
 │                                        │
 │  Primary Stream:                       │
 │  • Client 1: 3840×2160 H265 2Mbps     │
 │  • Maximum Quality                     │
 │  • Lowest Latency (HW Ring)           │
 │                                        │
 │  Secondary Stream:                     │
 │  • Client 2: 1920×1080 H264 1Mbps     │
 │  • Preview/Mobile                      │
 │  • Backup/Preview Use                  │
 │                                        │
 │  Snapshot Service:                     │
 │  • On-demand JPEG                      │
 │  • Web thumbnail                       │
 └────────────────────────────────────────┘
```

---

## Binding Topology Comparison

### Configuration A: Simple Single-Stream (Lowest CPU)
```
        Sensor
          │
         VIF
          │
         ISP
          │
        SCL (1 output)
          │
       VENC_H265
          │
       RTSP Stream
       (3840×2160)

Characteristics:
✓ Lowest CPU usage
✓ Highest resolution
✗ No backup stream
```

### Configuration B: Dual-Stream Scaler-Based (Frame-Based)
```
        Sensor
          │
         VIF
          │
         ISP
          │
        SCL (3 outputs)
          │
    ┌─────┼──────┐
    │     │      │
  Port0  Port1  Port2
    │     │      │
    ▼     ▼      ▼
  VENC0 VENC1  VENC2
  H265  H264   JPEG
    │     │
    ▼     ▼
  Stream1 Stream2

Data Flow:
SCL Port0(3840×2160) → VENC Ch0(H265) → Main Stream
SCL Port1(1920×1080) → VENC Ch1(H264) → Sub Stream
SCL Port2(640×360)   → VENC Ch2(JPEG) → Snapshot

Characteristics:
✓ Multiple resolutions
✓ Independent scaling
✓ Snapshot capability
◆ Moderate CPU usage
◆ Latency: ~100-150ms
```

### Configuration C: Dual-Stream HW-Ring Mode (Optimized)
```
        Sensor
          │
         VIF
          │
         ISP
          │
        SCL
          │
        VENC_Ch0 (H265) ◄─ HW Ring Input
        3840×2160
          │
          ├─→ Main Stream Output
          │
          └─→ VENC_Ch1 (H264) ◄─ HW Ring Interconnect
              1920×1080 (From Ch0)
              │
              └─→ Sub Stream Output

Data Flow:
Channel 0 (Main):
  Input: SCL (HW Ring) → Output: H265 3840×2160 → RTSP

Channel 1 (Sub - derives from Ch0):
  Input: VENC Ch0 (HW Ring) → Output: H264 1920×1080 → RTSP

Characteristics:
✓ Lowest latency (HW Ring)
✓ Efficient sub-stream generation
✓ Reduced memory footprint
✓ Optimal for RTSP streaming
◆ Fixed resolution ratio
```

### Configuration D: Full-Featured (AI/VDF Support)
```
        Sensor
          │
         VIF
          │
         ISP
          │
    ┌─────┴─────┐
    │           │
  SCL_0      SCL_1
    │       (Dev1)
    │           │
 3-Port      2-Channel
    │      ┌────┴────┐
    │      │         │
    ▼   Ch_IPU  Ch_VDF
 [Enc]      │       │
 ┌┬┬┬       ▼       ▼
 ||||     [YUV]  [Motion]
 ││││      │       │
 ││││   To AI   To Edge
 ││││      │      Proc
 ││││     VDF    Module
 ││││      │
  ├─→ H265 Main Stream
  ├─→ H264 Sub Stream
  ├─→ JPEG Snapshot
  └─→ YUV Raw (IPU/VDF)

Characteristics:
✓ Multi-purpose pipeline
✓ AI/Detection integration
✓ Motion analysis capability
✓ Maximum flexibility
◆ Highest CPU usage
◆ Requires dual SCL
```

---

## Timing & Synchronization Diagram

```
FRAME TIMING ACROSS PIPELINE:

Sensor Frame Rate: 30 FPS (33.33ms per frame)

┌─ Frame n
│ ┌─────────────────────────────┐
│ │ Sensor Capture              │ 0ms
│ │ (Bayer pattern from CMOS)   │
│ └─────────────┬───────────────┘
│               │
│               ▼ VIF Capture (~1ms)
│ ┌─────────────────────────────┐
│ │ ISP Processing              │ ~3-5ms
│ │ • Bayer demosaic            │
│ │ • Color matrix              │
│ │ • 3DNR filtering            │
│ └─────────────┬───────────────┘
│               │
│               ▼ SCL Scaling (~2-3ms)
│ ┌─────────────────────────────┐
│ │ Scaler Output               │ ~5-8ms
│ │ • Multiple resolutions      │
│ │ • Quality preservation      │
│ └─────────┬───────────┬───────┘
│           │           │
│           ▼           ▼ Parallel Processing
│ ┌──────────┐    ┌──────────┐
│ │ VENC H265│    │ VENC H264│ ~10-15ms
│ │ Encode   │    │ Encode   │
│ │Ch0       │    │Ch1       │
│ └─────┬────┘    └─────┬────┘
│       │              │
│       │              ▼ (15-20ms)
│       ▼ Encoding Complete
│ ┌─────────────────────────────┐
│ │ Stream Output               │ ~20-25ms
│ │ • RTSP packetization        │
│ │ • Network transmission      │
│ └──────────────────────────────┘
│
└─ Frame n+1 (33.33ms later)

Total End-to-End Latency:
• Sensor → Output: ~20-30ms (typical)
• Prebuffer adds: +100 frames × 33.33ms = ~3.3 seconds
• Network: +variable (depends on bandwidth)

Synchronization Points:
✓ VIF manages frame capture timing
✓ ISP processes at fixed intervals
✓ SCL maintains rate (frame drops if needed)
✓ VENC produces stream with PTS (Presentation Time Stamp)
✓ Prebuffer circular array maintains frame order

Rate Control:
If pipeline can't keep up:
• Frame drops at scaler (FRAME_BASE mode)
• Ring buffer flush (HW_RING mode)
• Encoder skips frames (if configured)
```

---

## Memory Layout Diagram

```
SYSTEM MEMORY ORGANIZATION:

┌────────────────────────────────────────────────────────┐
│              SYSTEM MEMORY (DDR)                       │
├────────────────────────────────────────────────────────┤
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  VIF/ISP Pipeline Buffers                      │   │
│  │  ┌─────────────────────────────────────────┐  │   │
│  │  │ Raw Bayer Input: 3840×2160×2 = 16.6 MB │  │   │
│  │  │ (8-bit per channel, CFA pattern)        │  │   │
│  │  └─────────────────────────────────────────┘  │   │
│  │  ┌─────────────────────────────────────────┐  │   │
│  │  │ ISP Output YUV: 3840×2160×1.5 = 12.4 MB│  │   │
│  │  │ (Multiple buffers: 3-4 frames)          │  │   │
│  │  └─────────────────────────────────────────┘  │   │
│  │  Size: ~50-60 MB                              │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  SCL Buffers (Multi-Resolution Scaling)       │   │
│  │  ┌─────────────────────────────────────────┐  │   │
│  │  │ Port 0 (3840×2160×1.5): ~12 MB × 2     │  │   │
│  │  │ Port 1 (1920×1080×1.5): ~3 MB × 2      │  │   │
│  │  │ Port 2 (640×360×1.5): ~0.3 MB × 2      │  │   │
│  │  │ (Ping-pong buffers for each port)       │  │   │
│  │  └─────────────────────────────────────────┘  │   │
│  │  Size: ~30-40 MB                              │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  VENC Encoder Buffers                          │   │
│  │  ┌─────────────────────────────────────────┐  │   │
│  │  │ H265 Encoder Ring: ~30 MB               │  │   │
│  │  │ H264 Encoder Ring: ~15 MB               │  │   │
│  │  │ JPEG Encoder: ~5 MB                     │  │   │
│  │  │ (Hardware-managed ring buffers)         │  │   │
│  │  └─────────────────────────────────────────┘  │   │
│  │  Size: ~50-60 MB                              │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  Prebuffer Storage (Circular)                  │   │
│  │  ┌─────────────────────────────────────────┐  │   │
│  │  │ Video Frames: 100 × 200 KB avg = 20 MB │  │   │
│  │  │ (NAL units + headers for each frame)    │  │   │
│  │  │ Ring index: write_pos = 0-99            │  │   │
│  │  │ With DTS/PTS timestamps                 │  │   │
│  │  └─────────────────────────────────────────┘  │   │
│  │  Size: ~20-30 MB                              │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│  ┌────────────────────────────────────────────────┐   │
│  │  System & Control Structures                   │   │
│  │  • Configuration tables: ~1 MB                 │   │
│  │  • Thread stacks: ~10 MB                       │   │
│  │  • ISP calibration data: ~5 MB                 │   │
│  │  Size: ~20 MB                                  │   │
│  └────────────────────────────────────────────────┘   │
│                                                         │
│  Total Video Pipeline Memory: ~200-250 MB typical    │
│  (Varies based on resolution & frame depth)          │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## State Machine: Stream Initialization

```
STREAM INITIALIZATION SEQUENCE:

                    START
                      │
                      ▼
            ┌─────────────────┐
            │  Load Config    │
            │ mediaserver.cfg │
            └────────┬────────┘
                     │
                     ▼
            ┌─────────────────────┐
            │ Initialize VIF      │
            │ CreateDevGroup()    │
            │ Status: VIF_READY   │
            └────────┬────────────┘
                     │
                     ▼
            ┌─────────────────────┐
            │ Initialize ISP      │
            │ CreateDevice()      │
            │ CreateChannel()     │
            │ Status: ISP_READY   │
            └────────┬────────────┘
                     │
                     ▼
            ┌─────────────────────┐
            │ Initialize SCL      │
            │ CreateDevice()      │
            │ CreateChannel()     │
            │ Status: SCL_READY   │
            └────────┬────────────┘
                     │
                     ▼
            ┌─────────────────────┐
            │ Initialize VENC     │
            │ CreateDev(0)        │
            │ CreateDev(8)        │
            │ CreateChn() ×4      │
            │ Status: VENC_READY  │
            └────────┬────────────┘
                     │
                     ▼
            ┌──────────────────────┐
            │ Bind Modules         │
            │ VIF → ISP           │
            │ ISP → SCL           │
            │ SCL → VENC          │
            │ Status: BOUND       │
            └────────┬─────────────┘
                     │
                     ▼
            ┌──────────────────────┐
            │ Configure Encoders   │
            │ Set VUI params       │
            │ Set Rate Control     │
            │ Enable Channels      │
            │ Status: CONFIGURED   │
            └────────┬─────────────┘
                     │
                     ▼
            ┌──────────────────────┐
            │ Start RTSP Server    │
            │ Create Stream threads│
            │ Start Prebuffering   │
            │ Status: STREAMING    │
            └────────┬─────────────┘
                     │
                     ▼
            ┌──────────────────────┐
            │   READY              │
            │ Streams Active       │
            │ Ready for clients    │
            └──────────────────────┘

Error Recovery Path:
    │
    └─ If error at any stage
       └─ Rollback previous initialization
       └─ Free allocated resources
       └─ Log error
       └─ Return to START or CONFIG state
```

---

## Performance Profile: Resolution vs CPU

```
CPU LOAD vs VIDEO RESOLUTIONS

                    CPU Usage (%)
                         │
                    100  │     ┌─ 4K×2 Stream (H265+H264)
                         │    /│
                     90  │   / │
                         │  /  │
                     80  │ /   │ ┌─ 4K×1 Stream (H265 only)
                         │/    │/
                     70  │     │
                         │     │  ┌─ 1080p×2 (H264+H265)
                     60  │     │ /│
                         │     │/ │
                     50  │     │  │
                         │     │  │ ┌─ 720p×3 (Multi-stream)
                     40  │     │  │/
                         │     │  │
                     30  │     │  │
                         │     │  │
                     20  │     │  │
                         │     │  │
                     10  │     │  │
                         │     │  │
                      0  │─────┴──┴──────────→ Encoding Quality
                              │  │  │
                           Low Med High

Typical Performance:
• 1 × 3840×2160 H265:     ~25% CPU
• 1 × 3840×2160 H264:     ~30% CPU
• 2 × 1920×1080 (Mixed):  ~35% CPU
• 1 × 3840×2160 + 1 × 1920×1080: ~40% CPU
• Maximum recommended: <70% CPU for stability
```

---

## Network Bandwidth Profile

```
BITRATE ALLOCATION

Main Stream H265 (3840×2160):
├─ I-frame (IDR): ~500 KB (1-2 per second)
├─ P-frame: ~50-100 KB
├─ B-frame: ~30-50 KB
├─ Target bitrate: 2 Mbps (250 KB/s)
└─ Actual: 1.8-2.2 Mbps (varies with scene)

Sub Stream H264 (1920×1080):
├─ I-frame (IDR): ~200 KB (1-2 per second)
├─ P-frame: ~15-30 KB
├─ Target bitrate: 1 Mbps (125 KB/s)
└─ Actual: 0.9-1.2 Mbps (varies with scene)

Network Requirements:
           Bandwidth    Storage/Hr    Cache (ms)
Main       2.2 Mbps     ~1 GB         1000
Sub        1.2 Mbps     ~0.5 GB       1000
Combined   3.4 Mbps     ~1.5 GB       2000

Multiple Clients Scaling:
1 Main Client:   2.2 Mbps
2 Main Clients:  4.4 Mbps
1 Main + 1 Sub:  3.4 Mbps
```

---

