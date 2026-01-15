# Video Pipeline Architecture - Documentation Index

## Overview

This documentation provides comprehensive analysis of the Innofusion Media Interface video processing pipeline, including block diagrams, technical implementation details, and configuration guides.

---

## Documentation Files

### 1. **VIDEO_PIPELINE_ARCHITECTURE.md** 📊
**High-level system architecture and overview**

Contains:
- Complete video pipeline block diagram
- Detailed pipeline architecture explanation
- Module descriptions (VIF, ISP, SCL, VENC, etc.)
- Data flow modes (Frame-based, Hardware Ring, Real-time)
- Stream configuration table
- Module dependencies and initialization order
- Binding topology examples
- Performance considerations

**Best for**: Understanding overall system flow, architecture decisions, performance tradeoffs

---

### 2. **VIDEO_PIPELINE_TECHNICAL_REFERENCE.md** 🔧
**Detailed implementation and code reference**

Contains:
- Module initialization sequence with line numbers
- VIF, ISP, SCL, VENC creation code
- Binding operations and patterns
- Stream configuration structure definition
- Prebuffering system implementation
- Stream output delivery mechanism
- Codec configuration (H265, H264, JPEG)
- Thread management
- Error handling patterns
- Performance monitoring
- Configuration file reference

**Best for**: Developers implementing or modifying pipeline stages, debugging, code-level understanding

---

### 3. **VIDEO_PIPELINE_DIAGRAMS.md** 📈
**Visual representations and architecture diagrams**

Contains:
- Complete video processing pipeline flow diagram
- Detailed codec and bitrate flow diagram
- Binding topology comparisons (4 configurations)
- Frame timing and synchronization diagram
- Memory layout and buffer organization
- State machine for stream initialization
- CPU load vs resolution graph
- Network bandwidth profile

**Best for**: Visual learners, presentation materials, system design reviews

---

### 4. **VIDEO_PIPELINE_QUICK_REFERENCE.md** ⚡
**Quick lookup guide and configuration reference**

Contains:
- Module abbreviations and definitions
- Default stream configuration
- Key configuration variables
- Enabling/disabling streams (3 methods)
- Changing resolutions and performance impact
- Codec selection guide (H265, H264, JPEG)
- Bitrate control modes (CBR, VBR, AVBR)
- Frame rate control
- ISP controls and settings
- Prebuffering operations
- RTSP integration
- Performance tuning checklist
- Troubleshooting table

**Best for**: Quick answers, configuration changes, troubleshooting, performance tuning

---

## Quick Navigation

### By Role

**System Architects / Managers**
- Read: Architecture overview
- Focus: Block diagrams, system topology
- Files: `VIDEO_PIPELINE_ARCHITECTURE.md`, `VIDEO_PIPELINE_DIAGRAMS.md`

**Software Developers**
- Read: Technical reference + quick reference
- Focus: Code patterns, API usage, configuration
- Files: `VIDEO_PIPELINE_TECHNICAL_REFERENCE.md`, `VIDEO_PIPELINE_QUICK_REFERENCE.md`

**System Integrators**
- Read: Architecture + quick reference
- Focus: Configuration, performance tuning
- Files: `VIDEO_PIPELINE_ARCHITECTURE.md`, `VIDEO_PIPELINE_QUICK_REFERENCE.md`

**QA/Test Engineers**
- Read: Troubleshooting + diagrams
- Focus: Performance metrics, test scenarios
- Files: `VIDEO_PIPELINE_DIAGRAMS.md`, `VIDEO_PIPELINE_QUICK_REFERENCE.md`

---

## Key Concepts at a Glance

### The Pipeline in 30 Seconds
```
Sensor → VIF → ISP → SCL → VENC → RTSP Network
         Capture  Process  Scale  Encode  Stream
```

### Main Stream (Enabled by Default)
```
Resolution:  3840 × 2160 (4K)
Codec:       H265 (HEVC)
Bitrate:     2 Mbps
Latency:     ~20-30ms (HW Ring mode)
Output:      RTSP network stream
```

### Sub Streams (Optional)
```
Stream 1: 1920×1080 H264 @ 1Mbps (Backup/Preview)
Stream 2: 640×360 JPEG (Snapshots/Thumbnails)
Stream 3: 640×360 YUV (AI Processing - optional)
Stream 4: RAW Size (Motion Detection - optional)
```

### Key Performance Metrics
```
Memory:       ~200-250 MB (video pipeline)
CPU:          ~50% for 4K H265 encoding
Throughput:   3.5 Mbps (main + sub combined)
Latency:      20-30ms (sensor to output)
Prebuffer:    100 frames (~3.3 seconds)
```

---

## Architecture Highlights

### Multi-Stream Capability
- **4 simultaneous streams** (H265, H264, JPEG, Optional)
- **Flexible resolution** per stream
- **Independent bitrate control**
- **Synchronized timing** across streams

### Dual-Mode Processing
- **HW Ring Mode**: Lowest latency (~20ms), multi-stream from single encoder
- **Frame-Based Mode**: Maximum compatibility, independent buffers

### ISP Integration
- **Auto Exposure** (AE) with manual override
- **Auto White Balance** (AWB)
- **3D Noise Reduction** (3DNR)
- **Lens Distortion Correction** (LDC)
- **Color & Contrast** controls

### Intelligent Scaling
- **Primary Scaler**: Multi-port resolution scaling (3840→3 outputs)
- **Secondary Scaler**: Optional AI/VDF processing at reduced resolution

### Flexible Binding
- **Module interconnection** via hardware or software
- **Programmable frame rates** between stages
- **Error resilience** with configurable drop policies

---

## Configuration Quick Start

### Enable Multiple Streams
```c
// Main stream (already enabled)
g_stStreamAttr[0].bEnable = TRUE;

// Enable sub-stream
g_stStreamAttr[1].bEnable = TRUE;

// Enable JPEG snapshots
g_stStreamAttr[2].bEnable = TRUE;

// Rebuild project
make clean; make
```

### Change Main Stream Codec (H265 → H264)
```c
// In st_main_mediaserver.c, line ~530
g_stStreamAttr[0].eType = E_MI_VENC_MODTYPE_H264E;  // Change H265E to H264E
```

### Adjust Bitrate & Resolution
```c
// Modify stream configuration
g_stStreamAttr[0].u32Width = 1920;      // 4K → 1080p
g_stStreamAttr[0].u32Height = 1080;
g_stStreamAttr[0].f32Mbps = 1.0;        // 2 Mbps → 1 Mbps
```

### Enable AI/Motion Processing
```c
// Uncomment in code or config
#define USE_IPU 1      // Image Processing Unit
#define USE_VDF 1      // Video Dynamic Filter

// Set boot setting
g_stCameraBootSetting.u8CameraBootSettingenableIPU = 1;
g_stCameraBootSetting.u8CameraBootSettingenableVDF = 1;
```

---

## Performance Tuning Guide

### For Lowest Latency
```
✓ Use HW_RING binding mode
✓ Reduce prebuffer depth (50 frames)
✓ Single stream only
✓ Lower resolution (1080p)
✓ Fixed frame rate (30fps)
Result: ~20-30ms end-to-end latency
```

### For Best Quality
```
✓ Use H265 codec
✓ Increase bitrate (3-5 Mbps)
✓ Enable VBR rate control
✓ 4K resolution (3840×2160)
✓ 30fps frame rate
✓ Enable 3DNR
Result: Best subjective quality
```

### For Maximum Compatibility
```
✓ Use H264 codec
✓ Multiple bitrate streams
✓ 1080p primary stream
✓ CBR rate control
✓ Include SPS/PPS in stream
Result: Works with all RTSP clients
```

### For Bandwidth-Constrained Networks
```
✓ Use H265 codec
✓ Reduce resolution (720p)
✓ Adaptive bitrate (AVBR)
✓ Lower frame rate (15fps)
✓ Multiple streams
Result: Minimal bandwidth usage
```

---

## Common Tasks & Solutions

### Enable Second Stream for Multi-Client Support
**Reference**: `VIDEO_PIPELINE_QUICK_REFERENCE.md` → "Enabling/Disabling Streams"
```
1. Set g_stStreamAttr[1].bEnable = TRUE
2. Configure input: SCL Port 1 or VENC Ch0 (HW Ring)
3. Set resolution: 1920×1080
4. Set codec: H264
5. Rebuild and test
```

### Change Encoding Bitrate
**Reference**: `VIDEO_PIPELINE_QUICK_REFERENCE.md` → "Bitrate Control Modes"
```
1. Modify g_stStreamAttr[x].f32Mbps
2. Set rate control mode (CBR/VBR)
3. Configure max/target bitrate
4. Monitor actual bandwidth with: iftop, nethogs
5. Adjust based on network feedback
```

### Adjust Image Quality (Brightness, Contrast, Color)
**Reference**: `VIDEO_PIPELINE_QUICK_REFERENCE.md` → "ISP Controls"
```
1. Call MI_ISP_IQ_GetBrightness() / GetContrast() / GetSaturation()
2. Adjust value property
3. Call MI_ISP_IQ_SetBrightness() / SetContrast() / SetSaturation()
4. Monitor effect in real-time
```

### Reduce Latency from 150ms to 50ms
**Reference**: `VIDEO_PIPELINE_TECHNICAL_REFERENCE.md` → "Binding Operations"
```
1. Verify HW_RING binding enabled (check eBindType)
2. Reduce prebuffer depth: 50 frames instead of 100
3. Disable non-critical streams
4. Use single resolution (no scaling)
5. Monitor with framerate tools
```

### Integrate AI/Motion Detection Module
**Reference**: `VIDEO_PIPELINE_ARCHITECTURE.md` → "Optional Processing Modules"
```
1. Enable Secondary SCL (Device 1)
2. Create IPU/VDF channels
3. Bind SCL_Dev0 → SCL_Dev1 (reduced resolution)
4. Implement detection algorithm
5. Output metadata via RTSP extensions or TCP
```

---

## Recommended Reading Order

### First Time Setup
1. Read: `VIDEO_PIPELINE_ARCHITECTURE.md` (Overview section)
2. Skim: `VIDEO_PIPELINE_DIAGRAMS.md` (Visual understanding)
3. Reference: `VIDEO_PIPELINE_QUICK_REFERENCE.md` (Configuration)

### Implementation / Integration
1. Study: `VIDEO_PIPELINE_TECHNICAL_REFERENCE.md` (Code patterns)
2. Reference: `VIDEO_PIPELINE_ARCHITECTURE.md` (Design decisions)
3. Debug: `VIDEO_PIPELINE_QUICK_REFERENCE.md` (Troubleshooting)

### Performance Optimization
1. Study: `VIDEO_PIPELINE_DIAGRAMS.md` (Performance profiles)
2. Reference: `VIDEO_PIPELINE_QUICK_REFERENCE.md` (Tuning checklist)
3. Deep dive: `VIDEO_PIPELINE_TECHNICAL_REFERENCE.md` (Rate control details)

### Troubleshooting
1. Check: `VIDEO_PIPELINE_QUICK_REFERENCE.md` (Troubleshooting table)
2. Verify: `VIDEO_PIPELINE_TECHNICAL_REFERENCE.md` (Error handling)
3. Analyze: `VIDEO_PIPELINE_DIAGRAMS.md` (State machines)

---

## File Locations

All documentation is stored in:
```
d:\GIT\innofusion-media-interface\mediaserver\
├── VIDEO_PIPELINE_ARCHITECTURE.md
├── VIDEO_PIPELINE_TECHNICAL_REFERENCE.md
├── VIDEO_PIPELINE_DIAGRAMS.md
├── VIDEO_PIPELINE_QUICK_REFERENCE.md
└── VIDEO_PIPELINE_DOCUMENTATION_INDEX.md (this file)
```

**Source code references**:
- Main pipeline: `st_main_mediaserver.c` (12,196 lines)
- ISP control: `st_isp.c`
- Audio processing: `st_audio.c`
- Prebuffer: `st_prebuffer.c`
- Encoding: included `st_venc.h`
- API interface: `agw_api_interface/agw_mediaserver_ctrl.c`

---

## Key Statistics

| Metric | Value |
|--------|-------|
| **Pipeline Stages** | 6 (VIF→ISP→SCL→VENC→Output) |
| **Max Streams** | 4 simultaneous |
| **Max Resolution** | 3840×2160 (4K) |
| **Max Bitrate** | 3-5 Mbps (combined) |
| **Min Latency** | ~20-30ms (HW Ring) |
| **Prebuffer Depth** | 100 frames (~3.3s) |
| **Memory Usage** | ~200-250 MB |
| **CPU Usage** | 25-50% (1×4K stream) |
| **ISP Filters** | 8+ (AE, AWB, 3DNR, LDC, etc.) |
| **Codec Support** | H265, H264, JPEG |
| **Rate Control** | CBR, VBR, AVBR |
| **Supported Platforms** | Harpy, Lory (Innofusion SoCs) |

---

## Contact & Support

For questions or clarifications about the pipeline architecture:

1. **Review documentation** in this folder
2. **Check code comments** in `st_main_mediaserver.c`
3. **Consult example configs** in `mediaserver/configs/`
4. **Review README files** in subdirectories

---

**Last Updated**: January 2026
**Version**: 1.0
**Status**: Production

