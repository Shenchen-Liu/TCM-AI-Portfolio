<div align="center">

<h1>🩺 中医多模态 AI 系统</h1>

<p><strong>从 PPT 方案到产品原型｜主导 0→1 软件产品落地</strong></p>

<p>以 <strong>ToB 四诊采集工作站</strong> 为核心，整合数字人问诊、双屏协同与多模态采集，并推进 <strong>ToC 视觉算法服务化</strong>。</p>

<p>
  <code>Python / FastAPI</code> ·
  <code>Vue 3 / Electron</code> ·
  <code>Qwen3.5-9B / SGLang</code> ·
  <code>LangGraph</code> ·
  <code>OpenAvatarChat</code> ·
  <code>Docker / Linux</code>
</p>

</div>

---

这份作品集记录了我将中医数字化项目从早期方案推进到可运行原型的过程：从理解实际接诊场景、选择技术路线，到组织模块开发、跨服务联调和交付。重点不是单独开发某个算法，而是让前端、模型、数字人和采集设备在同一流程中配合工作。

**我的职责**：主导软件需求梳理、架构设计、技术选型和开发推进，负责各模块集成、联调与交付。代码实现结合 AI Coding 和团队协作，面舌算法及传感硬件由项目团队共同研发。

## 🎯 01｜项目背景：从方案走向产品

**项目起点**：团队最初在项目方案中提出了纳米传感四诊采集、数字人交互和名医辅助诊断的平台设想。虽然已有面诊、舌诊等算法 Demo 和部分硬件基础，但它们各自运行，还不是一套能让患者完成检查、医助全程操作的产品。

**我负责的任务**：把 PPT 里的平台设想拆解成可开发、可交付的软件产品。没有一开始就追求完整的自动诊断，而是先让四诊信息采集、数字人交互和医生辅助操作在一台工作站上跑通，同时将已有视觉算法改造成可复用的 API 服务。

| 路线 | 目标 | 实际范围 |
| --- | --- | --- |
| **ToB 四诊采集工作站（主线）** | 在实体工作站实现患者引导、医助监控、智能问诊、多模态采集和会话归档 | 双屏数字人交互、多轮问诊、面舌采集、医助复核和数据归档已形成产品原型；脉诊仪接入与波形联动持续完善 |
| **ToC：视觉算法 API 服务化** | 将原有文件式算法 Demo 改造为可复用、可部署的推理服务 | 负责服务接口、模型生命周期和多服务部署的工程化思路与落地 |

产品落地按 **四诊采集 → 特征标准化与医生标注 → 辅助诊断 Agent** 逐步推进。第一阶段优先解决医患交互、设备协同和数据归档，后续再围绕真实采集数据完善标准化与个性化辅助诊断。

## 🛠️ 02｜核心工作：把独立模块整合成系统

| 负责方向 | 具体做法 | 要解决的问题 |
| --- | --- | --- |
| **产品与前端交互** | Electron 单应用双窗口，分别服务患者和医助；Vue 管理两端交互，医助 WebSocket 控制、患者只读 SSE 观察 | 角色分工、跨屏状态一致性和人工接管 |
| **AI 工作流** | 使用 LangGraph 编排受约束多轮问诊、采集子图、状态检查点、暂停/补采；Qwen 负责抽取与自然问句 | 多阶段流程可控、会话数据累计、异常可追踪 |
| **数字人和模型服务** | OpenAvatarChat 接入 VAD/ASR/TTS/数字人链路；FastAPI 连接业务图与 SGLang 本地 Qwen 推理 | 让数字人交互、业务状态和模型推理解耦 |
| **多模态与部署** | 摄像头按会话复用、业务与媒体入口分流、媒体归档；独立服务部署及健康检查 | 连续采集、视频回看和多组件部署维护 |
| **视觉算法工程化** | 将文件路径/重复模型加载方式调整为启动预加载、预热、内存传递和模型实例复用 | 降低重复初始化和磁盘 I/O 对连续 API 调用的影响 |

[整体系统架构、数字人链路、四诊采集流程](docs/architecture.md) · [技术选型理由与 Trade-off](docs/design-decisions.md)

### 🏗️ 整体架构：Qwen 底座、采集 Agent 与专业 Tool

以 **Qwen 智能底座**为核心，通过 **LangGraph 采集 Agent + FastAPI**连接 ToB 四诊采集工作站、独立视觉算法服务与外部传感设备，并向标准化特征、诊断子 Agent 和个性化辅助诊疗扩展。

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

**图示说明**：实线呈现已具备的软件交互与独立服务，虚线连接持续集成或规划的能力。脉诊仪作为专业采集设备接入平台，诊断子 Agent 将进一步整合标准化四诊特征、算法 Tool 与医生经验。

[查看分层架构与数字人调用链](docs/architecture.md) · [技术选型与取舍](docs/design-decisions.md)

## 🚀 03｜交付成果

- **🩺 ToB 四诊采集工作站**：在约三个月的集中开发中，推动核心业务从方案走到可运行的双屏原型。患者端由数字人引导问诊和面舌采集，医助端负责流程控制、采集监控、资料回看和人工接管；脉诊设备接入及波形联动继续完善。
- **⚡ ToC 算法服务化**：把原先依赖本地文件、重复加载权重的舌诊等算法封装为常驻推理 API，调整模型加载、图片传递与部署方式，让独立算法能够接入应用。早期代表性业务调用体验从**约 20 秒降至约 3 秒**（历史体验记录，非标准压测）。
- **📚 特征标准化与标注体系**：基于《证素辨证学》与 GB/T 16751.2—2021 整理中医四诊临床术语，构建按观察维度（Dimension）和参考状态（State）组织的特征标签目录。形成 **110 个候选维度、510 个参考标签、681 条主检索表达**，以及 **5,209 条 GB/T 扩展检索候选**，为医助分类浏览、相似表达检索和结构化标注提供数据基础。
- **🔧 工程问题处理**：在实际联调中先后处理了跨屏流程误触发、媒体预览等待、问诊状态恢复和数字人播报时序等问题，让采集流程更连贯，也让异常更容易定位和处理。

### 🩺 ToB 四诊采集工作站 · 现场与采集流程

<p align="center">
  <img src="assets/%E5%8C%BB%E9%99%A2%E8%AF%95%E7%94%A8.png" alt="四诊工作站在医院的现场试用" width="760" />
</p>
<p align="center"><sub>工作站现场展示：患者引导、医助操作与多模态采集由同一业务流程协同。</sub></p>

**🎥 采集系统实机演示（2× 加速）**

https://github.com/user-attachments/assets/4eb2a0f3-a9b4-4c57-9e02-ff123c23c523

*视频展示数字人交互和四诊采集过程；为方便浏览已加速至 2×。*

### 📱 ToC 算法服务化 · 移动端应用场景

<p align="center">
  <img src="assets/%E5%95%86%E4%B8%9AAPP.jpg" alt="移动端多模态健康评估应用方案与报告界面" width="740" />
</p>
<p align="center"><sub>移动端应用与健康评估界面示意。已完成的工程重点是视觉算法 API 服务化，APP 界面用于说明下游产品形态。</sub></p>

### 📚 特征标准化与标签检索

将临床特征整理为可浏览的**观察维度—参考标签**：例如「寒热」下有“恶风、恶寒、潮热”等状态；疼痛则分别按**部位**（腰痛、肩痛）和**性质**（刺痛、绞痛）组织。标签保留原始描述及来源信息，方便标注时检索和选择。

| 输入或检索表达 | 所属维度 | 关联的候选标签 |
| --- | --- | --- |
| 寒热往来 | 寒热 | 往来寒热 |
| 高热 | 寒热 | 壮热 |
| 声重 | 声音与言语 | 声音重浊 |

结合 Embedding 相似表达召回与 LLM 辅助术语整理，为“医生原话 → 检索候选 → 选择标准标签”的标注流程提供支持。以上示例取自 **v0.3 特征目录的实际关联数据**。

### 🧠 辅助诊断与医生个性化 · 发展方向

<table>
  <tr>
    <th align="center" width="50%">Agent 架构与端到端模型路线</th>
    <th align="center" width="50%">诊断子 Agent 的目标工作流</th>
  </tr>
  <tr>
    <td align="center">
      <img src="assets/Agent%20%E7%B3%BB%E7%BB%9F%E5%AF%B9%E6%AF%94%E7%AB%AF%E5%88%B0%E7%AB%AF%E5%A4%A7%E6%A8%A1%E5%9E%8B.png" alt="Agent 与端到端大模型技术路线对比" width="410" />
    </td>
    <td align="center">
      <img src="assets/%E8%AF%8A%E6%96%ADAgent.png" alt="诊断子 Agent 的证据整合与工具调用方案" width="410" />
    </td>
  </tr>
  <tr>
    <td>由工作流协调模型、工具和医生审核。</td>
    <td>基于标准化特征形成候选判断，决定补问或调用专业工具。</td>
  </tr>
</table>

<p align="center">
  <img src="assets/%E4%B8%AA%E6%80%A7%E5%8C%96%E6%95%B0%E5%AD%97%E5%AD%AA%E7%94%9F%E5%8C%BB%E7%94%9F.png" alt="基于医生特征偏好的个性化数字孪生诊疗方案" width="720" />
</p>
<p align="center"><sub>名医个性化数字孪生方案：共享 Qwen 底座，根据医生经验与特征关注构建差异化辅助策略。</sub></p>

> 以上是项目的后续架构方向；当前已经落地的是受控采集工作流与独立算法服务，诊断子 Agent 和名医个性化能力仍在规划建设中。

## ⚙️ 04｜工程问题与优化

| 遇到的问题 | 如何处理 | 改进效果 / 当前进展 |
| --- | --- | --- |
| **算法响应慢** | 把模型加载移到服务启动阶段，复用权重和图片预处理，减少文件读写 | 代表性业务体验约 20s → 3s，仍待规范化复测 |
| **患者屏重连影响流程** | 拆分流程执行和状态订阅，患者屏只负责观察进度 | 避免订阅重连误触发流程推进 |
| **连续采集时预览卡顿** | 将图片/视频流与业务请求分流，共享采集缓冲 | 缓解浏览器长连接竞争，需继续进行实机回归 |
| **问诊中断后状态异常** | 修正子图暂停、恢复和字段保留逻辑 | 减少恢复时丢失已采信息的问题 |
| **数字人响应等待** | 对齐语音识别、两次模型调用与播报时序，排查重复请求 | 完成链路分析和部分时序调整，整体时延仍待优化 |

[查看工程问题的完整复盘](docs/engineering-cases.md) · [为何选择这些技术](docs/design-decisions.md)

## 🔍 05｜技术详情

- [架构与数据流](docs/architecture.md)：系统分层、数字人语音/模型调用、四诊采集和人工操作。
- [工程案例](docs/engineering-cases.md)：真实现象 → 任务 → 处理方案 → 结果与限制。
- [技术选型](docs/design-decisions.md)：Electron、IPC/WS/SSE、LangGraph、SGLang、OAC、FastAPI、Linux/Docker 的设计理由和代价。

**发展方向**：在现有 ToB 四诊采集工作站和 ToC 专业算法服务基础上，进一步建设特征标准化、诊断子 Agent 与医生个性化辅助决策闭环。
