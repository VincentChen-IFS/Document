# Video Pipeline - Quick Reference Guide

## Module Abbreviations & Definitions

| Module | Full Name | Function | Device ID | Typical Channels |
|--------|-----------|----------|-----------|-----------------|
| **VIF** | Video Input Front | Sensor capture interface | 0 | 0-3 |
| **ISP** | Image Signal Processing | Raw processing, filtering | 0 | 0-3 |
| **SCL** | Scaler | Resolution scaling | 0-1 | 0-3 |
| **RGN** | Region Manager | OSD/overlay | 0 | 0-3 |
| **VDF** | Video Dynamic Filter | Motion detection | 0 | 0-1 |
| **IPU** | Image Processing Unit | Custom processing | - | - |
| **VENC** | Video Encoder | H264/H265/JPEG encoding | 0, 8 | 0-4 |
| **RTSP** | Real Time Streaming Protocol | Network delivery | - | - |

---

## Default Stream Configuration

### Enabled by Default
```
Stream 0 (Main Stream):
- Resolution: 3840 × 2160 (4K Ultra HD)
- Codec: H265 (HEVC)
- Bitrate: 2 Mbps
- Input: SCL Port 0
- Binding: HW_RING (lowest latency)
- Output: RTSP Network Stream
```

### Disabled by Default (Enable in config)
```
Stream 1 (Sub Stream):
- Resolution: 1920 × 1080 (Full HD)
- Codec: H264 (AVC)
- Bitrate: 1 Mbps
- Input: VENC Ch0 (HW Ring) or SCL Port 1
- Binding: HW_RING or FRAME_BASE
- Output: RTSP Network Stream

Stream 2 (JPEG Snapshot):
- Resolution: 640 × 360 (VGA)
- Format: JPEG
- Quality: 75%
- Input: SCL Port 2
- Binding: FRAME_BASE
- Output: On-demand snapshot

Streams 3-4 (Optional AI/VDF):
- Resolution: 640 × 360 or custom
- Purpose: AI processing, motion detection
- Input: SCL Device 1
- Output: Analysis data
```

---

## Key Configuration Variables

### System-Level Settings
```c
// From g_stCameraBootSetting
.u8CameraBootSettingSensorNum = 1        // Number of sensors
.u8CameraBootSettingVideoNum = 1         // Number of video streams
.u8CameraBootSettingenableIPU = 0        // Enable IPU (AI)
.u8CameraBootSettingenableVDF = 0        // Enable VDF (motion)
.u8CameraBootSettingenableLDC = 0        // Lens distortion correction
.u32CameraBootSettingPreloadVideoFrame = 100  // Prebuffer depth
```

### Per-Stream Settings
```c
// From g_stStreamAttr[index]
.bEnable                 // Enable/disable stream
.eInputModule            // Source: SCL or VENC
.u32InputDev             // Device ID (0, 1, 8)
.u32InputChn             // Channel ID
.u32InputPort            // Port ID (0-2)
.eType                   // Codec type
.f32Mbps                 // Bitrate in Mbps
.u32Width, .u32Height    // Output resolution
.eBindType               // HW_RING, FRAME_BASE, REALTIME
```

---

## Enabling/Disabling Streams

### Method 1: Configuration File
```bash
# Edit mediaserver.cfg
[Video]
VideoStream0=Enable      # Main stream
VideoStream1=Disable     # Sub stream
VideoStream2=Disable     # JPEG
```

### Method 2: Code Modification
```c
// In st_main_mediaserver.c

// Enable Stream 1 (Sub Stream)
g_stStreamAttr[1].bEnable = TRUE;

// Enable Stream 2 (JPEG)
g_stStreamAttr[2].bEnable = TRUE;

// Change resolution
g_stStreamAttr[0].u32Width = 1920;
g_stStreamAttr[0].u32Height = 1080;

// Rebuild and deploy
```

### Method 3: Runtime API
```c
// Via AGW API interface
agw_mediaserver_Enable_Stream(handle, stream_id, TRUE);
agw_mediaserver_Set_Stream_Resolution(handle, stream_id, 1920, 1080);
agw_mediaserver_Set_Stream_Bitrate(handle, stream_id, 1500);  // 1.5 Mbps
```

---

## Changing Resolutions

### Resolution Presets
```
Primary Stream (4K):
├─ 3840 × 2160 (4K UHD)     [Default]
├─ 2560 × 1440 (2.5K)       [Alternative]
├─ 1920 × 1080 (Full HD)    [Lower power]
└─ 1280 × 720 (HD)          [Minimum]

Secondary Stream (HD):
├─ 1920 × 1080 (Full HD)    [Default]
├─ 1280 × 720 (HD)
└─ 960 × 540 (qHD)

Snapshot (VGA):
├─ 640 × 360 (nHD)          [Default]
├─ 320 × 180 (2qHD)         [Mobile]
└─ 800 × 600 (SVGA)         [High quality]
```

### Performance Impact
```
Resolution    CPU    Memory   Bandwidth   Latency
1280×720      15%    200MB    0.5 Mbps    50ms
1920×1080     25%    400MB    1.0 Mbps    80ms
2560×1440     35%    700MB    1.5 Mbps    100ms
3840×2160     50%    1200MB   2.5 Mbps    120ms
```

---

## Codec Selection Guide

### H265 (HEVC)
```
Advantages:
✓ 30-50% better compression than H264
✓ Lower bitrate for same quality
✓ Better for bandwidth-limited networks
✓ Modern codec (HEVC standard)

Disadvantages:
✗ Higher CPU usage (~20% more)
✗ Not supported on older clients
✗ More complex decoding

Best For:
• Primary/main streams
• 4K resolution
• Bandwidth-constrained networks
• Modern clients (2015+)

Usage:
g_stStreamAttr[0].eType = E_MI_VENC_MODTYPE_H265E;
```

### H264 (AVC)
```
Advantages:
✓ Universal compatibility
✓ Lower CPU usage
✓ Widely supported
✓ Established standard

Disadvantages:
✗ Larger file size than H265
✗ Higher bandwidth requirement
✗ Older standard

Best For:
• Backup/secondary streams
• Mobile client compatibility
• Legacy systems
• Lower CPU systems

Usage:
g_stStreamAttr[1].eType = E_MI_VENC_MODTYPE_H264E;
```

### JPEG
```
Advantages:
✓ Individual frame capture
✓ No streaming overhead
✓ Direct image format
✓ Easy thumbnail generation

Disadvantages:
✗ Much larger than video per frame
✗ No motion compensation
✗ Limited use cases

Best For:
• Snapshots/thumbnails
• Web interface preview
• Motion detection stills
• Archival images

Usage:
g_stStreamAttr[2].eType = E_MI_VENC_MODTYPE_JPEGE;
```

---

## Bitrate Control Modes

### CBR (Constant Bitrate)
```
Mode: E_MI_VENC_RC_MODE_H264CBR / E_MI_VENC_RC_MODE_H265CBR

Behavior:
• Fixed bitrate regardless of content
• Consistent file size per duration
• Predictable network bandwidth

Configuration:
.stRcAttr.stParamCbr.u32BitRate = 2000;  // 2 Mbps
.stRcAttr.stParamCbr.u32FrameRate = 30;

Use Cases:
✓ Network streaming with fixed bandwidth
✓ Predictable storage requirements
✓ Live broadcasting

Quality:
◆ May fluctuate with scene complexity
```

### VBR (Variable Bitrate)
```
Mode: E_MI_VENC_RC_MODE_H264VBR / E_MI_VENC_RC_MODE_H265VBR

Behavior:
• Bitrate varies with content complexity
• Better quality for variable scenes
• Higher average bitrate than CBR

Configuration:
.stRcAttr.stParamVbr.u32MaxBitRate = 3000;  // 3 Mbps max
.stRcAttr.stParamVbr.u32FrameRate = 30;

Use Cases:
✓ Recording to storage (file size less important)
✓ Variable quality preference
✓ Higher quality requirements

Quality:
◆ Better detail in high-motion scenes
```

### AVBR (Adaptive VBR)
```
Mode: E_MI_VENC_RC_MODE_AVBR (if supported)

Behavior:
• Automatically adapts to network conditions
• Balances quality and bandwidth

Configuration:
.stRcAttr.eRcMode = E_MI_VENC_RC_MODE_AVBR;
.stRcAttr.u32TargetBitRate = 2000;

Use Cases:
✓ Network-dependent streaming
✓ Mobile/cellular networks
✓ Adaptive streaming

Quality:
◆ Automatically optimized
```

---

## Frame Rate Control

### Standard Frame Rates
```
30 fps    Most common, smooth motion
25 fps    PAL standard regions
24 fps    Film-like appearance
15 fps    Low bandwidth, minimal motion
10 fps    Security/monitoring
5 fps     Very low bandwidth
```

### Implementation
```c
// Bind frame rates
MI_SYS_BindChnPort2(0,
                    &stSrcChnPort,
                    &stDstChnPort,
                    src_fps,        // VIF output FPS (30)
                    dst_fps,        // SCL/VENC output FPS
                    E_MI_SYS_BIND_TYPE_HW_RING,
                    0);

// Example: 30fps input, 15fps sub-stream output
MI_SYS_BindChnPort2(0, &srcChn, &dstChn, 30, 15, ...);
// Encoder will output every 2nd frame
```

---

## ISP (Image Signal Processing) Controls

### Exposure Control
```c
// Auto Exposure with manual adjustment
MI_ISP_AE_EvCompType_t evComp;
MI_ISP_AE_GetEvComp(0, 0, &evComp);
evComp.value += 10;  // Increase by 1.0 EV
MI_ISP_AE_SetEvComp(0, 0, &evComp);

// Exposure point for metering
MI_ISP_AE_ExpoPointParam_t point = {16, 300, 1024, 1024};
MI_ISP_AE_SetPlainLongExpoTable(0, 0, &point);
```

### Color & Contrast
```c
// Brightness
MI_ISP_IQ_BrightnessType_t brightness;
MI_ISP_IQ_GetBrightness(0, 0, &brightness);
brightness.value = 128 + adjustment;  // 0-256 range
MI_ISP_IQ_SetBrightness(0, 0, &brightness);

// Contrast
MI_ISP_IQ_ContrastType_t contrast;
MI_ISP_IQ_GetContrast(0, 0, &contrast);
contrast.value = 128 + adjustment;
MI_ISP_IQ_SetContrast(0, 0, &contrast);

// Saturation
MI_ISP_IQ_SaturationType_t saturation;
MI_ISP_IQ_GetSaturation(0, 0, &saturation);
saturation.value = 128 + adjustment;
MI_ISP_IQ_SetSaturation(0, 0, &saturation);
```

### Day/Night Mode
```c
MI_ISP_AE_ExpoPointParam_t day_params = {{16, 300, 1024, 1024}, ...};
MI_ISP_AE_ExpoPointParam_t night_params = {{16, 30, 1024, 1024}, ...};

// Switch to day mode
MI_ISP_AE_SetPlainLongExpoTable(0, 0, &day_params);

// Switch to night mode (higher gain)
MI_ISP_AE_SetPlainLongExpoTable(0, 0, &night_params);
```

### Flicker Removal
```c
// AC Flicker compensation
MI_ISP_AE_FlickerType_e flicker_type = MI_ISP_AE_FLICKER_50HZ;  // Europe
// OR
// flicker_type = MI_ISP_AE_FLICKER_60HZ;  // USA
MI_ISP_AE_SetFlicker(0, 0, &flicker_type);
```

---

## Prebuffering & NAL Unit Management

### Configuration
```c
#define VIDEO_FRAME_BUFFER_DEPTH 100

// Prebuffer structure
typedef struct {
    uint8_t *nal_unit;        // NAL unit data
    uint32_t length;          // NAL unit size
    int64_t dts;             // Decoding timestamp
    int64_t pts;             // Presentation timestamp
    uint8_t nal_unit_type;   // IDR, P, B frame
} H264Frame;

H264Frame prebuffer[VIDEO_FRAME_BUFFER_DEPTH];
uint32_t write_index = 0;
uint32_t read_index = 0;
```

### Ring Buffer Operation
```c
// Write: Store encoded frame
prebuffer[write_index] = current_frame;
write_index = (write_index + 1) % VIDEO_FRAME_BUFFER_DEPTH;

// Read: Retrieve frame for streaming
if (read_index != write_index) {
    frame = prebuffer[read_index];
    read_index = (read_index + 1) % VIDEO_FRAME_BUFFER_DEPTH;
}

// Check fullness
frames_buffered = (write_index - read_index + DEPTH) % DEPTH;
```

### NAL Unit Types
```
H264/H265 NAL Unit Classification:

Type  Name           Significance
────  ──────────────  ─────────────────────────
0     Unspecified    (Skip)
1     Non-IDR Slice  P-frame (predicted)
2     A slice part   (Fragment of slice)
3     A slice part   (Fragment of slice)
4     NALU Type 4    (SEI - Supplemental Enhancement Info)
5     IDR Slice      I-frame (key frame) ⭐
6     Filler data    (Padding)
7     SPS            Sequence Parameter Set (codec init) ⭐
8     PPS            Picture Parameter Set (codec init) ⭐
...

⭐ = Required for stream start/seeking
     Must always include in prebuffer head
```

---

## RTSP Integration Points

### Stream Registration
```c
// Define stream endpoint
#define MAIN_STREAM "MainStream"
#define SUB_STREAM0 "SubStream"
#define SUB_STREAM1 "Snapshot"

// Register stream
// RTSP client requests: rtsp://camera_ip:554/MainStream
```

### SDP (Session Description Protocol) Generation
```
v=0
o=camera 1234567890 2 IN IP4 192.168.1.100
s=Innofusion Camera Stream
c=IN IP4 192.168.1.100
t=0 0
a=tool:InnofusionMediaServer
a=type:broadcast
m=video 0 RTP/AVP 97
a=rtpmap:97 H265/90000
a=fmtp:97 sprop-sps=...;sprop-pps=...
a=control:trackID=0

[For H264]
m=video 0 RTP/AVP 96
a=rtpmap:96 H264/90000
a=fmtp:96 sprop-parameter-sets=...
```

---

## Performance Tuning Checklist

```
✓ Resolution optimization
  └─ Match to actual requirement
  └─ 1080p usually sufficient for remote viewing

✓ Bitrate tuning
  └─ Start at 2Mbps for 4K, 1Mbps for 1080p
  └─ Adjust based on quality feedback
  └─ Monitor actual bandwidth usage

✓ Codec selection
  └─ H265 for newer systems
  └─ H264 for compatibility
  └─ Consider CPU vs quality tradeoff

✓ Frame rate
  └─ 30fps for general use
  └─ 15fps to save bandwidth
  └─ 5fps for monitoring

✓ Prebuffering
  └─ 100 frames typical
  └─ Increase for network resilience
  └─ Decrease for latency sensitivity

✓ ISP settings
  └─ 3DNR: Moderate (quality vs smoothing)
  └─ AE: Auto with compensation
  └─ AWB: Auto preferred
  └─ LDC: Enable if fisheye distortion

✓ CPU monitoring
  └─ Target <70% sustained load
  └─ Monitor with: top, htop, /proc/cpuinfo
  └─ Profile with: perf, valgrind

✓ Memory usage
  └─ Target <500MB for video pipeline
  └─ Monitor with: free, /proc/meminfo
  └─ Check for memory leaks: valgrind --leak-check=full
```

---

## Troubleshooting Quick Reference

| Symptom | Possible Causes | Solutions |
|---------|-----------------|-----------|
| **High latency** | HW_RING disabled, Prebuffer too deep | Enable HW_RING, Reduce prebuffer to 50 |
| **Choppy video** | Frame drops, CPU overload | Reduce resolution, Lower bitrate, Optimize ISP |
| **Quality loss** | Bitrate too low, VBR not optimized | Increase bitrate, Enable VBR, Check codec |
| **Stream drops** | Memory leak, Buffer overflow | Monitor memory, Reduce frame depth, Restart |
| **No keyframes** | IDR not configured | Set IDR interval, Verify encoder params |
| **Sync issues** | DTS/PTS mismatch | Verify timestamp generation, Check bind rates |
| **Color issues** | ISP color matrix wrong | Reset to defaults, Adjust AWB settings |
| **Motion blur** | 3DNR too aggressive | Reduce 3DNR level, Increase shutter speed |

---

