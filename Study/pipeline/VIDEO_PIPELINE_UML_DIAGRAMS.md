# Video Pipeline - Comprehensive UML Documentation

## Table of Contents
1. [Class Diagrams](#class-diagrams)
2. [Sequence Diagrams](#sequence-diagrams)
3. [State Machine Diagrams](#state-machine-diagrams)
4. [Component Diagrams](#component-diagrams)
5. [Use Case Diagrams](#use-case-diagrams)
6. [Activity Diagrams](#activity-diagrams)
7. [Deployment Diagrams](#deployment-diagrams)

---

## Class Diagrams

### Main Data Structures Class Hierarchy

**yUML Class Diagram:**

```
// Abstract base module
[<<abstract>> PipelineModule|#moduleId:MI_MODULE_ID_e;#deviceId:MI_U32;#channelId:MI_U32;#isEnabled:MI_BOOL|+create():MI_S32;+destroy():MI_S32;+configure():MI_S32;+getStatus():ModuleStatus;+query():MI_VENC_ChnStat_t]

// VIF Module
[VIF|inputBuff[];sensorFmt|]
[PipelineModule]^-[VIF]

// ISP Module
[ISP|procBuff[];bayer;yuv|]
[PipelineModule]^-[ISP]

// SCL Module
[SCL|outBuf0;outBuf1;outBuf2|]
[PipelineModule]^-[SCL]

// VENC Module
[VENC|encBuff[];refFrames[];mbuf|]
[PipelineModule]^-[VENC]

// RGN Module
[RGN||]
[PipelineModule]^-[RGN]

// VDF Module
[VDF|detBuff[]|]
[PipelineModule]^-[VDF]
```

**Render at:** `https://yuml.me/diagram/plain/class/[diagram-code]`

### Stream Configuration Class

**yUML Class Diagram:**

```
[ST_Stream_Attr_T|+bEnable:MI_BOOL;+eInputModule:MI_MODULE_ID_e;+u32InputDev:MI_U32;+u32InputChn:MI_U32;+u32InputPort:MI_U32;+u32VencDev:MI_U32;+vencChn:MI_VENC_CHN;+eType:MI_VENC_MODTYPE_e;+f32Mbps:MI_F32;+u32Width:MI_U32;+u32Height:MI_U32;+u32MaxWidth:MI_U32;+u32MaxHeight:MI_U32;+u32CropX:MI_U32;+u32CropY:MI_U32;+u32CropWidth:MI_U32;+u32CropHeight:MI_U32;+enFunc:MI_U32;+pszStreamName:char*;+eBindType:MI_SYS_BindType_e;+u32BindPara:MI_U32|+isValid():MI_BOOL;+calculateFrameSize():MI_U32;+calculateBitrate():MI_U32;+getCodecInfo():CodecInfo]
```

**Render at:** `https://yuml.me/diagram/plain/class/[diagram-code]`

### Binding Configuration Class

**yUML Class Diagram:**

```
[ST_SysBindPort_T|+u32SrcDevId:MI_U32;+eInputModuleId:MI_MODULE_ID_e;+u32SrcChnId:MI_U32;+u32SrcPortId:MI_U32;+u32DstDevId:MI_U32;+eOutputModuleId:MI_MODULE_ID_e;+u32DstChnId:MI_U32;+u32DstPortId:MI_U32;+u32SrcFrameRate:MI_U32;+u32DstFrameRate:MI_U32;+eBindType:MI_SYS_BindType_e;+u32BindPara:MI_U32|+validate():MI_BOOL;+getLatency():MI_U32;+getMemoryUsage():MI_U32]
```

**Render at:** `https://yuml.me/diagram/plain/class/[diagram-code]`

### Encoder Configuration Class

**yUML Class Diagram:**

```
[MI_VENC_ChnAttr_t|+stVencAttr:VencAttr;+stRcAttr:RateCtrlAttr;+stVuiParam:VuiParam;+stJpegParam:JpegParam;+u32Priority:MI_U32|+setCodecType(type):void;+setRateControl(mode,param):void;+setVUI(param):void;+validate():MI_S32;+getMemoryRequirement():MI_U32]
[RateCtrlAttr|+eRcMode:enum;+u32BitRate:MI_U32;+u32FrameRate:MI_U32;+u32Gop:MI_U32;+u32Quality:MI_U32|+validate():MI_BOOL;+getMemory():MI_U32]
[MI_VENC_ChnAttr_t]uses-.->[RateCtrlAttr]
[note: eRcMode supports CBR/VBR/AVBR{bg:wheat}]
```

**Render at:** `https://yuml.me/diagram/plain/class/[diagram-code]`

### Pipeline Manager Class

**yUML Class Diagram:**

```
[MediaServerPipelineManager|-vifModule:VIF_Module;-ispModule:ISP_Module;-scalModule:SCL_Module;-vencModule:VENC_Module;-streamConfig[]:ST_Stream_Attr_T;-bindings[]:ST_SysBindPort_T;-threads:pthread_t[];-isRunning:MI_BOOL|+initialize():MI_S32;+start():MI_S32;+stop():MI_S32;+addStream(config):MI_S32;+removeStream(streamId):MI_S32;+configureStream(config):MI_S32;+bindModules(binding):MI_S32;+unbindModules(binding):MI_S32;+getStatus():PipelineStatus;+queryStream(streamId):StreamStatus;+enableStream(streamId):MI_S32;+disableStream(streamId):MI_S32;+applyISPSettings(settings):MI_S32;+getMetrics():PerformanceMetrics]
[MediaServerPipelineManager]->[VIF]
[MediaServerPipelineManager]->[ISP]
[MediaServerPipelineManager]->[SCL]
[MediaServerPipelineManager]->[VENC]
[MediaServerPipelineManager]1-*>[ST_Stream_Attr_T]
[MediaServerPipelineManager]1-*>[ST_SysBindPort_T]
```

**Render at:** `https://yuml.me/diagram/plain/class/[diagram-code]`

---

## Sequence Diagrams

### Pipeline Initialization Sequence

**yUML Sequence Diagram:**

```
// Pipeline Initialization
[User/App]-initialize()>[Manager]
[Manager]-create()>[VIF]
[VIF]-->[Manager]
[Manager]-create()>[ISP]
[ISP]-->[Manager]
[Manager]-create()>[SCL]
[SCL]-->[Manager]
[Manager]-create()>[VENC]
[VENC]-->[Manager]
[Manager]-bind(VIF→ISP)>[System]
[System]-->[Manager]
[Manager]-bind(ISP→SCL)>[System]
[System]-->[Manager]
[Manager]-bind(SCL→VENC)>[System]
[System]-->[Manager]
[Manager]-success>[User/App]
```

**Render at:** `https://yuml.me/diagram/scruffy/sequence/[diagram-code]`

### Video Frame Processing Sequence

**yUML Sequence Diagram:**

```
// Frame Processing Flow (30fps)
[Sensor]-raw frame>[VIF]
[VIF]-process()>[ISP]
[ISP]-YUV420 3840x2160>[SCL]
[SCL]-scale(4K)>[VENC]
[SCL]-scale(1080p)>[VENC]
[SCL]-scale(snapshot)>[VENC]
[VENC]-encode(H265/H264/JPEG)>[Prebuffer]
[Prebuffer]-bitstream>[RTSP]
[RTSP]-store(100 frames)>[Buffer]
[RTSP]-transmit>[Clients]
[note: 30fps = 33.3ms per cycle{bg:wheat}]
```

**Render at:** `https://yuml.me/diagram/scruffy/sequence/[diagram-code]`

### Stream Enable/Disable Sequence

**yUML Sequence Diagram:**

```
// Enable Stream
[User]-enable(1)>[Manager]
[Manager]-validate()>[Config]
[Config]-ok>[Manager]
[Manager]-create()>[Module]
[Module]-ok>[Manager]
[Manager]-configure()>[Module]
[Module]-ok>[Manager]
[Manager]-bind()>[Binding]
[Binding]-ok>[Manager]
[Manager]-success>[User]

// Disable Stream
[User]-disable(1)>[Manager]
[Manager]-unbind()>[Binding]
[Binding]-ok>[Manager]
[Manager]-destroy()>[Module]
[Module]-ok>[Manager]
[Manager]-success>[User]
```

**Render at:** `https://yuml.me/diagram/scruffy/sequence/[diagram-code]`

### Rate Control Configuration Sequence

**yUML Sequence Diagram:**

```
// Rate Control Configuration
[User/App]-setRate()>[Manager]
[Manager]-validate()>[RateControl]
[RateControl]-ok>[Manager]
[Manager]-setAttr()>[VENC]
[VENC]-setMode()>[RateControl]
[RateControl]-ok>[VENC]
[VENC]-setParam()>[RateControl]
[RateControl]-ok>[VENC]
[VENC]-ok>[Manager]
[Manager]-success>[User/App]
[note: Encoding continues with new rate (CBR/VBR/AVBR){bg:wheat}]
```

**Render at:** `https://yuml.me/diagram/scruffy/sequence/[diagram-code]`

---

## State Machine Diagrams

### Pipeline Module State Machine

**yUML Activity Diagram:**

```
// Module Lifecycle State Machine
(start)-initialize()->(CREATED)
(CREATED)-configure()->(CONFIGURED)
(CONFIGURED)-bind()->(BOUND)
(CONFIGURED)->(IDLE)
(BOUND)-start()->(RUNNING)
(IDLE)-start()->(RUNNING)
(RUNNING)-stop()->(PAUSED)
(PAUSED)-resume()->(RUNNING)
(PAUSED)-destroy()->(DESTROYED)
(RUNNING)-destroy()->(DESTROYED)
(DESTROYED)->(end)
```

**States:**
- `CREATED` - Module initialized
- `CONFIGURED` - Parameters set
- `BOUND/IDLE` - Connected or standalone
- `RUNNING` - Active processing
- `PAUSED` - Temporarily stopped
- `DESTROYED` - Cleaned up

**Render at:** `https://yuml.me/diagram/scruffy/activity/[diagram-code]`

### Stream Processing State Machine

**yUML Activity Diagram:**

```
// Stream Processing Lifecycle
(start)-create stream->(DISABLED)
(DISABLED)-enable()->(ENABLED)
(ENABLED)-waitBuffer()->(CAPTURING)
(CAPTURING)-encodeFrame()->(ENCODING)
(ENCODING)-getNALUnits()->(READY_TRANSMIT)
(READY_TRANSMIT)-next frame->(ENABLED)
(READY_TRANSMIT)-disable()->(DISABLED)
(DISABLED)-destroy()->(DESTROYED)
(DESTROYED)->(end)
```

**Render at:** `https://yuml.me/diagram/scruffy/activity/[diagram-code]`

### Encoder Rate Control State Machine

**yUML Activity Diagram:**

```
// Rate Control State Machine
(start)-setRateControl()->(IDLE)
(IDLE)<CBR>-monitor bitrate->(RATE_CONTROLLED)
(IDLE)<VBR>-monitor quality->(RATE_CONTROLLED)
(IDLE)<AVBR>-monitor variation->(RATE_CONTROLLED)
(RATE_CONTROLLED)-queryStats()->(STATS_READY)
(STATS_READY)-adjustRate()->(ADJUSTED)
(ADJUSTED)->(RATE_CONTROLLED)
(RATE_CONTROLLED)-disableRateControl()->(end)
```

**Modes:**
- **CBR** (Constant Bitrate) - Fixed bitrate monitoring
- **VBR** (Variable Bitrate) - Quality-based monitoring
- **AVBR** (Adaptive VBR) - Variation monitoring

**Render at:** `https://yuml.me/diagram/scruffy/activity/[diagram-code]`

---

## Component Diagrams

### High-Level System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      Video Pipeline System                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │ Sensor Input│                                                │
│  └──────┬──────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              VIF Component                               │  │
│  │ ┌──────────────────────────────────────────────────────┐ │  │
│  │ │ Raw Bayer Capture (10-bit)                          │ │  │
│  │ │ - 4 Ring Buffers @ 10.4 MB                          │ │  │
│  │ │ - Frame synchronization                            │ │  │
│  │ │ - Metadata injection                               │ │  │
│  │ └──────────────────────────────────────────────────────┘ │  │
│  └──────────────┬───────────────────────────────────────────┘  │
│                 │                                              │
│                 ▼                                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              ISP Component                               │  │
│  │ ┌──────────────────────────────────────────────────────┐ │  │
│  │ │ Image Signal Processing                             │ │  │
│  │ │ - Bayer Demosaicing                                │ │  │
│  │ │ - AE/AWB/3DNR/LDC                                 │ │  │
│  │ │ - Color Correction                                │ │  │
│  │ │ - YUV420 Output (3 Buffers @ 12.4 MB)             │ │  │
│  │ └──────────────────────────────────────────────────────┘ │  │
│  └──────────────┬───────────────────────────────────────────┘  │
│                 │                                              │
│                 ▼                                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              SCL Component                               │  │
│  │ ┌──────────────────────────────────────────────────────┐ │  │
│  │ │ Multi-Output Scaler                                │ │  │
│  │ │                                                    │ │  │
│  │ │ Port 0: 3840x2160 (37.2 MB, 3×)                  │ │  │
│  │ │ Port 1: 1920x1080 (9.3 MB, 3×)                   │ │  │
│  │ │ Port 2: 640x360 (1.02 MB, 3×)                    │ │  │
│  │ │                                                    │ │  │
│  │ │ - Interpolation filtering                        │ │  │
│  │ │ - Frame scheduling                               │ │  │
│  │ └──────────────────────────────────────────────────────┘ │  │
│  └──┬──────────────────────┬──────────────────┬──────────────┘  │
│     │                      │                  │                │
│     ▼                      ▼                  ▼                │
│ ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐   │
│ │  VENC Dev 0      │  │  VENC Dev 0      │  │  VENC Dev 8  │   │
│ │  H265 Encoder    │  │  H264 Encoder    │  │  JPEG Encoder│   │
│ │  (Ch 0)          │  │  (Ch 1)          │  │  (Ch 2)      │   │
│ │ ┌──────────────┐ │  │ ┌──────────────┐ │  │ ┌──────────┐ │   │
│ │ │ 4K @ 2 Mbps  │ │  │ │1080p @ 1 Mbps│ │  │ │Snapshot  │ │   │
│ │ │ 30fps        │ │  │ │ 30fps        │ │  │ │640x360   │ │   │
│ │ │Ref: 24.8 MB  │ │  │ │Ref: 6.2 MB   │ │  │ │3 MB      │ │   │
│ │ └──────────────┘ │  │ └──────────────┘ │  │ └──────────┘ │   │
│ └────────┬─────────┘  │  └────────┬───────┘  │  └──────┬─────┘   │
│          │            │           │         │         │        │
│          └────────────┼───────────┴─────────┘─────────┘        │
│                       ▼                                        │
│          ┌──────────────────────────────────┐                 │
│          │     Prebuffer Component          │                 │
│          │ ┌──────────────────────────────┐ │                 │
│          │ │ Circular Frame Buffer        │ │                 │
│          │ │ - 100 Frames @ 200 KB (avg) │ │                 │
│          │ │ - 20.5 MB total             │ │                 │
│          │ │ - Ring management            │ │                 │
│          │ │ - PTS tracking               │ │                 │
│          │ └──────────────────────────────┘ │                 │
│          └────────────┬─────────────────────┘                 │
│                       │                                       │
│                       ▼                                       │
│          ┌──────────────────────────────────┐                 │
│          │      RTSP Output Component       │                 │
│          │ ┌──────────────────────────────┐ │                 │
│          │ │ - NAL Unit Extraction        │ │                 │
│          │ │ - Frame Packetization        │ │                 │
│          │ │ - RTP Streaming              │ │                 │
│          │ │ - Multi-client Support       │ │                 │
│          │ └──────────────────────────────┘ │                 │
│          └────────────┬─────────────────────┘                 │
│                       │                                       │
│                       ▼                                       │
│          ┌──────────────────────────────────┐                 │
│          │   Network Output (Clients)       │                 │
│          └──────────────────────────────────┘                 │
│                                                                │
└──────────────────────────────────────────────────────────────────┘
```

### Modular Binding Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                 Module Binding Structure                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VIF ──binding──► ISP ──binding──► SCL ──binding──► VENC       │
│                              │                      │           │
│                              └──binding───┬─────────┘           │
│                                           │                    │
│                                    JPEG VENC                   │
│                                           │                    │
│                              ISP ─ SCL ─ VENC                  │
│                              (optional secondary scaler)        │
│                                                                 │
│  Each Binding Has:                                              │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ - Source Module (VIF, ISP, SCL, VENC)                │    │
│  │ - Source Port (0-2 for multi-output modules)          │    │
│  │ - Destination Module                                  │    │
│  │ - Destination Port                                    │    │
│  │ - Bind Type (HW_RING, FRAME_BASE, REALTIME)          │    │
│  │ - Frame Rate Control                                  │    │
│  │ - Buffer Depth                                        │    │
│  │ - Latency Specification                               │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Binding Configuration Table:                                   │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Binding  │  Src Module  │  Dst Module │  Bind Type  │    │
│  ├───────────┼──────────────┼─────────────┼──────────────┤    │
│  │  1        │  VIF(0,0,0)  │  ISP(0,0,0) │  FRAME_BASE  │    │
│  │  2        │  ISP(0,0,0)  │  SCL(0,0,0) │  HW_RING     │    │
│  │  3        │  SCL(0,0,0)  │  VENC(0,0,0)│  HW_RING     │    │
│  │  4        │  SCL(0,0,1)  │  VENC(0,1,0)│  FRAME_BASE  │    │
│  │  5        │  SCL(0,0,2)  │  VENC(8,2,0)│  FRAME_BASE  │    │
│  │  6        │  VENC(0,0,0) │  VENC(0,1,0)│  HW_RING     │    │
│  │           │  (opt)       │   (optional)│  (sub-stream)│    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Use Case Diagrams

### System Use Cases

**yUML Use Case Diagram:**

```
// Main Use Cases
[User]-(Initialize Pipeline)
[User]-(Configure Stream)
[User]-(Monitor Pipeline)
[User]-(Start Capture)
[User]-(Enable/Disable Stream)
[User]-(Get Performance Metrics)
[User]-(Encode Frames)
[User]-(Change Resolution)
[User]-(Diagnose Issues)
[User]-(Stream Output)
[User]-(Apply ISP Settings)

// Use Case Relationships
(Initialize Pipeline)<(Start Capture)
(Configure Stream)<(Enable/Disable Stream)
(Configure Stream)<(Change Resolution)
(Monitor Pipeline)<(Get Performance Metrics)
(Monitor Pipeline)<(Diagnose Issues)
(Encode Frames)>(Stream Output)
```

**Render at:** `https://yuml.me/diagram/scruffy/usecase/[diagram-code]`

### Configuration Use Cases

**yUML Use Case Diagram:**

```
// Configuration Use Cases
[Configuration UI]-(Resolution Setting)
[Configuration UI]-(Codec Selection)
[Configuration UI]-(Bitrate Control)

(Resolution Setting)>(Set Width/Height)
(Codec Selection)>(Select H265/H264/JPEG)
(Bitrate Control)>(Set CBR/VBR/AVBR)

(Set Width/Height)>(Validate Configuration)
(Select H265/H264/JPEG)>(Validate Configuration)
(Set CBR/VBR/AVBR)>(Validate Configuration)

(Validate Configuration)>(Apply Config)
(Validate Configuration)>(Show Error Message)
```

**Render at:** `https://yuml.me/diagram/scruffy/usecase/[diagram-code]`

---

## Activity Diagrams

### Frame Processing Activity Flow

```
                    ┌─────────────┐
                    │   [*] START │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ Sensor Input│
                    │ (30 fps)    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │  VIF Capture│
                    │   Raw Bayer │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ ISP Process │
                    │ Demosaic    │
                    │ AE/AWB      │
                    │ 3DNR/LDC    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌──────────────────┐
                    │  Convert to      │
                    │  YUV420 @ 3840   │
                    │  x2160           │
                    └──────┬───────────┘
                           │
                  ┌────────┴──────────┐
                  │                   │
                  ▼                   ▼
            ┌──────────────┐   ┌─────────────┐
            │  SCL Port 0  │   │  SCL Port 1 │
            │  3840x2160   │   │  1920x1080  │
            └──────┬───────┘   └──────┬──────┘
                   │                  │
          ┌────────┴───────┐          │
          │                │          │
          ▼                ▼          ▼
    ┌──────────┐   ┌──────────┐  ┌────────┐
    │H265 Enc. │   │H264 Enc. │  │JPEG    │
    │2 Mbps    │   │1 Mbps    │  │Enc.    │
    └────┬─────┘   └────┬─────┘  └───┬────┘
         │              │            │
         │              │      ┌──────┴──────┐
         │              │      │             │
         │              │      ▼             │
         │              │  ┌─────────────┐  │
         │              │  │ JPEG Output │  │
         │              │  └─────────────┘  │
         │              │                   │
         ├──────────────┼───────────────────┤
         │              │                   │
         ▼              ▼                   ▼
    ┌──────────────────────────────────────┐
    │      Prebuffer (100 frames)          │
    │  - Store bitstream                   │
    │  - Maintain PTS timestamps           │
    │  - Ring buffer management            │
    └──────────┬───────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────┐
    │     NAL Unit Extraction              │
    │  - Parse encoded data                │
    │  - Extract SPS/PPS/VPS               │
    │  - Prepare RTP packets               │
    └──────────┬───────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────┐
    │     RTSP Stream Transmission         │
    │  - Multiple client support           │
    │  - Network packetization             │
    │  - Rate adaptation                   │
    └──────────┬───────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────┐
    │        Network Clients               │
    │  - Playback                          │
    │  - Recording                         │
    │  - Analysis                          │
    └──────────┬───────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────┐
    │   End of Frame Processing            │
    │   Wait for next frame (33.3ms)       │
    └──────────┬───────────────────────────┘
               │
               ▼
    ┌─────────────┐
    │  [*] END    │
    └─────────────┘
```

### ISP Configuration Activity Flow

```
              ┌────────────────┐
              │  User Requests │
              │  ISP Changes   │
              └────────┬───────┘
                       │
                       ▼
              ┌────────────────┐
              │  Parse Request │
              │  (brightness,  │
              │   contrast,    │
              │   saturation)  │
              └────────┬───────┘
                       │
                       ▼
              ┌────────────────┐
              │  Validate      │
              │  Parameters    │
              └────────┬───────┘
                       │
              ┌────────┴────────┐
              │                 │
           Invalid           Valid
              │                 │
              ▼                 ▼
        ┌──────────┐      ┌──────────────┐
        │Return    │      │Calculate IQ  │
        │Error     │      │Adjustment    │
        └──────────┘      └──────┬───────┘
                                 │
                                 ▼
                         ┌──────────────┐
                         │Update ISP    │
                         │LUT Tables    │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │Apply to Live │
                         │Pipeline      │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │Monitor Result│
                         │(next frames) │
                         └──────┬───────┘
                                │
                                ▼
                         ┌──────────────┐
                         │  [*] Done    │
                         └──────────────┘
```

---

## Deployment Diagrams

### Hardware Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  Innofusion SoC Device                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  Sensor Interface                        │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │  │
│  │  │ Sensor 1     │  │ Sensor 2     │  │ Sensor N     │  │  │
│  │  │ (Bayer RGB)  │  │ (Bayer RGB)  │  │ (Bayer RGB)  │  │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │  │
│  └──────────┬──────────────────────────────────────────────┘  │
│             │                                                  │
│             ▼                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │            VIF (Video Input Front)                       │  │
│  │         [Hardware Module - Ring Buffers]                │  │
│  └──────────┬──────────────────────────────────────────────┘  │
│             │                                                  │
│             ▼                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │      ISP (Image Signal Processing)                       │  │
│  │  [Hardware Accelerated Module]                           │  │
│  │  - Demosaicing Engine                                    │  │
│  │  - AE/AWB Engine                                         │  │
│  │  - 3DNR Processing Unit                                  │  │
│  │  - LDC Engine                                            │  │
│  │  - Color Processing                                      │  │
│  └──────────┬──────────────────────────────────────────────┘  │
│             │                                                  │
│             ▼                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │       SCL (Scaler - Multi-output)                        │  │
│  │  [Hardware Module - 3 Output Ports]                      │  │
│  │  - Interpolation Unit                                    │  │
│  │  - Frame Scheduling                                      │  │
│  └──┬─────────────────────┬─────────────────────┬───────────┘  │
│    │                     │                     │              │
│    ▼                     ▼                     ▼              │
│  ┌────────┐         ┌─────────┐          ┌─────────┐         │
│  │Port 0  │         │Port 1   │          │Port 2   │         │
│  │4K      │         │1080p    │          │Snapshot │         │
│  └────┬───┘         └────┬────┘          └────┬────┘         │
│       │                  │                    │               │
│       └──────────────────┼────────────────────┘               │
│                          │                                    │
│                          ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           VENC - H264/H265 Encoder                       │  │
│  │         [Hardware Codec Engine - Dev 0]                  │  │
│  │  - H265 Main Stream @ 4K                                 │  │
│  │  - H264 Sub-Stream @ 1080p                               │  │
│  └──────┬───────────────────────────────────────────────────┘  │
│         │                                                      │
│         ├─────────────────────┐                               │
│         │                     │                               │
│         ▼                     ▼                               │
│  ┌─────────────────┐  ┌──────────────────┐                  │
│  │ Main Stream Out │  │ Sub-Stream Out   │                  │
│  │ (H265 @ 2 Mbps)│  │ (H264 @ 1 Mbps)  │                  │
│  └────────┬────────┘  └────────┬─────────┘                  │
│           │                    │                             │
│           └────────┬───────────┘                             │
│                    │                                         │
│                    ▼                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │      VENC - JPEG Encoder                                 │  │
│  │      [Hardware Module - Dev 8]                           │  │
│  │      - JPEG Snapshot @ 640x360                           │  │
│  └──────────┬───────────────────────────────────────────────┘  │
│             │                                                  │
│             ▼                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           System Memory                                  │  │
│  │  ┌──────────────────────────────────────────────────┐   │  │
│  │  │ - Frame Buffers (~200 MB)                       │   │  │
│  │  │ - Prebuffer (~20 MB)                            │   │  │
│  │  │ - Configuration Structures (~5 MB)              │   │  │
│  │  │ - Thread Stacks & Synchronization (~5 MB)       │   │  │
│  │  └──────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           CPU Core Pool                                  │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │  │
│  │  │ Pipeline     │  │ Audio        │  │ Application  │  │  │
│  │  │ Threads      │  │ Threads      │  │ Threads      │  │  │
│  │  │ (~50% util)  │  │ (~10% util)  │  │ (~20% util)  │  │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                             │
                             ▼
                ┌────────────────────────┐
                │   Network Interface    │
                │   (RTSP Output)        │
                │   - Ethernet / WiFi    │
                └────────────────────────┘
                             │
                             ▼
          ┌──────────────────────────────────────┐
          │        Client Devices                │
          │  - Playback                          │
          │  - Recording                         │
          │  - Analysis                          │
          └──────────────────────────────────────┘
```

### Software Stack Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                  Application Layer                          │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  AGW MediaServer Control API                         │ │
│  │  agw_mediaserver_*.c                                 │ │
│  └───────────────────────────────────────────────────────┘ │
└────────────┬────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│              Pipeline Management Layer                      │
│  ┌───────────────────────────────────────────────────────┐ │
│  │  MediaServerPipelineManager                          │ │
│  │  - Initialization & Control                          │ │
│  │  - Stream Configuration                              │ │
│  │  - Binding Management                                │ │
│  │  - ISP Control Functions                             │ │
│  │  - Prebuffer Management                              │ │
│  └───────────────────────────────────────────────────────┘ │
└────────────┬────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│            Module Abstraction Layer                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ VIF Mgr  │  │ ISP Mgr  │  │ SCL Mgr  │  │VENC Mgr  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  st_main_mediaserver.c implementation                      │
└────────────┬────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│           Hardware Abstraction Layer (HAL)                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  MI (Multimedia Interface) Library                  │  │
│  │  - MI_VIF_*  (VIF Module Interface)                │  │
│  │  - MI_ISP_*  (ISP Module Interface)                │  │
│  │  - MI_SCL_*  (Scaler Module Interface)             │  │
│  │  - MI_VENC_* (Encoder Module Interface)            │  │
│  │  - MI_SYS_*  (System Binding Interface)            │  │
│  │  - MI_RGN_*  (Region/OSD Interface)                │  │
│  │  - MI_VDF_*  (Video Detection Interface)           │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────┬────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│             Kernel Driver Layer                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Media Server Kernel Drivers                        │  │
│  │  - ISP Driver                                       │  │
│  │  - VIF Driver                                       │  │
│  │  - Scaler Driver                                    │  │
│  │  - Encoder Driver (Hardware codec)                  │  │
│  │  - DMA Engine                                       │  │
│  │  - Ring Buffer Manager                             │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────┬────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────┐
│            Hardware (SoC)                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  VIF / ISP / SCL / VENC / Memory Controllers        │  │
│  └──────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## Data Structure Relationships

### Complete Class Relationship Diagram

```
┌────────────────────────────────────────────────────────────────┐
│           Complete System Data Model                           │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  ST_Config_S (System Configuration)                    │ │
│  │  - Device parameters                                   │ │
│  │  - Boot settings                                       │ │
│  │  - Stream count                                        │ │
│  └────────────┬────────────────────────────────────────────┘ │
│               │ contains                                      │
│               ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  ST_Stream_Attr_T[] (Stream Array)                     │ │
│  │  [0] Main Stream (4K H265)                             │ │
│  │  [1] Sub Stream (1080p H264)                           │ │
│  │  [2] JPEG Snapshot                                     │ │
│  │  [3] Reserved                                          │ │
│  │  ...                                                   │ │
│  └────────────┬────────────────────────────────────────────┘ │
│               │ references                                    │
│               ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  ST_SysBindPort_T[] (Binding Array)                    │ │
│  │  Binding 0: VIF → ISP                                  │ │
│  │  Binding 1: ISP → SCL                                  │ │
│  │  Binding 2: SCL → VENC                                 │ │
│  │  Binding 3: VENC → VENC (sub-stream)                   │ │
│  │  Binding 4: SCL → JPEG VENC                            │ │
│  └────────────┬────────────────────────────────────────────┘ │
│               │ uses                                         │
│               ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  MI_VENC_ChnAttr_t (Encoder Channel Attribute)        │ │
│  │  ├─ stVencAttr                                         │ │
│  │  │  ├─ eType (H264/H265/JPEG)                         │ │
│  │  │  ├─ u32MaxWidth, u32MaxHeight                      │ │
│  │  │  └─ u32BufSize                                     │ │
│  │  ├─ stRcAttr (Rate Control)                           │ │
│  │  │  ├─ eRcMode (CBR/VBR/AVBR)                         │ │
│  │  │  ├─ u32BitRate                                     │ │
│  │  │  └─ stParamCbr / stParamVbr                        │ │
│  │  └─ stJpegParam (JPEG specific)                       │ │
│  └────────────┬────────────────────────────────────────────┘ │
│               │ references                                    │
│               ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  H264_frame_t[] (Prebuffer Frames)                     │ │
│  │  [0..99] Frame data + metadata                         │ │
│  │  - pData (encoded bitstream)                           │ │
│  │  - u32DataLen                                          │ │
│  │  - u64PTS (timestamp)                                  │ │
│  │  - bIsKeyFrame                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                                │
│  Relationships:                                                │
│  • ST_Config_S HAS-MANY ST_Stream_Attr_T (array)             │
│  • ST_Config_S HAS-MANY ST_SysBindPort_T (bindings)          │
│  • ST_Stream_Attr_T USES MI_VENC_ChnAttr_t (codec config)    │
│  • MI_VENC_ChnAttr_t CONTAINS RateCtrlAttr (rate control)    │
│  • Prebuffer STORES H264_frame_t[] (circular buffer)         │
│  • Pipeline EXECUTES Stream Processing Flow                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## Summary

This comprehensive UML documentation provides:

1. **Class Diagrams** - Data structures and their relationships
2. **Sequence Diagrams** - Interaction flows for initialization, processing, and configuration
3. **State Machines** - Module lifecycle and processing states
4. **Component Diagrams** - High-level system architecture and module binding
5. **Use Cases** - System capabilities and user interactions
6. **Activity Diagrams** - Detailed process flows for frame processing and ISP configuration
7. **Deployment Diagrams** - Hardware and software stack architecture
8. **Data Structure Relationships** - Complete system data model

### Key UML Insights:

**Module Hierarchy**: VIF → ISP → SCL → VENC pipeline with parallel JPEG encoding
**Binding Pattern**: Point-to-point connections with configurable bind types (HW_RING, FRAME_BASE, REALTIME)
**Stream Management**: 4-stream array with independent enable/disable capability
**Rate Control**: CBR/VBR/AVBR modes with configurable bitrate and frame rate
**Memory Architecture**: Ring buffers at each stage with 200-250 MB total allocation
**Processing Flow**: 33.3ms per frame at 30fps with optional ISP processing

All diagrams are ASCII-rendered for easy integration into documentation and version control systems.
