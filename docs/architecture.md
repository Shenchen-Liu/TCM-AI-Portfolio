# 🏗️ ToB 四诊采集工作站｜系统架构与能力扩展

[返回作品集](../README.md) · [技术决策](design-decisions.md) · [工程案例](engineering-cases.md)

以 **ToB 四诊采集工作站**为核心，连接 Qwen 智能底座、双屏数字人交互、专业视觉算法服务及脉诊仪等外部设备，同时为诊断子 Agent 和个性化辅助诊疗预留能力扩展空间。**实线代表现有服务与数据流，虚线代表集成推进或后续规划。**

## 01｜整体架构：Qwen 底座、采集 Agent 与专业 Tool

```mermaid
flowchart TB
  subgraph Human["ToB 四诊采集工作站 · 交互入口"]
    PAT["患者屏 · Vue / Electron<br/>数字人 / 字幕 / 引导"]
    STAFF["医助屏 · Vue / Electron<br/>发起 / 监控 / 复核"]
    OAC["OpenAvatarChat<br/>VAD / ASR / TTS / 数字人"]
  end
  subgraph Brain["Qwen 智能底座 · AI 业务中枢"]
    BACK["FastAPI<br/>会话 API / WS / 只读 SSE / 设备协调"]
    GRAPH["LangGraph 采集 Agent<br/>问诊子图 / 阶段控制 / 检查点"]
    QWEN["🧠 Qwen3.5-9B 智能底座<br/>当前：理解 / 字段提取 / 问句生成<br/>本地 SGLang 推理服务"]
  end
  subgraph Sources["外部硬件与专用算法"]
    CAM["摄像头 / 补光灯<br/>图片 · 视频 · 媒体服务"]
    MIC["麦克风 / 音响"]
    PULSE["脉诊仪 / 外部传感设备"]
    PULSAPP["脉诊采集与波形处理<br/>设备适配 / 联调推进中"]
    VIS["视觉算法服务<br/>Face / Tongue / Sublingual / General<br/>独立 ToC FastAPI"]
  end
  subgraph Data["采集资料与后续特征层"]
    REC[("会话信息 / 多模态媒体档案")]
    STANDARD["标准化特征 / 医生复核<br/>离线工作持续推进"]
  end
  subgraph Roadmap["后续辅助诊断 · 目标架构"]
    DIAG["诊断子 Agent<br/>候选证候 / 证据缺口 / 动态追问"]
    TOOL["Tool 适配与权限管理<br/>算法服务 / 脉诊 / 专业检索"]
    RAG["知识库 / RAG 与医生经验"]
    REVIEW["医生审核 / 个性化校准"]
  end
  APP["ToC APP / 外部调用方"]

  PAT <--> OAC
  MIC --> OAC
  OAC <-->|"业务回复 / 会话关联"| BACK
  STAFF <-->|"HTTP + WebSocket"| BACK
  BACK -->|"只读 SSE"| PAT
  BACK <--> GRAPH
  GRAPH <--> QWEN
  BACK --> CAM
  CAM --> REC
  BACK --> REC
  APP --> VIS
  PULSE --> PULSAPP
  PULSAPP -.->|"待完整集成"| BACK
  REC -.->|"标准化接入推进"| STANDARD
  GRAPH -.->|"后续子图/路由"| DIAG
  QWEN -.->|"规划 / 推理与结果整合"| DIAG
  STANDARD -.-> DIAG
  DIAG -.-> TOOL
  DIAG -.-> RAG
  DIAG -.-> REVIEW
  TOOL -.-> VIS
  TOOL -.-> PULSAPP

  classDef brain fill:#deefff,stroke:#175caa,stroke-width:3px,color:#123653;
  classDef planned fill:#f7f7f7,stroke:#9098a4,stroke-dasharray:5 4,color:#47515d;
  classDef ongoing fill:#fff3de,stroke:#b9802e,color:#554020;
  class QWEN brain;
  class DIAG,TOOL,RAG,REVIEW planned;
  class PULSAPP,STANDARD ongoing;
```

**Qwen 底座的准确定位**：当前 Qwen 经 SGLang 为 LangGraph 提供问诊语义理解、字段提取和受约束问句生成；PPT 描述的**规划、工具调用、结果整合**是下一阶段 Diagnosis Agent 及业务编排的目标能力，不代表模型当前直接控制全部设备、调用全部算法或输出临床诊断。

**算法 Tool 的集成边界**：Face、Tongue、Sublingual、General 等 ToC 视觉 API 已具备独立部署与调用接口；它们不等于已经接入 LangGraph 的动态 Tool Calling。图中经 Tool 适配层的虚线体现**后续统一调用**，当前工作站仍主要通过自身采集流程及已有设备/服务接口协调。

**脉诊和外设**：工作站面向摄像头、麦克风、音响、补光灯和脉诊仪等多模态设备。脉诊模块具备独立采集基础，正推进通信数据质量、波形呈现和统一会话接入；后续可通过设备适配服务与标注体系联动。

**技术分工**：同机 Electron 双窗口不等于两台远程终端；窗口级操作走 IPC，医助双向控制走 WebSocket，患者状态观察走只读 SSE，图像与视频走独立媒体入口。业务状态由 FastAPI / LangGraph 协调，不由数字人独立决定。

## 02｜数字人语音、问诊与模型调用链

```mermaid
sequenceDiagram
  participant U as 患者
  participant O as OAC：VAD/ASR
  participant B as FastAPI：业务门面
  participant G as LangGraph：问诊子图
  participant Q as SGLang：Qwen
  participant T as OAC：TTS/数字人
  U->>O: 语音回答
  O->>B: 识别文字 + 会话标识
  B->>G: 恢复当前会话 / 注入输入
  G->>Q: 字段提取与有限意图识别
  Q-->>G: 候选结构化结果
  G->>G: 规则检查缺项 / 确定提问目标
  G->>Q: 受约束的自然问句生成
  Q-->>G: 回复文本
  G-->>B: 流式回复或固定引导
  B-->>T: OpenAI 兼容响应
  T-->>U: 语音播报 + 数字人动作
  T-->>B: 播报完成等同步信号
```

这是一轮典型问诊**逻辑路径**：按具体意图，固定提示、知识回复或主动引导可能走不同分支。VAD 判断语音结束，ASR 转文字，LLM 不直接处理全部音频。OAC 的兼容模型客户端连接的是**业务中枢提供的兼容门面**，而非把采集业务写入 OAC 或由数字人直接决定下一采集阶段。

**实际工程难点**：语音结束判断、ASR、非流式 NLU、下一问文本生成、TTS 和播报结束门控可能依次等待；VAD 重分段会造成一段用户回答产生多次请求。项目处理过问句重复、阶段播报与拍照争用的时序问题，也分析过串行调用造成的响应等待。**这里不声称已完成全链路低延迟改造或有可复核的端到端提速数值**。

## 03｜ToB 四诊采集工作站：业务流程与数据归档

```mermaid
flowchart TD
  A["医助准备并发起会话"] --> W["数字人欢迎 + 多轮问诊"]
  W --> D{"必填字段与进展检查"}
  D -->|"仍有缺项"| W
  D -->|"完成"| F["面部图像采集"]
  F --> TT["舌面图像采集"]
  TT --> TB["舌下图像采集"]
  TB --> GAIT["已有步态记录检查"]
  GAIT --> P["脉诊阶段 / 设备适配"]
  P --> CHECK{"采集完整性检查"}
  CHECK -->|"按规则补采"| F
  CHECK -->|"收尾"| R["会话记录 + 媒体归档"]
  W -.-> H["医助接管 / 纠错"]
  F -.-> H
  P -.-> H
  H -.-> W
```

- **已实现的软件基础**：主图串联采集阶段；问诊子图维护字段、意图、缺项与重试；面舌等子图按引导、等待、执行、质量检查组织流程，数据归档按同一会话关联。医助可监督、基础标注和接管。
- **暂停与恢复**：检查点保存 Graph 的业务状态，不能自动重建活动摄像头句柄、OAC 语音连接和全部设备状态。
- **脉诊特别说明**：图中的脉诊阶段包含设备接口和采集流程的衔接设计；独立脉诊采集已有基础，设备数据、波形展示与 ToB 四诊采集工作站的完整闭环仍在联调完善。
- **尚未交付**：标准特征匹配与个性化辅助诊断 Agent 的生产闭环、医学诊断报告与临床效果评估。

## 04｜下一阶段：诊断子 Agent 与 Tool 适配设计

目前的 ToB 四诊采集工作站先形成**会话关联的问诊、面舌图像、视频和人工标注**，后续逐步补入脉诊仪的真实波形特征。对接诊断子 Agent 的最小数据路径可以是：

```mermaid
flowchart LR
  CASE["四诊采集记录"] --> NORM["特征标准化与医生审核<br/>在建"]
  NORM -.-> DIAG["诊断子 Agent<br/>目标"]
  DIAG -.-> CAND["候选证候与证据缺口<br/>目标"]
  CAND -.-> NEXT["补问 / 调用专业 Tool<br/>目标"]
  NEXT -.-> FACE["视觉算法 API<br/>已独立服务化"]
  NEXT -.-> PULSE["脉诊适配层<br/>待集成"]
  NEXT -.-> KB["知识检索 / RAG<br/>规划"]
  NEXT -.-> DOC["医生复核与决策<br/>目标"]
```

这里**不是当前已经运行的自主诊断链路**。未来 Diagnosis Agent 的任务是基于标准化特征判断还需要哪类证据，并建议继续问诊或调用专业工具；由 LangGraph 管理授权、分支和人工审核，不让模型直接控制硬件。

### 脉诊设备接入的数据路径

`脉诊仪 → 采集与波形处理模块 → 设备适配接口 → 工作站会话关联 → 医助端实时显示 → 标注与归档`。

脉诊采集和设备适配已有工程基础，仍需完善数据完整性、波形显示与工作站会话级归档；脉象分析与诊断能力作为后续专业工具扩展。

### 独立 ToC 服务与平台 Tool 的关系

ToC 以 APP / 外部调用者为入口，向 **General、Face、Tongue、Sublingual** 等独立 FastAPI 服务发送请求，各服务维持自身模型生命周期并返回结构化结果，部署上由 Docker Compose 管理。

如果未来作为平台 Tool，先做**统一输入输出适配、超时/错误边界、会话关联和调用权限控制**，再由诊断子 Agent 通过业务编排访问这些接口。**独立 API 的完成，不等于已实现 Agent 的动态工具调用。**

## 05｜为什么不是一个“大模型包办所有功能”

ToB 四诊采集工作站采用**确定性业务骨架 + Qwen 语言能力 + 专业算法/设备服务 + 医助审核**：由 LangGraph 和代码规则控制采集阶段、缺项补问与暂停，大模型负责理解和受约束表达，采集算法/设备独立处理专业数据。后续才考虑在标准特征与医生复核基础上扩展证据缺口驱动的辅助诊断逻辑。

ToC 视觉算法 API 是**另一条产品工程路线**，具备独立服务化部署，不应画成工作站业务的必经调用。详见 [工程案例](engineering-cases.md)。
