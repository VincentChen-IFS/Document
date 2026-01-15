0.0.2.7# st_main_mediaserver.c - Video and Audio Functions

This document provides a comprehensive list of video and audio related functions in `st_main_mediaserver.c`.

---

## **VIDEO FUNCTIONS**

### Video Pipeline Management
Functions for managing the video processing pipeline (VIF → ISP → SCL → VENC).

| Function | Description |
|----------|-------------|
| `ST_StartPipeLine()` | Start video pipeline with specified dimensions and cropping |
| `ST_StopPipeLine()` | Stop video pipeline |
| `ST_PiPeReinit()` | Reinitialize video pipeline |
| `ST_BaseModuleInit()` | Initialize base video modules (VIF/ISP/SCL/VENC) |
| `ST_BaseModuleUnInit()` | Uninitialize base video modules |

### Video Encoding
Functions for managing video encoding operations.

| Function | Description |
|----------|-------------|
| `ST_VencStart()` | Start video encoder |
| `ST_VencStop()` | Stop video encoder |
| `ST_VideoReadStream()` | Read encoded video stream data |
| `ST_CloseStream()` | Close video stream |
| `ST_SaveStreamToFile()` | Save video stream to file |

### Video Capture & JPEG
Functions for JPEG snapshot capture.

| Function | Description |
|----------|-------------|
| `ST_CaptureJPGProc()` | JPEG capture processing |
| `ST_DoCaptureJPGProc()` | Execute JPEG capture with rotation |

### Video Frame Rate Control
Functions for managing video frame rate and adaptive streaming.

| Function | Description |
|----------|-------------|
| `ST_VideoDropPreload()` | Drop video preload frames |
| `ST_Set_RateAdtFramerate()` | Set adaptive frame rate |
| `ST_Get_Framerate()` | Get current frame rate |
| `ST_Send_Framerate_Result()` | Send frame rate result |
| `ST_Mapping_Drop_Framerate_Group()` | Map frame rate drop group |

### Video OSD/RGN (On-Screen Display/Region)
Functions for managing video overlays and regions.

| Function | Description |
|----------|-------------|
| `ST_OsdStart()` | Start OSD overlay |
| `ST_OsdStop()` | Stop OSD overlay |
| `ST_UpdateRgnOsdProc()` | Update OSD region |

### Video VDF (Video Defect Detection/Motion Detection)
Functions for video analytics and motion detection.

| Function | Description |
|----------|-------------|
| `ST_VdfStart()` | Start VDF module |
| `ST_VdfSop()` | Stop VDF module |
| `ST_ModuleInit_VDF()` | Initialize VDF module |
| `ST_ModuleunInit_VDF()` | Uninitialize VDF module |
| `ST_ModuleInit_VDF_MDOD_Rect()` | Initialize MD/OD rectangle detection |
| `ST_VDFMDSadMdNumCal()` | Calculate MD SAD (Sum of Absolute Differences) number |
| `ST_VDFMDToRectBase()` | Convert MD to rectangle base coordinates |
| `ST_VDFMDtoRECT_SAD()` | Convert MD to rectangle SAD values |

---

## **AUDIO FUNCTIONS**

### Audio Input (AI - Audio In)
Functions for managing audio input capture.

| Function | Description |
|----------|-------------|
| `ST_Ai_Init()` | Initialize audio input device |
| `ST_Ai_Uninit()` | Uninitialize audio input device |
| `ST_mi_ai_GetChnPortBuf()` | Get AI channel port buffer data |
| `ST_Ai_getpreload_pcmframe()` | Get preload PCM frames |
| `ST_Send_Ai_Readpktmsg()` | Send AI read packet message |

### Audio Output (AO - Audio Out)
Functions for managing audio output playback.

| Function | Description |
|----------|-------------|
| `ST_AOInit()` | Initialize audio output device |
| `ST_AOExit()` | Exit audio output |
| `ST_AOExitMSg()` | Exit audio output with message queue cleanup |
| `ST_AoInProc()` | Audio output input processing |
| `ST_AoInProc_traller()` | Audio output trailer processing |

### Audio Processing Chain (AI Path)
Master control functions for AI audio processing.

| Function | Description |
|----------|-------------|
| `ST_AI_APC_init()` | Initialize AI audio processing chain |
| `ST_AI_APC_uninit()` | Uninitialize AI audio processing chain |

### AI - High Pass Filter (HPF)
Functions for AI path high-pass filtering.

| Function | Description |
|----------|-------------|
| `ST_AI_APC_HPF_init()` | Initialize AI HPF (removes low frequencies) |
| `ST_AI_APC_HPF_Run()` | Run AI HPF processing on audio buffer |
| `ST_AI_APC_HPF_uninit()` | Uninitialize AI HPF |

### AI - Acoustic Echo Cancellation (AEC)
Functions for echo cancellation (removes speaker output from microphone input).

| Function | Description |
|----------|-------------|
| `ST_AI_AEC_init()` | Initialize AI AEC |
| `ST_AI_AEC_Run()` | Run AI AEC processing (near-end + far-end) |
| `ST_AI_AEC_uninit()` | Uninitialize AI AEC |

### AI - Automatic Noise Reduction (ANR)
Functions for noise reduction/suppression.

| Function | Description |
|----------|-------------|
| `ST_AI_APC_ANR_init()` | Initialize AI ANR with smoothing level |
| `ST_AI_APC_ANR_Run()` | Run AI ANR processing on audio buffer |
| `ST_AI_APC_ANR_uninit()` | Uninitialize AI ANR |

### AI - Automatic Gain Control (AGC)
Functions for automatic gain control on input audio.

| Function | Description |
|----------|-------------|
| `ST_AI_APC_AGC_init()` | Initialize AI AGC |
| `ST_AI_APC_AGC_Run()` | Run AI AGC processing on audio buffer |
| `ST_AI_APC_AGC_uninit()` | Uninitialize AI AGC |

### Audio Processing Chain (AO Path)
Master control functions for AO audio processing.

| Function | Description |
|----------|-------------|
| `ST_AO_APC_init()` | Initialize AO audio processing chain |
| `ST_AO_APC_uninit()` | Uninitialize AO audio processing chain |

### AO - High Pass Filter (HPF)
Functions for AO path high-pass filtering.

| Function | Description |
|----------|-------------|
| `ST_AO_APC_HPF_init()` | Initialize AO HPF |
| `ST_AO_APC_HPF_Run()` | Run AO HPF processing on output audio |
| `ST_AO_APC_HPF_uninit()` | Uninitialize AO HPF |

### AO - Automatic Gain Control (AGC/DRC)
Functions for dynamic range compression on output audio.

| Function | Description |
|----------|-------------|
| `ST_AO_APC_AGC_init()` | Initialize AO AGC/DRC |
| `ST_AO_APC_AGC_Run()` | Run AO AGC/DRC processing |
| `ST_AO_APC_AGC_uninit()` | Uninitialize AO AGC/DRC |

### Audio Preload & Buffering
Functions for audio buffer management and shared memory operations.

| Function | Description |
|----------|-------------|
| `ST_AudioDropPreload()` | Drop audio preload frames |
| `ST_dump_audio_to_sharemem()` | Dump audio data to shared memory |
| `ST_dump_pcm_to_prequeue()` | Dump PCM data to pre-queue buffer |
| `ST_shmq_windex_update()` | Update shared memory queue write index |

---

## **UTILITY & SUPPORT FUNCTIONS**

### Configuration & Parsing
Functions for loading and parsing configuration files.

| Function | Description |
|----------|-------------|
| `Parser_APC_AEC_config()` | Parse audio APC/AEC config file (HPF/ANR/AEC/AGC parameters) |
| `Parser_mediaserver_config()` | Parse mediaserver config file |
| `parsemodparamJsonFile()` | Parse JSON module parameters |
| `ST_DefaultConfig()` | Set default configuration values |
| `ST_DefaultArgs()` | Set default argument values |
| `ST_ResetArgs()` | Reset arguments to defaults |

### Codec Support
Functions for audio codec operations.

| Function | Description |
|----------|-------------|
| `ST_mediasrvr_parser_ogg_header2()` | Parse OGG file header |
| `ST_mediasrvr_read_ogg()` | Read OGG packet data |
| `Siren_packet_Write()` | Write siren audio packet data |

### System & IPC
Functions for inter-process communication and system management.

| Function | Description |
|----------|-------------|
| `Creat_mi_ipcmsgq()` | Create IPC message queue |
| `MI_Share_Mem_uninit()` | Uninitialize shared memory |
| `ST_HandleSig()` | Handle system signals (SIGINT, SIGTERM, etc.) |
| `ST_DoExitProc()` | Perform exit cleanup processing |
| `ST_RegisterInputCmd()` | Register input command handler |

### Miscellaneous
General utility functions.

| Function | Description |
|----------|-------------|
| `ST_Flush()` | Flush buffers |
| `av_usleep()` | Microsecond precision sleep |
| `ST_Do_MICLISendDataToRtos()` | Send data to RTOS via MICLI |
| `ST_LoadPreloadSettingTXT()` | Load preload settings from text file |
| `ST_GetPreloadSettingTXTVol()` | Get preload setting volume value |
| `ST_ReplacePreloadSettingTXTVol()` | Replace preload setting volume value |
| `ST_GetRTOSCameraBootSetting()` | Get RTOS camera boot settings |

### Main Entry Point

| Function | Description |
|----------|-------------|
| `main()` | Application entry point - initializes modules and starts processing |

---

## **Audio Processing Pipeline (AI Path)**

The AI (Audio Input) processing chain follows this order:

```
Microphone → AI Device → HPF → AEC → ANR → AGC → Encoder → Output
```

1. **HPF (High Pass Filter)** - Removes low-frequency noise
2. **AEC (Acoustic Echo Cancellation)** - Removes speaker echo from microphone
3. **ANR (Automatic Noise Reduction)** - Suppresses background noise
4. **AGC (Automatic Gain Control)** - Normalizes audio levels

## **Audio Processing Pipeline (AO Path)**

The AO (Audio Output) processing chain follows this order:

```
Decoder → HPF → AGC/DRC → AO Device → Speaker
```

1. **HPF (High Pass Filter)** - Removes low-frequency distortion
2. **AGC/DRC (Dynamic Range Compression)** - Normalizes output levels and prevents clipping

---

## **Video Processing Pipeline - Detailed Block Diagram**

### Pipeline Architecture with Device/Channel/Port Structure

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          VIDEO PIPELINE ARCHITECTURE                            │
└─────────────────────────────────────────────────────────────────────────────────┘

┌────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  SENSOR    │      │     VIF      │      │     ISP      │      │   SCL DEV0   │
│            │      │              │      │              │      │              │
│  Sensor 0  │─────▶│  Dev: 0/4    │─────▶│  Dev: 0      │─────▶│  Dev: 0      │
│  (Pad 0)   │      │  Chn: 0      │      │  Chn: 0      │      │  Chn: 0      │
│            │      │  Port: 0     │      │  Port: 0     │      │  Port: 0, 1  │
└────────────┘      └──────────────┘      └──────────────┘      └──────┬───────┘
                    VIF Input             ISP Processing                │
                    Frontend              - AE/AWB/AF                   │
                                          - 3DNR Level 3                │
                                          - HDR Processing              │
                                                                       │
┌────────────┐      ┌──────────────┐      ┌──────────────┐             │
│  SENSOR    │      │     VIF      │      │     ISP      │             │
│            │      │              │      │              │      ┌──────▼───────┐
│  Sensor 1  │─────▶│  Dev: 8/12   │─────▶│  Dev: 0      │      │   SCL DEV1   │
│  (Pad 2)   │      │  Chn: 0      │      │  Chn: 1      │      │  (For IFORD) │
│  (Optional)│      │  Port: 0     │      │  Port: 0     │      │  Dev: 1      │
└────────────┘      └──────────────┘      └──────────────┘      │  Chn: 0,1,2  │
                                                                 │  Port: 0     │
                                                                 └──────┬───────┘
                                                                        │
                    ┌───────────────────────────────────────────────────┼──────┐
                    │                                                   │      │
                    │                                                   │      │
         ┌──────────▼──────────┐    ┌──────────▼──────────┐  ┌─────────▼──────┐
         │   VENC (Stream 0)   │    │   VENC (Stream 1)   │  │  VDF/IPU/VDF   │
         │   Main Stream        │    │   Sub Stream        │  │                │
         │  Dev: 0              │    │  Dev: 0             │  │  Chn: 0,1,2    │
         │  Chn: 0              │    │  Chn: 1             │  │  Port: 0       │
         │  Port: 0             │    │  Port: 0            │  └────────────────┘
         │                      │    │                     │
         │  H.264/H.265         │    │  H.264/H.265        │
         │  CBR/VBR             │    │  CBR/VBR            │
         └──────────┬───────────┘    └──────────┬──────────┘
                    │                           │
                    │  ┌─────────────┐          │
                    └─▶│  RGN/OSD    │◀─────────┘
                       │  Overlay    │
                       │  Handle 0-3 │
                       └─────────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Encoded Stream   │
                    │  Output (IPC/SHM) │
                    └───────────────────┘
```

### Device/Channel/Port Mapping Details

#### **1. VIF (Video Input Frontend)**

| Component | Sensor 0 (Pad 0) | Sensor 1 (Pad 2) | Purpose |
|-----------|------------------|------------------|---------|
| **Device** | 0 or 4 | 8 or 12 | Physical sensor interface |
| **Channel** | 0 | 0 | Video input channel |
| **Port** | 0 | 0 | Output port to ISP |
| **Format** | Bayer (RAW) | Bayer (RAW) | Raw sensor data |
| **Binding** | → ISP Dev:0 Chn:0 | → ISP Dev:0 Chn:1 | Bind to ISP |

**Pad Mapping:**
- Pad 0 → VIF Dev 0
- Pad 1 → VIF Dev 8
- Pad 2 → VIF Dev 4
- Pad 3 → VIF Dev 12

#### **2. ISP (Image Signal Processor)**

| Component | ISP Channel 0 | ISP Channel 1 | Purpose |
|-----------|---------------|---------------|---------|
| **Device** | 0 | 0 | ISP device (shared) |
| **Channel** | 0 | 1 | Processing channel per sensor |
| **Port** | 0 | 0 | Output port to SCL |
| **Input** | VIF Dev:0 Port:0 | VIF Dev:8 Port:0 | From VIF |
| **Output** | YUV420 SP | YUV420 SP | Processed video |
| **Binding** | → SCL Dev:0 Chn:0 | → SCL Dev:0 Chn:1 | Bind to SCL |

**ISP Features:**
- 3DNR Level 3 (or OFF for reduced memory)
- HDR Type: OFF or VC (Virtual Channel)
- AE/AWB/AF processing
- Mirror/Flip support
- Rotation support

#### **3. SCL (Scaler)**

**SCL Device 0 (Main Pipeline):**

| Component | Channel 0 | Purpose |
|-----------|-----------|---------|
| **Device** | 0 | Main scaler device |
| **Channel** | 0 | Scaling channel |
| **Port 0** | Main output | To VENC stream 0 |
| **Port 1** | Secondary output | To SCL Dev 1 (IFORD) |
| **HW SCL Mask** | HWSCL0 \| HWSCL1 | Hardware scaler ports |
| **Input** | ISP Dev:0 Chn:0 Port:0 | From ISP |
| **Binding** | → VENC Dev:0 Chn:0 | Main stream |

**SCL Device 1 (For IFORD Chip - Sub-Streams):**

| Component | Channel | Purpose | Binding |
|-----------|---------|---------|---------|
| **Device** | 1 | Secondary scaler device | - |
| **Channel 0** | 0 | IPU (Image Processing Unit) | For AI/Analytics |
| **Channel 1** | 1 | VDF (Video Defect/Motion Detect) | Motion detection |
| **Channel 2** | 2 | Sub-stream encoding | → VENC Dev:0 Chn:1 |
| **Port** | 0 | Output port | - |
| **HW SCL Mask** | HWSCL2 | Hardware scaler port 2 |
| **Input** | SCL Dev:0 Chn:0 Port:1 | From main SCL |

#### **4. VENC (Video Encoder)**

**Encoder Configuration:**

| Stream | Device | Channel | Port | Input Source | Output Format |
|--------|--------|---------|------|--------------|---------------|
| **Stream 0** | 0 | 0 | 0 | SCL Dev:0 Chn:0 Port:0 | H.264/H.265 Main |
| **Stream 1** | 0 | 1 | 0 | SCL Dev:1 Chn:2 Port:0 | H.264/H.265 Sub |
| **Stream 2** | 0 | 2 | 0 | SCL Dev:0 Chn:0 Port:0 | H.264/H.265 Third |
| **JPEG** | 8 | 0 | 0 | SCL Dev:2 Chn:0 Port:0 | JPEG Snapshot |

**VENC Parameters per Stream:**

| Parameter | Stream 0 (Main) | Stream 1 (Sub) | Stream 2 (Third) |
|-----------|-----------------|----------------|------------------|
| **Type** | H.264/H.265 | H.264/H.265 | H.264/H.265 |
| **RC Mode** | CBR/VBR | CBR/VBR | CBR/VBR |
| **Resolution** | Up to 4K | Up to 1080p | Configurable |
| **Bitrate** | Configurable | Configurable | Configurable |
| **GOP** | Configurable | Configurable | Configurable |
| **Profile** | Baseline/Main/High | Baseline/Main/High | Baseline/Main/High |
| **Frame Rate** | Up to 30fps | Up to 30fps | Up to 30fps |
| **Bind Type** | HW_RING/REALTIME | HW_RING/REALTIME | HW_RING/REALTIME |

#### **5. RGN (Region/OSD - On-Screen Display)**

| Handle | Attachment | Purpose |
|--------|------------|---------|
| **Handle 0** | VENC Dev:0 Chn:0 Port:0 | Main stream OSD |
| **Handle 1** | VENC Dev:0 Chn:1 Port:0 | Sub stream OSD |
| **Handle 2** | VENC Dev:0 Chn:2 Port:0 | Third stream OSD |
| **Handle 3** | Reserved | Additional overlay |
| **Handle 4+** | VDF Motion Detect | MD rectangle overlay |

**RGN Features:**
- Pixel Format: I4 (4-bit indexed) or ARGB1555
- Layer: 0-3
- Position: (X, Y) coordinates
- Alpha blending support
- Dynamic update support

#### **6. VDF (Video Defect/Motion Detection)**

| Component | Configuration |
|-----------|---------------|
| **Input** | SCL Dev:1 Chn:1 Port:0 |
| **Output** | Motion detection metadata |
| **Resolution** | Configurable grid (col x row) |
| **Algorithm** | SAD (Sum of Absolute Differences) |
| **Features** | MD (Motion Detection), OD (Object Detection) |

### Binding Types

| Bind Type | Usage | Characteristics |
|-----------|-------|-----------------|
| **E_MI_SYS_BIND_TYPE_REALTIME** | VIF→ISP, ISP→SCL | Real-time, low latency |
| **E_MI_SYS_BIND_TYPE_FRAME_BASE** | SCL→SCL, Multi-sensor | Frame-based buffering |
| **E_MI_SYS_BIND_TYPE_HW_RING** | SCL→VENC, VENC→VENC | Hardware ring buffer, multi-stream |

### Memory Pool Configuration

| Module | Pool Type | Configuration |
|--------|-----------|---------------|
| **SCL Dev 0** | Private Ring Pool | Max resolution + 2 lines aligned to 32 |
| **VENC Dev 0** | Private Ring Pool | Max resolution + 2 lines aligned to 32 |
| **Heap** | MMA Heap | "mma_heap_name0" |

### Complete Data Flow Example

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                        MULTI-STREAM VIDEO PIPELINE                           │
└──────────────────────────────────────────────────────────────────────────────┘

Sensor 0 (2MP@30fps RAW)
    │
    ├─▶ VIF Dev:0 Chn:0 Port:0 (RAW → ISP)
         │
         └─▶ ISP Dev:0 Chn:0 Port:0 (RAW → YUV420, 3DNR, AWB, AE)
              │
              └─▶ SCL Dev:0 Chn:0 Port:0 (1920x1080 YUV420)
                   │
                   ├─▶ VENC Dev:0 Chn:0 Port:0 (H.265 Main Stream - 4Mbps CBR)
                   │    │
                   │    └─▶ RGN Handle:0 (Timestamp OSD)
                   │         │
                   │         └─▶ Encoded Stream 0 → IPC/Shared Memory
                   │
                   └─▶ SCL Dev:0 Chn:0 Port:1 (Full Resolution Pass-through)
                        │
                        └─▶ SCL Dev:1 Chn:2 Port:0 (640x360 YUV420)
                             │
                             └─▶ VENC Dev:0 Chn:1 Port:0 (H.264 Sub Stream - 512Kbps VBR)
                                  │
                                  └─▶ RGN Handle:1 (Logo OSD)
                                       │
                                       └─▶ Encoded Stream 1 → IPC/Shared Memory
```

### Key Notes

1. **Device Numbers**: Fixed hardware resources (VIF, ISP, SCL, VENC devices)
2. **Channel Numbers**: Logical processing channels within a device
3. **Port Numbers**: Output ports from a module (0-based indexing)
4. **Binding**: Connects source module's output port to destination module's input
5. **Frame Rate Control**: Source FPS → Destination FPS with frame dropping
6. **Ring Buffer**: Used for multi-stream encoding from single SCL output

---

## **Configuration Files**

- **mediaserver.cfg** - Main configuration file for video/audio settings
- **audio_ai_ao_init.json** - JSON configuration for audio initialization
- **APC_AEC_CONFIG_config_V03.cfg** - Audio algorithm tuning parameters (HPF/AEC/ANR/AGC)
- **PreloadSetting.txt** - Preload buffer settings

---

**Document Generated:** January 8, 2026  
**Source File:** `st_main_mediaserver.c` (11,296 lines)  
**Total Functions Documented:** 87
