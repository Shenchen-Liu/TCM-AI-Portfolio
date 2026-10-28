<div align="center">

<h1>🩺 中医多模态 AI 系统</h1>

<p><strong>主导软件产品从 PPT 方案到双屏工作站的 0→1 落地</strong></p>

<p>Qwen3.5-9B + LangGraph 驱动问诊采集，OpenAvatarChat 负责数字人交互，FastAPI 整合业务流程与专业视觉算法服务。</p>

<p>
  <code>Python / FastAPI</code> ·
  <code>Vue 3 / Electron</code> ·
  <code>Qwen / SGLang</code> ·
  <code>LangGraph</code> ·
  <code>OpenAvatarChat</code> ·
  <code>Docker / Linux</code>
</p>

</div>

---

## 🎯 01｜项目概况与个人职责

项目最初具备面诊、舌诊等独立算法与硬件基础，但缺少面向真实接诊场景的统一软件产品。我负责软件需求拆解、系统架构、技术选型、开发统筹与跨模块联调；与团队协作，在约三个月的集中开发中推动 **ToB 四诊采集工作站原型**落地，并推进 **ToC 视觉算法 API 服务化**。

**个人工作重点**：设计患者/医助双屏业务流程，集成数字人、Qwen 与 LangGraph 问诊采集链路；推进视觉算法服务化、跨服务数据通信、状态恢复及 Linux/Docker 部署。面舌算法和传感硬件由项目团队共同研发。

## 🛠️ 02｜核心技术与系统架构

| 工作方向 | 关键实现 |
| --- | --- |
| **双屏交互与会话控制** | Vue 3 + Electron 双窗口；医助 WebSocket 控制、患者 SSE 只读订阅，IPC 处理本机窗口操作 |
| **AI 问诊与数字人** | LangGraph 管理多轮问诊、阶段路由与检查点；Qwen 负责信息提取和问句生成；OpenAvatarChat 集成语音及数字人 |
| **视觉算法工程化** | FastAPI 常驻推理服务、模型预加载与实例复用、图片内存传递，Docker 统一管理多个算法服务 |
| **采集与部署** | 摄像头会话复用、媒体与业务入口分离、采集资料归档及 Linux 多组件联调 |

### 🏗️ 整体系统架构

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
    PULSE["脉诊仪<br/>Linux 设备已适配"]
    PULSAPP["独立真机采集工具<br/>串口采集 / DAT 保存<br/>离线波形回放已验证"]
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
  PULSAPP -.->|"真实数据接入中"| BACK
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
  classDef adapted fill:#e5f5ec,stroke:#277852,stroke-width:2px,color:#194d35;
  class QWEN brain;
  class DIAG,TOOL,RAG,REVIEW planned;
  class STANDARD ongoing;
  class PULSE,PULSAPP adapted;
```

实线表示已具备的组件交互，虚线表示接入中或后续能力。详细调用链见 [系统架构与数据流](docs/architecture.md)。

## 🚀 03｜交付成果

### 🩺 ToB 四诊采集工作站

形成可运行的**患者端 + 医助端双屏原型**：数字人引导多轮问诊、面舌采集；医助端完成流程控制、采集监控、资料回看和人工接管。脉诊仪已完成 Linux 真机适配与独立采集验证，正在衔接工作站会话流程。

<p align="center">
  <img src="assets/%E5%8C%BB%E9%99%A2%E8%AF%95%E7%94%A8.png" alt="四诊工作站医院试用现场" width="760" />
</p>
<p align="center"><sub>医院试用现场 · 双屏工作站与采集环境</sub></p>

**采集系统演示（2× 加速）**

https://github.com/user-attachments/assets/4eb2a0f3-a9b4-4c57-9e02-ff123c23c523

### ⚡ ToC 视觉算法服务化

将舌诊等文件式算法整理为可复用的推理 API，通过模型常驻、预热和内存数据流减少重复加载及文件 I/O。**代表性业务调用体验由约 20 秒缩短到约 3 秒**（开发阶段记录，非标准压测）。

<p align="center">
  <img src="assets/%E5%95%86%E4%B8%9AAPP.jpg" alt="移动端健康评估应用场景与报告界面示意" width="740" />
</p>
<p align="center"><sub>移动端应用场景示意 · 视觉算法 API 的下游应用方向</sub></p>

### 📚 临床特征标准化与标签检索

基于《证素辨证学》与 GB/T 16751.2—2021 整理中医临床特征，构建 **110 个候选观察维度、510 个参考标签、681 条主检索表达**，另整理 **5,209 条扩展检索候选**。按“维度—状态”组织标签，例如「寒热」对应恶风、恶寒、潮热；疼痛按部位、性质分别归类。

| 检索表达 | 维度 | 关联的候选标签 |
| --- | --- | --- |
| 寒热往来 | 寒热 | 往来寒热 |
| 高热 | 寒热 | 壮热 |
| 声重 | 声音与言语 | 声音重浊 |

结合 Embedding 候选召回与 LLM 辅助术语整理，服务于医助的分类浏览、候选检索和结构化标注。

<details>
<summary><strong>🧠 查看辅助诊断 Agent 与名医数字孪生方案</strong></summary>

以下图片展示后续辅助诊断平台的设计方向，不代表现阶段已上线的自动诊断功能。

<table>
  <tr>
    <td align="center" width="50%"><img src="assets/Agent%20%E7%B3%BB%E7%BB%9F%E5%AF%B9%E6%AF%94%E7%AB%AF%E5%88%B0%E7%AB%AF%E5%A4%A7%E6%A8%A1%E5%9E%8B.png" alt="Agent 与端到端模型技术路线比较" width="410" /></td>
    <td align="center" width="50%"><img src="assets/%E8%AF%8A%E6%96%ADAgent.png" alt="诊断 Agent 目标工作流" width="410" /></td>
  </tr>
</table>

<p align="center">
  <img src="assets/%E4%B8%AA%E6%80%A7%E5%8C%96%E6%95%B0%E5%AD%97%E5%AD%AA%E7%94%9F%E5%8C%BB%E7%94%9F.png" alt="名医数字孪生个性化方案" width="720" />
</p>

</details>

## ⚙️ 04｜工程问题与优化

| 实际问题 | 处理方式 |
| --- | --- |
| **双屏重连导致流程误推进** | 将患者 SSE 订阅改为只读观察，业务推进与状态订阅解耦 |
| **媒体预览卡顿** | 分离媒体与业务请求入口，按会话复用摄像头与帧缓冲 |
| **问诊中断后状态异常** | 修正 LangGraph 子图的暂停、检查点及恢复逻辑，保留已采集信息 |
| **数字人响应等待** | 拆解 VAD、ASR、LLM、TTS 调用链并修正阶段播报时序，持续优化交互体验 |

[查看真实问题、代码路径与技术取舍](docs/engineering-cases.md)

## 🔍 05｜技术详情

- [系统架构与数据流](docs/architecture.md)：整体架构、数字人调用链及采集流程
- [工程实践案例](docs/engineering-cases.md)：算法服务化、双屏通信、媒体资源与问诊状态恢复
- [技术选型与设计理由](docs/design-decisions.md)：Electron、LangGraph、SGLang、OpenAvatarChat、FastAPI 与 Docker

> 本项目展示软件产品和工程落地经历；医疗诊断 Agent、名医个性化能力属于后续研发方向。
