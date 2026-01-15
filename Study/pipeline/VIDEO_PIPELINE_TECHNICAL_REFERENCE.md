a# Video Pipeline - Technical Implementation Reference

## Module Initialization Sequence

### 1. VIF (Video Input Front) Stage
**File**: `st_main_mediaserver.c:2768`

```c
// Device Group Creation
MI_VIF_CreateDevGroup(nVCapGroup, &stGroupAttr);
// Creates input pipeline from sensor

Parameters:
- nVCapGroup = 0 (Device group ID)
- stGroupAttr.ePixelFormat = E_MI_SYS_PIXEL_FRAME_YUV420
- stGroupAttr.u32DevNum = Number of sensors
```

**Output**: Raw/Processed video frames at sensor resolution (3840x2160 typical)

---

### 2. ISP (Image Signal Processing) Stage
**File**: `st_main_mediaserver.c:2839-2881`

```c
// ISP Device Creation
MI_ISP_CreateDevice(IspDevId, &stIspDevAttr);    // Line 2841
// IspDevId = 0

// ISP Channel Creation
MI_ISP_CreateChannel(IspDevId, IspChnId, &stIspChnAttr);  // Line 2881
// IspChnId = 0

Configuration:
- Input: VIF output
- Output Format: YUV420 (typical)
- Processing: Bayer demosaicing, AWB, AE, NR, LDC (optional)
```

**IQ Control Functions** (`st_isp.c`):
- `ST_Set_DayNight_Mode()` - Day/Night mode switching
- `ST_Set_Flicker_Mode()` - AC flicker compensation (50/60Hz)
- `ST_Set_Brightness()` - Brightness adjustment
- `ST_Set_Contrast()` - Contrast enhancement
- `ST_Set_Saturation()` - Color saturation control
- `ST_Set_ExposureCompensation()` - EV compensation

**Output**: Processed YUV video at full sensor resolution

---

### 3. SCL (Scaler) Stage - Primary
**File**: `st_main_mediaserver.c:2926-2927`

```c
// Primary Scaler Device & Channel Creation
MI_SCL_CreateDevice(SclDevId, &stSclDevAttr);      // Line 2926
// SclDevId = 0

MI_SCL_CreateChannel(SclDevId, SclChnId, &stSclChnAttr);  // Line 2927
// SclChnId = 0

Multi-Output Port Structure:
┌─ Port 0: SCL_Port_FOR_STREAM (Main Stream)
│  └─ 3840x2160 → VENC Channel 0 (H265)
│
├─ Port 1: SCL_Port_FOR_STREAM (Sub Stream)
│  └─ 1920x1080 → VENC Channel 1 (H264) OR VENC Channel 0 (HW Ring)
│
└─ Port 2: SCL_Port_FOR_STREAM (JPEG Snapshot)
   └─ 640x360 → VENC Channel 2 (JPEG)
```

**Configuration**:
```
.u32Width = Full sensor width
.u32Height = Full sensor height
.u32MaxWidth = 3840
.u32MaxHeight = 2160
```

**Output**: Multiple resolution-scaled video streams

---

### 4. SCL (Scaler) Stage - Secondary (Optional)
**File**: `st_main_mediaserver.c:2968-2988`

```c
// Secondary Scaler for AI/VDF processing
if (g_stCameraBootSetting.u8CameraBootSettingenableIPU) {
    MI_SCL_CreateDevice(SclDevId+1, &stSclDevAttr);      // Line 2968
    // SclDevId = 1
    
    MI_SCL_CreateChannel(SclDevId+1, SCL_CHN_FOR_IPU, &stSclChnAttr);     // Ch for IPU
    MI_SCL_CreateChannel(SclDevId+1, SCL_CHN_FOR_VDF, &stSclChnAttr);     // Ch for VDF
    MI_SCL_CreateChannel(SclDevId+1, SCL_CHN_FOR_SUBSTREAM, &stSclChnAttr); // Backup
}

Channels:
- SCL_CHN_FOR_IPU = 1 (Image Processing Unit input)
- SCL_CHN_FOR_VDF = 2 (Video Detection Filter input)
- SCL_CHN_FOR_SUBSTREAM = 0 (Optional sub-stream backup)

Output:
- Low-resolution streams for AI/detection processing
- Typical resolution: 640x360 to 1920x1080
```

---

### 5. VENC (Video Encoder) Stage
**File**: `st_main_mediaserver.c:3058-3059`

#### VENC Device 0: H264/H265 Encoder
```c
// H264/H265 Encoder Device Creation
MI_VENC_CreateDev(0, &pstVencInitParam);     // Line 3058
// Device ID = 0

// Create encoding channels
for each enabled stream {
    MI_VENC_CreateChn(VencDev, VencChn, &stChnAttr);  // Line 1495
}
```

**Channels Configuration** (`g_stStreamAttr[]`):

| Channel | Stream | Input | Codec | Resolution | Bitrate | Mode |
|---------|--------|-------|-------|------------|---------|------|
| 0 | Main | SCL_Port0 | H265 | 3840x2160 | 2 Mbps | HW_RING |
| 1 | Sub | VENC_Ch0/SCL | H264 | 1920x1080 | 1 Mbps | HW_RING/FRAME_BASE |
| 3 | IPU | SCL_Dev1 | - | 640x360 | - | FRAME_BASE |
| 4 | VDF | SCL_Dev1 | - | RAW | - | FRAME_BASE |

#### VENC Device 8: JPEG Encoder
```c
// JPEG Encoder Device Creation
MI_VENC_CreateDev(8, &pstVencInitParam);     // Line 3059
// Device ID = 8

// Create JPEG channel
MI_VENC_CreateChn(8, 2, &stChnAttr);  // Channel 2 for JPEG
```

**JPEG Configuration**:
```
.eType = E_MI_VENC_MODTYPE_JPEGE
.u32Width = 640
.u32Height = 360
Quality = Configurable (typically 75-90)
```

---

## Memory Usage Analysis - By Pipeline Stage

### Overview: Total Memory Consumption
```
Configuration: 4K Main Stream + 1080p Sub Stream + JPEG + Prebuffer
Total Video Pipeline Memory: ~200-250 MB

Breakdown:
├─ VIF Input Buffer:        ~30-40 MB    (10-15%)
├─ ISP Processing:          ~40-50 MB    (18-20%)
├─ SCL Output Buffers:      ~50-60 MB    (22-25%)
├─ VENC Encoder Buffers:    ~40-50 MB    (18-20%)
├─ Prebuffer (100 frames):  ~20-30 MB    (10-12%)
├─ Misc (Config, State):    ~10-15 MB    (5-7%)
└─ Total:                   ~200-250 MB  (100%)
```

---

### Stage 1: VIF (Video Input Front) - Memory Details
**Memory Role**: Raw frame capture from sensor

```c
// Ring buffer structure
typedef struct {
    MI_U32 addr;                    // Physical address
    MI_U32 size;                    // Buffer size
    MI_U32 stride;                  // Line stride
    MI_U32 frame_number;            // Frame index
} VIF_Buffer_t;

// Number of buffers (typically 3-4 for double/triple buffering)
#define VIF_BUFFER_COUNT 4          // Default

// Frame size calculation for raw Bayer data
// Sensor: 4K @ 24fps
// Format: Bayer RGB (10-bit)
pixel_width = 3840;
pixel_height = 2160;
bytes_per_pixel = 1.25;           // 10-bit packed
frame_size = pixel_width * pixel_height * bytes_per_pixel;
                = 3840 * 2160 * 1.25
                = ~10.4 MB per frame

// Total VIF buffer memory
total_vif_memory = frame_size * VIF_BUFFER_COUNT
                = 10.4 MB * 4
                = ~41.6 MB
```

**Detailed Memory Breakdown**:
| Component | Size | Notes |
|-----------|------|-------|
| Frame 0 (Raw Bayer) | 10.4 MB | Main capture buffer |
| Frame 1 (Raw Bayer) | 10.4 MB | Secondary buffer |
| Frame 2 (Raw Bayer) | 10.4 MB | Tertiary buffer |
| Frame 3 (Raw Bayer) | 10.4 MB | Quaternary buffer (spare) |
| VIF Metadata | 100 KB | Frame info, timestamps |
| **VIF Total** | **~41.6 MB** | **Ring buffer mode** |

**Memory Optimization Tips**:
- Use 2-buffer mode if latency allows: ~20.8 MB (saves 20.8 MB)
- Reduce resolution: 1920x1080 Bayer = ~2.6 MB/frame → 10.4 MB total
- Hardware supports on-chip buffering for minimal memory footprint

---

### Stage 2: ISP (Image Signal Processing) - Memory Details
**Memory Role**: Bayer processing, demosaicing, color correction

```c
// ISP processing pipeline buffers
// Input: Raw Bayer (10-bit) = 10.4 MB
// Output: YUV420 = 12.4 MB
// Working memory: Intermediate processing

// ISP working memory structure
typedef struct {
    MI_U32 debayer_buffer;          // Demosaicing working buffer
    MI_U32 color_convert_buffer;    // Color space conversion
    MI_U32 nr_buffer;               // Noise reduction (3DNR)
    MI_U32 ldc_buffer;              // Lens distortion correction
    MI_U32 output_buffer;           // YUV420 output
} ISP_Buffers_t;

// Memory calculation per buffer
// Frame: 3840x2160 YUV420
// Y plane: 3840 * 2160 = 8,294,400 bytes
// U plane: 1920 * 1080 = 2,073,600 bytes
// V plane: 1920 * 1080 = 2,073,600 bytes
// Total per frame: 12.4 MB

isp_output_size = (pixel_width * pixel_height) + 
                  (pixel_width/2 * pixel_height/2) + 
                  (pixel_width/2 * pixel_height/2)
               = 8.29 MB + 2.07 MB + 2.07 MB = 12.43 MB

// ISP typically maintains 2-3 buffers for processing
#define ISP_BUFFER_COUNT 3

// ISP working memory allocation
debayer_temp = pixel_width * pixel_height * 2;     // ~16 MB (2-byte working)
color_lut = 64K * 4;                                // ~256 KB
nr_weight = pixel_width * pixel_height / 4;         // ~2.6 MB
ldc_map = pixel_width * pixel_height / 4;           // ~2.6 MB
```

**Detailed Memory Breakdown**:
| Component | Size | Notes |
|-----------|------|-------|
| ISP Output Buffer 0 | 12.4 MB | Current frame YUV420 |
| ISP Output Buffer 1 | 12.4 MB | Next frame YUV420 |
| ISP Output Buffer 2 | 12.4 MB | Tertiary buffer |
| Debayer Working Space | 16 MB | Temporary processing |
| Color Conversion LUT | 0.25 MB | 64K lookup table |
| 3DNR Weight Maps | 2.6 MB | Noise reduction state |
| LDC Distortion Maps | 2.6 MB | Lens correction tables |
| ISP Metadata/State | 1 MB | IQ parameters, status |
| **ISP Total** | **~60 MB** | **With working memory** |

**Performance Note**:
- ISP processing is typically a bottleneck at 4K
- Memory bandwidth required: 12.4 MB/frame × 30fps = 372 MB/s
- CPU/GPU offloading recommended for real-time processing

---

### Stage 3: SCL (Scaler) - Memory Details
**Memory Role**: Multi-resolution scaling and output distribution

```c
// SCL multi-output configuration
// Input: 3840x2160 YUV420 (12.4 MB)
// Outputs:
//   Port 0: 3840x2160 → VENC Ch0 (H265) = 12.4 MB
//   Port 1: 1920x1080 → VENC Ch1 (H264) = 3.1 MB
//   Port 2: 640x360 → VENC Ch2 (JPEG) = 0.34 MB

// Frame size calculations for each output
scl_output_sizes = {
    [0] = {3840, 2160, 12.4},  // 3840x2160 = 12.4 MB
    [1] = {1920, 1080, 3.1},   // 1920x1080 = 3.1 MB
    [2] = {640, 360, 0.34},    // 640x360 = 0.34 MB
};

// Total concurrent output memory
// SCL maintains buffers for parallel output to multiple encoders
scl_total_frame_buffers = 12.4 MB + 3.1 MB + 0.34 MB = 15.84 MB per iteration

// Ring buffer (2-4 frames per output)
#define SCL_BUFFER_DEPTH 3          // 3 frames in flight

scl_port0_memory = 12.4 MB * 3 = 37.2 MB
scl_port1_memory = 3.1 MB * 3 = 9.3 MB
scl_port2_memory = 0.34 MB * 3 = 1.02 MB
```

**Detailed Memory Breakdown**:
| Component | Size | Notes |
|-----------|------|-------|
| Port 0 Buffers (4K) | 37.2 MB | 3× 12.4 MB frames |
| Port 1 Buffers (1080p) | 9.3 MB | 3× 3.1 MB frames |
| Port 2 Buffers (JPEG res) | 1.02 MB | 3× 0.34 MB frames |
| Scaler Working Memory | 5 MB | Interpolation, filtering |
| Output FIFO Queues | 1 MB | Frame scheduling |
| SCL Metadata/Control | 0.5 MB | Port configs, status |
| **SCL Total** | **~54 MB** | **All 3 ports active** |

**Memory Optimization**:
```c
// If Port 2 (JPEG) disabled: Save 1.02 MB
// If Port 1 (Sub-stream) disabled: Save 9.3 MB
// Minimum SCL (Port 0 only): ~37.7 MB

// HW Ring mode optimization
// Shared buffer between ISP → SCL → VENC (reduced copying)
// Can save ~12 MB by eliminating intermediate copy
```

---

### Stage 4: VENC (Video Encoder) - Memory Details
**Memory Role**: Codec processing and encoded output

```c
// VENC device memory allocation
// Two separate devices:
//   Device 0: H264/H265 encoder
//   Device 8: JPEG encoder

// H264/H265 VENC (Device 0) configuration
#define VENC_H264H265_BUFFER_COUNT 4

// Encoder input buffers (from SCL)
h265_input_4k = 12.4 MB;
h264_input_1080p = 3.1 MB;

// Encoder working memory (per channel)
// H265 4K encoding working space
h265_4k_workspace = {
    .reference_frames = 2 * 12.4,       // 2× ref frames = 24.8 MB
    .reconstructed = 12.4 MB,           // Reconstructed frame
    .motion_vectors = 2.1 MB,           // MV buffer
    .transform_buffer = 8 MB,           // Transform working space
    .entropy_buffer = 2 MB,             // Entropy coder state
};
h265_4k_total_workspace = ~49 MB

// Encoder output buffers
// Bitstream output (variable size, bounded by max bitrate)
max_coded_frame_h265 = (2 Mbps / 30 fps) = 0.067 MB per frame
                     ≈ 67 KB per frame (typical compression)

// Maintain output buffer depth
h265_output_buffer = 67 KB * 4 = 268 KB

// Sub-stream (H264 1080p)
h264_workspace = ~25 MB           // Smaller reference frames
h264_output_buffer = 33 KB * 4 = 132 KB
```

**Detailed Memory Breakdown (Device 0)**:
| Component | Size | Notes |
|-----------|------|-------|
| H265 Reference Frames (2×) | 24.8 MB | For motion prediction |
| H265 Reconstructed Frame | 12.4 MB | After decoding |
| H265 Motion Vectors | 2.1 MB | MB-level MVs |
| H265 Transform Buffers | 8 MB | DCT/IDCT working |
| H265 Entropy Buffers | 2 MB | Arithmetic coding |
| H264 Reference Frames (2×) | 6.2 MB | 1080p × 2 |
| H264 Reconstructed Frame | 3.1 MB | 1080p output |
| H264 Working Space | 15 MB | Smaller than H265 |
| Output Bitstream Buffers | 0.5 MB | Max 400 KB/frame |
| Control Structures | 2 MB | Channel configs |
| **VENC Device 0 Total** | **~78 MB** | **Both H265+H264** |

**Detailed Memory Breakdown (Device 8 - JPEG)**:
| Component | Size | Notes |
|-----------|------|-------|
| JPEG Input Buffer | 0.34 MB | 640×360 YUV420 |
| JPEG Working Memory | 3 MB | Compression buffers |
| JPEG Output Buffer | 50 KB | Typically small |
| **VENC Device 8 Total** | **~3.5 MB** | **JPEG encoder** |

**Total VENC Memory: ~81.5 MB**

---

### Stage 5: Prebuffer System - Memory Details
**Memory Role**: Frame storage for recovery and buffering

```c
// From st_prebuffer.c
#define g_VIDEO_FRAME_NUMBER 100    // 100-frame circular buffer

// Prebuffer data structure
typedef struct {
    MI_U8 *pData;                   // Encoded frame data
    MI_U32 u32DataLen;              // Frame size in bytes
    MI_U64 u64PTS;                  // Presentation timestamp
    MI_BOOL bIsKeyFrame;            // I-frame flag
} H264_frame_t;

// Memory calculation for 100 frames
// Main stream: H265 4K @ 2 Mbps, 30fps
avg_frame_size = (2 Mbps * 1000000 bits/sec) / 
                 (30 fps * 8 bits/byte)
               = 8.33 KB per frame (typical H265 compression)

// Worst case (key frames larger)
max_frame_size = ~200 KB                  // I-frame on scene change

prebuffer_storage = {
    frame_data_array: 200 KB * 100 = 20 MB,
    pts_timestamps: 8 bytes * 100 = 0.8 KB,
    keyframe_flags: 1 byte * 100 = 0.1 KB,
    metadata: ~500 KB,
};

total_prebuffer = ~20.5 MB
```

**Detailed Memory Breakdown**:
| Component | Size | Notes |
|-----------|------|-------|
| Frame Data (100×200KB) | 20 MB | Worst-case frame sizes |
| PTS Timestamps | 0.8 KB | 100× 8-byte timestamp |
| Keyframe Flags | 0.1 KB | 100× 1-byte flags |
| Write/Read Indices | 8 bytes | Ring buffer pointers |
| Access Mutex/Lock | ~100 bytes | Thread synchronization |
| Metadata & Config | 0.5 MB | Prebuffer state |
| **Prebuffer Total** | **~20.5 MB** | **At 100 frames depth** |

**Prebuffer Depth Calculations**:
```c
// Typical frame rates and prebuffer duration
// @ 30fps: 100 frames = 3.33 seconds
// @ 24fps: 100 frames = 4.17 seconds

// Memory vs. buffer time trade-off
buffer_configs = {
    {depth: 50, memory: 10.25 MB, duration_30fps: 1.67s},
    {depth: 100, memory: 20.5 MB, duration_30fps: 3.33s},
    {depth: 150, memory: 30.75 MB, duration_30fps: 5.0s},
};
```

---

### Summary: Complete Memory Map (Typical Configuration)

```
Video Pipeline Memory Distribution (4K + 1080p + JPEG):

┌─────────────────────────────────────────────────┐
│ Total Pipeline Memory: ~220 MB                  │
├─────────────────────────────────────────────────┤
│                                                  │
│ VIF Input Buffers:           42 MB (19%)       │
│  ├─ 4× Bayer frames @ 10.4 MB                  │
│  └─ Metadata & control                         │
│                                                  │
│ ISP Processing:              60 MB (27%)       │
│  ├─ 3× YUV420 output buffers                   │
│  ├─ Demosaicing working space                  │
│  ├─ NR/LDC processing buffers                  │
│  └─ Color correction tables                    │
│                                                  │
│ SCL Scaling & Distribution: 54 MB (25%)       │
│  ├─ 3× 4K buffers (Port 0)                    │
│  ├─ 3× 1080p buffers (Port 1)                 │
│  ├─ 3× snapshot buffers (Port 2)              │
│  └─ Scaling working memory                     │
│                                                  │
│ VENC Encoding:               81 MB (37%)       │
│  ├─ H265 reference frames (24.8 MB)            │
│  ├─ H265 working memory (20 MB)                │
│  ├─ H264 working memory (15 MB)                │
│  ├─ JPEG processing (3 MB)                     │
│  ├─ Bitstream output buffers (0.5 MB)         │
│  └─ Control structures (2 MB)                  │
│                                                  │
│ Prebuffer (100 frames):      20 MB (9%)        │
│  ├─ Circular frame storage                     │
│  ├─ Timestamps & metadata                      │
│  └─ Ring buffer management                     │
│                                                  │
│ Misc (Config, State):         6 MB (3%)        │
│  ├─ Stream configuration arrays                │
│  ├─ Thread stacks                              │
│  ├─ Synchronization primitives                 │
│  └─ Debug/logging buffers                      │
│                                                  │
└─────────────────────────────────────────────────┘
```

**Configuration Variants**:

| Configuration | Memory | Notes |
|---------------|--------|-------|
| **Minimal** (4K only) | ~160 MB | Port 1 & 2 disabled, 50-frame prebuffer |
| **Standard** (4K+1080p) | ~200 MB | Both main & sub, 100-frame prebuffer |
| **Maximum** (4K+2×1080p) | ~260 MB | Dual sub-streams, 150-frame prebuffer |
| **Lite** (1080p only) | ~80 MB | Single 1080p stream, minimal prebuffer |

**Memory Optimization Checklist**:
- ✓ Disable sub-stream if not needed: Save ~9 MB
- ✓ Disable JPEG snapshot: Save ~1 MB
- ✓ Reduce prebuffer from 100→50 frames: Save ~10 MB
- ✓ Use frame-based instead of HW ring: Save ~12 MB (ISP-SCL direct)
- ✓ Reduce ISP working memory with hardware acceleration: Save ~15 MB
- ✓ Single reference frame (H264 baseline): Save ~6 MB

**Total Potential Savings: ~53 MB** (reduction to ~167 MB for minimal setup)

---

## Binding Operations

### Binding Pattern: VIF → ISP
**File**: `st_main_mediaserver.c:2898`

```c
MI_SYS_BindChnPort2(0, &stSrcChnPort, &stDstChnPort,
                    g_sensor_fps_max, g_sensor_fps_max,
                    E_MI_SYS_BIND_TYPE_FRAME_BASE, 0);

Source Port (VIF):
- DevId = 0
- ModuleId = E_MI_MODULE_ID_VIF
- ChnId = 0
- PortId = 0

Destination Port (ISP):
- DevId = 0
- ModuleId = E_MI_MODULE_ID_ISP
- ChnId = 0
- PortId = 0

Frame Rate: Source FPS to Dest FPS ratio
Bind Type: FRAME_BASE or REALTIME
```

### Binding Pattern: ISP → SCL
**File**: `st_main_mediaserver.c:2912`

```c
if (g_stStreamAttr[0].eBindType == E_MI_SYS_BIND_TYPE_HW_RING) {
    // HW Ring binding for low latency
    MI_SYS_BindChnPort2(..., E_MI_SYS_BIND_TYPE_HW_RING, ...);
} else {
    // Frame-based binding
    MI_SYS_BindChnPort2(..., E_MI_SYS_BIND_TYPE_FRAME_BASE, ...);
}

Source Port (ISP):
- ModuleId = E_MI_MODULE_ID_ISP
- ChnId = 0
- PortId = 0

Destination Ports (SCL - multiple):
For each stream {
    ModuleId = E_MI_MODULE_ID_SCL
    ChnId = 0
    PortId = stream.u32InputPort (0, 1, or 2)
}
```

### Binding Pattern: SCL → VENC (Frame-Based)
**File**: `st_main_mediaserver.c:3023`

```c
// Frame-based binding for sub-stream
MI_SYS_BindChnPort2(0, &stSrcChnPort, &stDstChnPort,
                    g_sensor_fps_max, gyuv_eframerate,
                    E_MI_SYS_BIND_TYPE_FRAME_BASE, 0);

Source Port (SCL):
- DevId = 0
- ModuleId = E_MI_MODULE_ID_SCL
- ChnId = 0
- PortId = 1 (for sub-stream)

Destination Port (VENC):
- DevId = 0
- ModuleId = E_MI_MODULE_ID_VENC
- ChnId = 1 (sub-stream encoder)
- PortId = 0
```

### Binding Pattern: SCL → VENC (HW Ring Mode)
**File**: `st_main_mediaserver.c:3038`

```c
// HW Ring binding for main stream (lowest latency)
if (pstStreamAttr[i].eBindType == E_MI_SYS_BIND_TYPE_HW_RING) {
    MI_SYS_BindChnPort2(..., E_MI_SYS_BIND_TYPE_HW_RING, ...);
}

Source Port (SCL):
- ModuleId = E_MI_MODULE_ID_SCL
- ChnId = 0
- PortId = 0 (main stream)

Destination Port (VENC - Main Channel):
- ModuleId = E_MI_MODULE_ID_VENC
- ChnId = 0 (main encoder)
- PortId = 0
```

### Binding Pattern: VENC → VENC (HW Ring - Inter-channel)
**File**: `st_main_mediaserver.c:1531`

```c
// For sub-stream input from main encoder (HW Ring mode)
if (pstStreamAttr[i].eBindType == E_MI_SYS_BIND_TYPE_HW_RING &&
    eInputModule == E_MI_MODULE_ID_VENC) {
    
    // Sub-stream encoder takes input from main encoder
    Source Port (VENC - Main Channel):
    - ModuleId = E_MI_MODULE_ID_VENC
    - ChnId = 0 (main encoder output)
    - PortId = 0
    
    Destination Port (VENC - Sub Channel):
    - ModuleId = E_MI_MODULE_ID_VENC
    - ChnId = 1 (sub-stream encoder)
    - PortId = 0
}
```

---

## Stream Configuration Structure

**Location**: `st_main_mediaserver.c:520-700`

```c
typedef struct {
    MI_BOOL bEnable;              // Enable/disable stream
    MI_MODULE_ID_e eInputModule;  // Input source: SCL or VENC
    MI_U32 u32InputDev;           // Input device ID
    MI_U32 u32InputChn;           // Input channel ID
    MI_U32 u32InputPort;          // Input port ID (0-2)
    MI_U32 u32VencDev;            // Encoder device (0=H264/H265, 8=JPEG)
    MI_VENC_CHN vencChn;          // Encoder channel (0-4)
    MI_VENC_MODTYPE_e eType;      // Codec type
    MI_F32 f32Mbps;               // Bitrate in Mbps
    MI_U32 u32Width;              // Output width
    MI_U32 u32Height;             // Output height
    MI_U32 u32MaxWidth;           // Maximum width
    MI_U32 u32MaxHeight;          // Maximum height
    MI_U32 u32CropX, u32CropY;    // Crop offset
    MI_U32 u32CropWidth;          // Crop width
    MI_U32 u32CropHeight;         // Crop height
    MI_U32 enFunc;                // Function (RTSP, etc.)
    const char *pszStreamName;    // Stream identifier
    MI_SYS_BindType_e eBindType;  // HW_RING, FRAME_BASE, REALTIME
    MI_U32 u32BindPara;           // Binding parameters
} ST_Stream_Attr_T;

// Array definition
static struct ST_Stream_Attr_T g_stStreamAttr[] = {
    // Stream 0: Main stream (enabled)
    {
        .bEnable = TRUE,
        .eInputModule = E_MI_MODULE_ID_SCL,
        .u32InputDev = 0,
        .u32InputChn = 0,
        .u32InputPort = 0,
        .u32VencDev = 0,
        .vencChn = 0,
        .eType = E_MI_VENC_MODTYPE_H265E,
        .f32Mbps = 2,
        .u32Width = 3840,
        .u32Height = 2160,
        .u32MaxWidth = 3840,
        .u32MaxHeight = 2160,
        .enFunc = ST_Sys_Func_RTSP,
        .pszStreamName = MAIN_STREAM,
        .eBindType = E_MI_SYS_BIND_TYPE_HW_RING,
    },
    
    // Stream 1: Sub stream (disabled by default)
    {
        .bEnable = FALSE,
        .eInputModule = E_MI_MODULE_ID_VENC,  // Or SCL for frame-based
        .u32InputDev = 0,
        .u32InputChn = 0,
        .u32InputPort = 0,
        .u32VencDev = 0,
        .vencChn = 1,
        .eType = E_MI_VENC_MODTYPE_H264E,
        .f32Mbps = 1,
        .u32Width = 1920,
        .u32Height = 1080,
        .enFunc = ST_Sys_Func_RTSP,
        .pszStreamName = SUB_STREAM0,
        .eBindType = E_MI_SYS_BIND_TYPE_HW_RING,
    },
    
    // Stream 2: JPEG snapshot
    {
        .bEnable = FALSE,
        .eInputModule = E_MI_MODULE_ID_SCL,
        .u32InputDev = 0,
        .u32InputChn = 0,
        .u32InputPort = 2,
        .u32VencDev = 8,  // JPEG encoder
        .vencChn = 2,
        .eType = E_MI_VENC_MODTYPE_JPEGE,
        .f32Mbps = 0,
        .u32Width = 640,
        .u32Height = 360,
        .enFunc = ST_Sys_Func_BUTT,
        .pszStreamName = SUB_STREAM1,
        .eBindType = E_MI_SYS_BIND_TYPE_FRAME_BASE,
    },
};
```

---

## Prebuffering System

**Location**: `st_prebuffer.c`

### Video Prebuffer
```c
#define g_VIDEO_FRAME_NUMBER 100  // Prebuffer depth

static H264_frame_t prebuffer_vframe[g_VIDEO_FRAME_NUMBER];

// Initialization
memset(prebuffer_vframe, 0, sizeof(H264_frame_t) * g_VIDEO_FRAME_NUMBER);

// Ring buffer management
prebuffer_vframe[prebuffer_write_index] = current_frame;
prebuffer_write_index = (prebuffer_write_index + 1) % g_VIDEO_FRAME_NUMBER;

// Data retrieval
MI_VENC_GetStream(VencDev, VencChn, &stStream, timeout);
```

### NAL Unit Extraction
```c
// From st_prebuffer.c:227
static MI_U32 ST_GetDataDirect(MI_VENC_CHN stVencChn,
                                void **pData,
                                unsigned int *BufLen,
                                int64_t *u64PTS);

Process:
1. Query encoder status: MI_VENC_Query()
2. Get stream with timeout: MI_VENC_GetStream()
3. Extract NAL units from packets
4. Store in circular buffer with timestamp
5. Return to caller for transmission/storage
```

---

## Stream Output Delivery

### RTSP Stream Management
**Location**: `agw_api_interface/agw_mediaserver_ctrl.c`

```c
// Stream registration
void agw_mediaserver_Get_Framerate(handle, stream_id);
void agw_mediaserver_Get_StreamType(handle, stream_id);
void agw_mediaserver_Get_SPS_PPS(handle, stream_id);

// Data delivery
int agw_mediaserver_GetStream(handle, stream_id, buffer, buffer_len);

// File dumping (debug)
static FILE *H264file = NULL;
static FILE *AACfile = NULL;
static FILE *OPUSfile = NULL;

int agw_mediaserver_Set_Dump_Flag(handle, stream_id, flag);
```

---

## Codec Configuration

### H265/H264 VUI Parameters
**Location**: `st_main_mediaserver.c:765-870`

```c
// VUI (Video Usability Information) for H264
MI_VENC_ParamH264Vui_t _param_vui_h264 = {
    .stVuiTimeInfo = {
        .u8TimingInfoPresentFlag = 1,
        .u8FixedFrameRateFlag = 0,
        .u32NumUnitsInTick = 100,
        .u32TimeScale = 4800,
    },
    .stVuiVideoSignal = {
        .u8VideoSignalTypePresentFlag = 0,
        .u8VideoFormat = 5,
        .u8VideoFullRangeFlag = 0,
        .u8ColourDescriptionPresentFlag = 1,
        .u8ColourPrimaries = 1,
        .u8TransferCharacteristics = 1,
        .u8MatrixCoefficients = 1,
    },
};

// VUI for H265
MI_VENC_ParamH265Vui_t _param_vui_h365 = {
    // Similar structure for HEVC
};

// Application to encoder
MI_VENC_SetChnAttr(VencDev, VencChn, &stChnAttr);
```

### Rate Control Modes
```c
// CBR (Constant Bitrate)
.stRcAttr.eRcMode = E_MI_VENC_RC_MODE_H264CBR;
.stRcAttr.stParamCbr.u32BitRate = 2000;  // 2 Mbps
.stRcAttr.stParamCbr.u32FrameRate = 30;

// VBR (Variable Bitrate)
.stRcAttr.eRcMode = E_MI_VENC_RC_MODE_H264VBR;
.stRcAttr.stParamVbr.u32MaxBitRate = 3000;  // 3 Mbps max
.stRcAttr.stParamVbr.u32FrameRate = 30;

// AVBR (Adaptive VBR)
// Available for hardware that supports it
```

---

## Thread Management

### Main Pipeline Threads
```c
// Video capture thread
void *ST_Mi_Ai_ReadLoop_thread(void *args);           // Line 7253

// Encoder output thread
void *ST_get_AvcFrame_and_save(void *args);           // Main stream

// Audio processing threads
void *siren_playback_thread(void *args);              // Line 6440
void *ST_AoOutputProc_thread(void *args);             // Line 6580
void *ST_Mi_Ai_Manager(void *args);                   // Line 7427

// Pool configuration
pthread_t ai_getaudio_threadid;
pthread_t mi_getvideo_threadid;
pthread_t mi_playaudio_threadid;
```

---

## Error Handling & Status

### Module Status Checks
```c
#define STCHECKRESULT(x) \
    if (x != MI_SUCCESS) { \
        printf("[Error] at %s:%d, ret=0x%x\n", __func__, __LINE__, x); \
        return -1; \
    }

// Usage throughout initialization
STCHECKRESULT(MI_VIF_CreateDevGroup(...));
STCHECKRESULT(MI_ISP_CreateDevice(...));
STCHECKRESULT(MI_SCL_CreateDevice(...));
STCHECKRESULT(MI_VENC_CreateDev(...));
STCHECKRESULT(MI_SYS_BindChnPort2(...));
```

---

## Performance Monitoring

### Stream Statistics
```c
// Query encoder status
MI_VENC_ChnStat_t stStat;
MI_VENC_Query(VencDev, VencChn, &stStat);

Information:
- stStat.u32CurPacks: Current packet count
- stStat.u32LeftStreamFrames: Frames waiting for encoding
- stStat.u32LeftStreamBytes: Bytes queued
- stStat.u32GetStreamDoneFrames: Completed frames
- stStat.u32TryGetStreamTimes: Retrieval attempts

// Frame timing
u64PTS: Presentation timestamp
frame_duration: Time between frames
```

---

## Configuration Files

### mediaserver.cfg
```ini
[Video Pipeline]
VideoStreamCount=1-4          # Number of streams
MainStreamResolution=3840x2160
SubStreamResolution=1920x1080
JpegResolution=640x360
Bitrate=2000                  # Main stream bitrate (Kbps)
FrameRate=30

[Encoder]
CodecType=H265               # H264, H265, JPEG
RateControl=CBR              # CBR, VBR, AVBR
Quality=75                   # For JPEG

[ISP]
3DNR=Level1
LDC=Enable
AWB=Auto
AE=Auto
```

### audio_ai_ao_init.json
```json
{
  "audio_config": {
    "sample_rate": 16000,
    "bit_depth": 16,
    "channels": 1,
    "format": "pcm"
  },
  "vqe_config": {
    "aec_enable": true,
    "anr_enable": true,
    "agc_enable": true,
    "hpf_enable": true
  }
}
```

---

## Debug & Monitoring

### File Dumping
```c
// Dump encoder output to file (for debugging)
static FILE *H264file;
static bool Dump_H264_Flag = FALSE;
static char ps8H264DumptFile[] = "/tmp/dump/dumph264.264";

agw_mediaserver_Set_Dump_Flag(handle, STREAM_ID, FLAG_ENABLE);
// Encoder output is now saved to disk for analysis

// Analyze with:
ffprobe /tmp/dump/dumph264.264
ffplay /tmp/dump/dumph264.264
```

### Performance Metrics
```c
// Get framerate
int framerate = ST_Get_Framerate(handle, stream_id);

// Get stream information
MI_VENC_ChnAttr_t stAttr;
MI_VENC_GetChnAttr(VencDev, VencChn, &stAttr);

// Monitor encoder workload
MI_VENC_ChnStat_t stStat;
MI_VENC_Query(VencDev, VencChn, &stStat);
printf("Frames queued: %d\n", stStat.u32LeftStreamFrames);
```

---

