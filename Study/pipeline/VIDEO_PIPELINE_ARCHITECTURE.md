# Video Pipeline Architecture - Innofusion Media Interface

## Overview
The Innofusion media server implements a sophisticated multi-stream video processing pipeline with support for multiple encoding formats and quality levels. The pipeline supports simultaneous processing of multiple video streams from a single sensor source through various processing stages.

---

## High-Level Video Pipeline Block Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         SENSOR INPUT                             │
│                    (VIF - Video Input Front)                     │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │  VIF Device Group │
                    │  (nVCapGroup)     │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │   ISP Channel     │
                    │  (Image Signal    │
                    │   Processing)     │
                    └─────────┬─────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        │          (Optional) │          (Optional) │
        │                     │                     │
   ┌────▼───┐            ┌───▼────┐           ┌────▼──┐
   │RGN/OSD │            │ VDF    │           │ IPU   │
   │Region  │            │Motion  │           │Image  │
   │Overlay │            │Detection           │Process│
   └────────┘            └────────┘           └───────┘
        │
        └────────────────────┬────────────────────┐
                             │                     │
                    ┌────────▼────────┐           │
                    │  SCL Device 0   │           │
                    │  (Scaler Chn 0) │           │
                    └────────┬────────┘           │
                             │                     │
         ┌───────────────────┼───────────────────┬┘
         │                   │                   │
    ┌────▼──────┐      ┌─────▼────┐      ┌──────▼──┐
    │SCL Port 0 │      │SCL Port 1 │      │SCL Port 2│
    │(Main Res) │      │(Sub Res)  │      │(JPEG)    │
    └────┬──────┘      └─────┬────┘      └──────┬───┘
         │                   │                   │
         │            (Optional Scaler)         │
         │            ┌─────────────┐           │
         │            │SCL Device 1 │           │
         │            └─────────────┘           │
         │                   │                   │
         │       ┌───────────┼───────────┐      │
         │       │           │           │      │
         │  ┌────▼───┐  ┌────▼───┐ ┌────▼──┐   │
         │  │IPU Out │  │VDF Out │ │Other  │   │
         │  └────────┘  └────────┘ └───────┘   │
         │                                       │
         │       HW_RING or FRAME_BASE Mode    │
         │                                       │
    ┌────▼──────────────────────────────────────▼────┐
    │         VENC Module (Video Encoder)            │
    │                                                 │
    │  ┌────────────┬────────────┬────────────┐     │
    │  │  VENC CH0  │  VENC CH1  │  VENC CH2  │     │
    │  │ Main Stream│ Sub Stream │   JPEG     │     │
    │  │  H264/H265 │  H264/H265 │            │     │
    │  │ 3840x2160  │  1920x1080 │  640x360   │     │
    │  │   2Mbps    │   1Mbps    │            │     │
    │  └────────────┴────────────┴────────────┘     │
    │                                                 │
    │  Additional Channels (if enabled):             │
    │  ┌────────────┬────────────┐                  │
    │  │  VENC CH3  │  VENC CH4  │                  │
    │  │ IPU Stream │ VDF Stream │                  │
    │  │   640x360  │   RAW SIZE │                  │
    │  └────────────┴────────────┘                  │
    └────┬──────────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────────┐
    │      OUTPUT PROCESSING & DELIVERY             │
    │                                                 │
    │  ┌──────────────┐  ┌──────────────┐           │
    │  │   RTSP       │  │  File Dump   │           │
    │  │  Streaming   │  │  (for debug) │           │
    │  │              │  │              │           │
    │  │ Main Stream  │  │ Prebuffer:   │           │
    │  │ Sub Stream   │  │ 100 frames   │           │
    │  │ JPEG Capture │  │              │           │
    │  └──────────────┘  └──────────────┘           │
    └──────────────────────────────────────────────┘
```

---

## Detailed Pipeline Architecture

### 1. **Input Stage (VIF - Video Input Front)**
```
VIF Device Group Creation
├── Input source: Image Sensor
├── Device Group ID: nVCapGroup (0)
├── Attributes:
│   ├── Sensor interface configuration
│   ├── Data format (RAW/YUV)
│   └── Frame rate settings
└── Output: RAW video data stream
```

### 2. **ISP Stage (Image Signal Processing)**
```
ISP Device Creation (Dev: 0, Chn: 0)
├── Input: RAW video from VIF
├── Processing:
│   ├── Bayer to RGB conversion
│   ├── Auto Exposure (AE) control
│   ├── White Balance (AWB) adjustment
│   ├── Lens Distortion Correction (LDC) - optional
│   ├── 3D Noise Reduction (3DNR)
│   ├── Gamma correction
│   └── Color enhancement
├── Output ports:
│   ├── Port 0: Full resolution video
│   ├── Port 1: Optional alternative output
│   └── Port 2: Optional alternative output
└── Output: Processed video frames (typically YUV420)
```

**ISP IQ (Image Quality) Controls:**
- Brightness adjustment
- Contrast adjustment
- Saturation adjustment
- Exposure compensation
- Flicker detection/correction
- Day/Night mode switching

### 3. **Optional Processing Modules**

#### 3a. Region/OSD Module (RGN)
```
RGN (Region Manager) - Optional Overlay
├── Input: Video from ISP
├── Processing:
│   ├── Text overlay
│   ├── Logo insertion
│   └── Rectangle/shape drawing
└── Output: Overlaid video
```

#### 3b. VDF (Video Dynamic Filter)
```
VDF Motion Detection - Optional
├── Input: Video frame from SCL
├── Processing:
│   ├── Object Detection
│   ├── Motion Detection
│   └── Metadata generation
└── Output: Motion analysis data
```

#### 3c. IPU (Image Processing Unit)
```
IPU User YUV Processing - Optional
├── Input: Video from SCL
├── Processing:
│   ├── Custom user algorithms
│   ├── GPU/NEON acceleration
│   └── Custom filters
└── Output: Processed YUV data
```

### 4. **Scaling Stage (SCL - Scaler)**

#### Primary Scaler (Device 0)
```
SCL Device 0 Creation (Dev: 0, Chn: 0)
├── Input: Processed video from ISP
├── Processing:
│   └── High-quality downsampling
├── Multiple Output Ports:
│   ├── Port 0: Main Stream
│   │   ├── Resolution: 3840x2160 (4K)
│   │   ├── Target: H265 encoder
│   │   └── Bitrate: 2 Mbps
│   │
│   ├── Port 1: Sub Stream 0
│   │   ├── Resolution: 1920x1080 (1080p)
│   │   ├── Target: H264 encoder
│   │   └── Bitrate: 1 Mbps
│   │
│   └── Port 2: JPEG Capture
│       ├── Resolution: 640x360 (VGA)
│       ├── Target: JPEG encoder
│       └── Use case: Snapshot
```

#### Secondary Scaler (Device 1) - Optional
```
SCL Device 1 Creation (Dev: 1)
├── Input: Video from ISP or Port 0 of Primary Scaler
├── Channels:
│   ├── Channel X: For IPU processing
│   │   └── Resolution: 640x360
│   │
│   └── Channel Y: For VDF processing
│       └── Resolution: VDF_RAW_W x VDF_RAW_H
│
└── Output: Low-resolution streams for AI/detection
```

### 5. **Encoder Stage (VENC - Video Encoder)**

#### VENC Device 0: H264/H265 Encoding
```
VENC Device 0 Creation
├── Channel 0: Main Stream Encoder
│   ├── Codec: H265 (HEVC)
│   ├── Input: SCL Port 0 (3840x2160)
│   ├── Bitrate: 2 Mbps (CBR or VBR)
│   ├── Profile: Main Profile
│   └── Output: H265 bitstream
│
├── Channel 1: Sub Stream Encoder
│   ├── Codec: H264
│   ├── Input: SCL Port 1 (1920x1080) OR VENC Ch0 (HW Ring mode)
│   ├── Bitrate: 1 Mbps (CBR or VBR)
│   ├── Profile: High Profile
│   └── Output: H264 bitstream
│
└── Channel 3-4: Optional AI/VDF Streams
    ├── Input: IPU/VDF outputs
    └── Output: Additional encoded streams
```

#### VENC Device 8: JPEG Encoding
```
VENC Device 8 Creation
└── Channel 2: JPEG Snapshot Encoder
    ├── Codec: JPEG
    ├── Input: SCL Port 2 (640x360)
    ├── Quality: Configurable (0-100)
    └── Output: JPEG image data
```

### 6. **Output/Delivery Stage**

#### Stream Management
```
Output Handler
├── Main Stream (Channel 0)
│   ├── RTSP streaming
│   ├── Circular buffer: 100 frames
│   └── Distribution: Network clients
│
├── Sub Stream (Channel 1)
│   ├── RTSP streaming
│   ├── Lower bandwidth
│   └── Mobile/preview clients
│
└── JPEG (Channel 2)
    ├── Snapshot service
    ├── On-demand capture
    └── Web interface thumbnail
```

#### Prebuffering System
```
Prebuffer Management
├── Video Frame Buffer
│   ├── Depth: 100 frames
│   ├── Format: H264/H265 NAL units
│   ├── Purpose: Pre-event recording
│   └── Circular buffer pattern
│
└── Audio Frame Buffer
    ├── Depth: 100 frames
    ├── Format: PCM/AAC/OPUS
    └── Synchronized with video
```

---

## Data Flow Modes

### Mode 1: Frame-Based Pipeline (E_MI_SYS_BIND_TYPE_FRAME_BASE)
```
VIF → ISP → SCL → VENC (frame buffering at each stage)
├── Characteristics:
│   ├── Lower latency sensitivity
│   ├── Buffer flexibility
│   ├── Used for: JPEG, VDF, IPU outputs
│   └── Typical use: Snapshot, AI processing
```

### Mode 2: Hardware Ring Pipeline (E_MI_SYS_BIND_TYPE_HW_RING)
```
VIF → ISP → SCL →[HW Ring]→ VENC (direct hardware buffer ring)
├── Characteristics:
│   ├── Lowest latency
│   ├── Direct hardware ring buffer
│   ├── Used for: Main stream (H265 Ch0)
│   ├── Sub stream from Main stream (H264 Ch1)
│   └── Typical use: Real-time streaming
```

### Mode 3: Real-Time Pipeline (E_MI_SYS_BIND_TYPE_REALTIME)
```
VIF → ISP → VENC (minimal buffering, frame drop on congestion)
├── Characteristics:
│   ├── Absolute lowest latency
│   ├── Drop frames if pipeline congested
│   └── Used for: Live video feed
```

---

## Stream Configuration Table

| Parameter | Main Stream | Sub Stream 0 | JPEG | Sub Stream 1 | Sub Stream 2 |
|-----------|-------------|------------|------|-------------|-------------|
| **Status** | Enabled | Disabled | Disabled | Disabled (IPU) | Disabled (VDF) |
| **Input Module** | SCL (Port 0) | VENC Ch0 (HW Ring) | SCL (Port 2) | SCL Dev1 | SCL Dev1 |
| **Codec** | H265 | H264 | JPEG | YUV | YUV |
| **Resolution** | 3840x2160 | 1920x1080 | 640x360 | 640x360 | RAW |
| **Bitrate** | 2 Mbps | 1 Mbps | N/A | N/A | N/A |
| **VENC Dev** | 0 | 0 | 8 | 0 | 0 |
| **VENC Chn** | 0 | 1 | 2 | 3 | 4 |
| **Bind Type** | HW_RING | HW_RING | FRAME_BASE | FRAME_BASE | FRAME_BASE |
| **Function** | RTSP | RTSP/Backup | Snapshot | AI Processing | Motion Detection |

---

## Module Dependencies & Initialization Order

```
1. VIF Device Group
   └── Creates input capture pipeline
   
2. ISP Device & Channel
   ├── Input: VIF output
   ├── Configures image processing
   └── Output: Processed video
   
3. SCL Devices & Channels
   ├── Primary SCL (Dev 0): Multi-resolution scaling
   └── Secondary SCL (Dev 1): Optional AI/VDF scaling
   
4. VENC Devices
   ├── VENC Dev 0: H264/H265 encoders
   └── VENC Dev 8: JPEG encoder
   
5. Binding (MI_SYS_BindChnPort2)
   ├── VIF → ISP
   ├── ISP → SCL
   ├── SCL → VENC (or VENC → VENC for HW Ring)
   └── Optional: VDF/IPU paths
```

---

## Binding Topology Examples

### Simple Single-Stream Setup
```
Sensor → VIF → ISP → SCL(3840x2160) → VENC_H265(Ch0) → RTSP_Stream1
```

### Multi-Resolution Setup (Default)
```
                    ┌→ SCL_Port0(3840x2160) → VENC_H265(Ch0) → RTSP_Main
Sensor → VIF → ISP -┼→ SCL_Port1(1920x1080) → VENC_H264(Ch1) → RTSP_Sub
                    └→ SCL_Port2(640x360)   → VENC_JPEG(Ch2) → Snapshot
```

### Hardware Ring Multi-Stream Setup
```
Sensor → VIF → ISP → SCL → [HW_RING_BUFFER] ← VENC_H265(Ch0) → Stream1
                                              ← VENC_H264(Ch1) → Stream2
```

### With AI/Motion Detection
```
Sensor → VIF → ISP → SCL_Dev0 → VENC_H265(Ch0) → Stream1
                ↓
             SCL_Dev1 → [VDF/IPU] → Analysis
```

---

## Key Configuration Parameters

### Video Parameters
```c
// From g_stStreamAttr[] configuration
.eInputModule    // E_MI_MODULE_ID_SCL or E_MI_MODULE_ID_VENC
.u32InputDev     // Device ID (0, 1, etc.)
.u32InputChn     // Channel ID
.u32InputPort    // Port ID (0-2 for multi-port)
.eType           // E_MI_VENC_MODTYPE_H265E, H264E, JPEGE
.f32Mbps         // Bitrate in Megabits per second
.u32Width        // Output width
.u32Height       // Output height
.eBindType       // HW_RING, FRAME_BASE, REALTIME
```

### ISP Control Parameters
```
Brightness       // Enhancement level
Contrast         // Edge emphasis
Saturation       // Color intensity
Exposure Comp    // EV compensation
Day/Night Mode   // Automatic mode switching
Flicker Mode     // AC 50/60Hz compensation
```

---

## Performance Considerations

### Latency Path Analysis
```
Lowest Latency:  Sensor → VIF → ISP → SCL → VENC (HW_RING) = ~100ms
Medium Latency:  Sensor → VIF → ISP → SCL → VENC (FRAME_BASE) = ~150ms
Snapshot Path:   Sensor → VIF → ISP → SCL → VENC_JPEG = Variable
AI Path:         Sensor → VIF → ISP → SCL_Dev1 → Processing = Custom
```

### Memory Usage
```
Prebuffer Video:  100 frames × 3840×2160 × 1.5 bytes = ~1.2 GB
ISP Pipeline:     Multiple frame buffers = ~200-300 MB
SCL Buffers:      Multiple resolution scales = ~100-150 MB
VENC Ring Buffer: HW managed, ~50 MB per encoder
Total System:     ~2 GB typical
```

### Throughput
```
Main Stream:      2 Mbps (H265 3840x2160 @ 30fps)
Sub Stream:       1 Mbps (H264 1920x1080 @ 30fps)
JPEG Rate:        On-demand (variable)
Total Bandwidth:  ~3.5 Mbps typical
```

---

## Configuration File References

- **Main Config**: `mediaserver.cfg`
- **Audio Config**: `audio_ai_ao_init.json`
- **Stream Preload**: `SigmaStarForQATest.cfg`
- **Sensor Params**: `param_snr0_1536x1536.ini`

---

## Notes

1. **Default Configuration**: Main stream (3840x2160 H265) is enabled by default
2. **Sub-Streams**: Disabled by default but can be enabled via configuration
3. **Prebuffering**: Enabled for video with 100-frame buffer depth
4. **Multi-Sensor**: System supports multiple sensors with separate ISP/SCL pipelines
5. **Resolution Limits**: Maximum supported is typically 4K (3840x2160) depending on hardware
6. **FPS Control**: System maintains synchronized frame rates across pipeline stages

