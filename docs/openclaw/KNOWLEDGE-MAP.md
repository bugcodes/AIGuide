# 🐾 OpenClaw 核心原理与实战 — 完整知识图谱

**课程完成时间：** 2026-05-19  
**总课时：** 36 课  
**笔记文件：** 36 份 md  
**学习方法：** 子 Agent 并行研究 + 文档源码阅读  

---

## 📐 一、架构基石（Module 1）

### Lesson 01：Gateway 架构概述
**文件：** `week1/lesson01.md`

```
┌─────────────────────────────────────────────────────────────┐
│                      Gateway (Daemon)                        │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │WhatsApp│  │Telegram │  │ Discord │  │  Slack  │  ...   │
│  │Baileys │  │ grammY  │  │         │  │         │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
│       └─────────────┴────────────┴─────────────┘             │
│                        Message Bus                            │
│                        ┌──────┐                               │
│                        │Agent │                               │
│                        │Loop  │                               │
│                        └──────┘                               │
│                                                             │
│            ┌────────────────────────┐                       │
│            │  WebSocket API (18789) │                       │
│            └────────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
              │                    │
    ┌─────────┴──┐        ┌───────┴────────┐
    │ CLI / App  │        │  Nodes (手机/PC) │
    └────────────┘        └────────────────┘
```

**核心要点：**
- Gateway 是唯一守护进程，管理所有消息面
- WebSocket 是核心协议（默认端口 18789）
- 单实例控制 WhatsApp（Baileys）
- Canvas Host = `/__openclaw__/canvas/` + `/__openclaw__/a2ui/`

**自我检测：**
1. Gateway 默认端口？ → `127.0.0.1:18789`
2. 哪个组件唯一打开 WhatsApp session？ → Gateway
3. 首帧必须是什么？ → `connect`

---

### Lesson 02：Wire Protocol 与连接生命周期
**文件：** `week1/lesson02.md`

**协议三要素：**
| 要素 | 格式 |
|------|------|
| 请求 | `{type:"req", id, method, params}` |
| 响应 | `{type:"res", id, ok, payload\|error}` |
| 事件 | `{type:"event", event, payload, seq?, stateVersion?}` |

**连接流程：**
```
Client → Gateway: req:connect
Gateway → Client: res (ok) + event:presence + event:tick
Client → Gateway: req:agent
Gateway → Client: res:agent (ack) + event:agent (streaming) + res:agent (final)
```

**安全机制：**
- 设备身份（device identity）+ 配对审批
- 签名验证 `connect.challenge` nonce
- 本地直连可自动审批

---

## ⚙️ 二、Agent Loop 核心引擎（Module 2）

### Lesson 03：Agent Loop 机制
**文件：** `week1/lesson03.md`

**执行流程：**
```
Inbound Message
  → Session 解析
  → Queue（串行化）
  → Agent Run（流式 + 工具）
  → Outbound Reply
```

**Hook 体系：**
| 类别 | Hooks |
|------|-------|
| **Agent Turn** | `before_model_resolve`, `before_prompt_build`, `before_agent_reply`, `agent_end` |
| **Tool** | `before_tool_call`, `after_tool_call`, `tool_result_persist` |
| **Message** | `message_received`, `message_sending`, `message_sent` |
| **Session** | `session_start`, `session_end`, `before_compaction` |

**决策语义：**
- `block: true` → 终端，停止低优先级
- `block: false` → 空操作，不清除已有 block

---

### Lesson 04：消息流程与会话管理
**文件：** `week1/lesson04.md`

**消息路由：**
| 来源 | 行为 |
|------|------|
| DM | 默认共享 session |
| Group | 独立 session per group |
| Cron | 每次全新 session |
| Webhook | 独立 session per hook |

**DM 隔离配置：**
```json
session: {
  dmScope: "per-channel-peer"  // 推荐：channel + sender 隔离
}
```

**Session 存储：**
```
~/.openclaw/agents/<agentId>/sessions/
├── sessions.json          # 元数据
└── <sessionId>.jsonl      # 聊天记录
```

---

### Lesson 05：流式传输与分块
**文件：** `week1/lesson05.md`

**两层流式：**
1. **Block Streaming**：按 block 发送完成文本块（channel messages）
2. **Preview Streaming**：编辑临时草稿消息（Telegram/Discord）

**分块控制：**
```json
agents.defaults: {
  blockStreamingDefault: "on",
  blockStreamingBreak: "text_end",  // 或 "message_end"
  blockStreamingChunk: { minChars: 100, maxChars: 500 },
  humanDelay: "natural"  // 800-2500ms 随机暂停
}
```

**边界语义：**
- `text_end`：每产生一个 block 就发送
- `message_end`：等消息结束，一次性 flush

---

### Lesson 06：上下文窗口与 Compaction
**文件：** `week1/lesson06.md`

**Compaction 流程：**
```
对话接近 limit
  → 旧消息 summarization → 存为 compact entry
  → 新消息保持完整
  → 腾出 token 空间继续
```

**Auto-compaction 触发条件：**
- 手动：`/compact [引导词]`
- 自动：session 接近 context limit
- 自动：收到 context overflow error

**Compaction 前自动内存 flush：**
```json
agents.defaults.compaction.memoryFlush.model: "ollama/qwen3:8b"
```

---

### Lesson 07：Hook 系统与生命周期事件
**文件：** `week1/lesson07.md`

**核心 Hook 决策表：**
| Hook | Terminal 行为 |
|------|-------------|
| `before_tool_call` | `block: true` 终端 |
| `before_install` | `block: true` 终端 |
| `message_sending` | `cancel: true` 终端 |

**Hook 优先级：**
- 数值越高越先执行
- 同优先级按注册顺序

---

## 💾 三、Memory 向量记忆系统（Module 3）

### Lesson 08：Memory 核心架构
**文件：** `week2/lesson08.md`

**三层记忆架构：**
```
┌─────────────────────────────────────────────┐
│              MEMORY.md                       │
│         长期记忆 · 持久化 · 核心事实           │
├─────────────────────────────────────────────┤
│           memory/YYYY-MM-DD.md               │
│         每日笔记 · 自动加载今日/昨日            │
├─────────────────────────────────────────────┤
│              DREAMS.md (可选)                 │
│         梦境日记 · 记忆巩固 · 人工审核          │
└─────────────────────────────────────────────┘
```

**记忆工具：**
- `memory_search`：语义搜索（向量 + BM25 混合）
- `memory_get`：精确读取特定文件/行范围

**Auto Memory Flush：**
- Compaction 前自动触发
- 把对话中的重要事实写入 memory 文件

---

### Lesson 09：Memory Search 混合搜索
**文件：** `week2/lesson09.md`

**检索架构：**
```
Query → [Embedding] → Vector Search
     → [Tokenize] → BM25 Search
            ↓
      Weighted Merge
            ↓
        Top Results
```

**Embedding Providers：**
| Provider | API Key | 备注 |
|----------|---------|------|
| OpenAI | ✅ | 自动检测 |
| Gemini | ✅ | 支持多模态 |
| Ollama | ❌ | 本地 |
| Local | ❌ | GGUF 模型 |

**优化选项：**
- `temporalDecay`：30天半衰期，新笔记权重更高
- `MMR`：去重，保持多样性

---

### Lesson 10：Dreaming 三阶段模型
**文件：** `week2/lesson10.md`

**三阶段：**
| Phase | 目的 | 写 MEMORY.md？ |
|-------|------|--------------|
| Light | 收集 + 排序短期材料 | ❌ |
| Deep | 评分 + 推广 durable candidate | ✅ |
| REM | 提取主题 + 反思信号 | ❌ |

**Deep 排名信号：**
| Signal | Weight |
|--------|--------|
| Relevance | 0.30 |
| Frequency | 0.24 |
| Query Diversity | 0.15 |
| Recency | 0.15 |
| Consolidation | 0.10 |
| Conceptual Richness | 0.06 |

---

### Lesson 11：Session 与消息路由
**文件：** `week2/lesson11.md`

**存储位置：**
- Store: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transcripts: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`

**Reset 策略：**
| 策略 | 触发条件 |
|------|---------|
| Daily reset | 凌晨 4:00（默认） |
| Idle reset | 空闲 N 分钟 |
| Manual | `/new` 或 `/reset` |

---

## 🔀 四、消息路由与多 Agent 协作（Module 4）

### Lesson 12：多 Agent 路由
**文件：** `week3/lesson12.md`

**Binding 匹配优先级：**
1. `peer` match（精确 DM/group/channel id）
2. `parentPeer` match
3. `guildId + roles`
4. `guildId`
5. `teamId`
6. `accountId` match
7. `Channel-level match`
8. **Default agent**

**多 Agent 模式：**
```bash
openclaw agents add coding
openclaw agents add social
```

**路径结构：**
```
~/.openclaw/agents/<agentId>/
├── agent/          # auth-profiles.json
├── sessions/       # session store
└── workspace/     # 文件系统
```

---

### Lesson 13：Delegate 架构
**文件：** `week3/lesson13.md`

**三层能力：**
| Tier | 能力 | 权限级别 |
|------|------|---------|
| Tier 1 | 只读 + 草稿 | Read-only |
| Tier 2 | 代发消息/日历 | Send-on-behalf |
| Tier 3 | 自主行动 | Autonomous |

**硬性约束（必须先配置）：**
- 永远不无审批发外部邮件
- 永远不导出联系人/财务数据
- 永远不执行入站命令（prompt injection 防御）

---

### Lesson 14：Model Failover
**文件：** `week3/lesson14.md`

**Failover 流程：**
```
Auth Profile Rotation
    ↓
Model Fallback Chain
    ↓
persist override (modelOverrideSource: "auto")
    ↓
auto override cleared by /new, /reset
```

**Selection Source 策略：**
| Source | Fallback 允许？ |
|--------|--------------|
| Configured default | ✅ |
| User session override | ❌ |
| Auto fallback override | ✅ |

---

### Lesson 15：Plugin Hooks
**文件：** `week3/lesson15.md`

**Hook 优先级注册：**
```typescript
api.on("before_tool_call", handler, { priority: 50 })
```

**Tool Call Policy：**
- `block: true` → 停止执行
- `block: false` → 无操作
- `requireApproval` → 人工审批

---

## 🎯 五、演练场一：应用实战（Lab 1）

### Lesson 16：System Prompt 体系
**文件：** `week3/lesson16.md`

**Prompt 结构：**
```
Tooling → Execution Bias → Safety → Skills
  → OpenClaw Self-Update → Workspace → Documentation
  → Bootstrap Files → Sandbox → Date/Time → Heartbeats → Runtime
```

**Prompt Modes：**
| Mode | 用于 | 包含内容 |
|------|------|---------|
| `full` | 默认主会话 | 全部 sections |
| `minimal` | Sub-agent | Tooling, Safety, Workspace, Date, Runtime |

---

### Lesson 17：Agent Workspace
**文件：** `week3/lesson17.md`

**Workspace 结构：**
```
<workspace>/
├── AGENTS.md      # 工作区定义
├── SOUL.md        # 人格定义
├── USER.md        # 用户信息
├── MEMORY.md      # 长期记忆
├── HEARTBEAT.md   # 心跳配置
├── memory/        # 每日笔记
├── skills/        # 技能（可选）
└── articles/      # 文章存档
```

---

### Lesson 18：Context 与 Compaction
**文件：** `week3/lesson18.md`

**Token 限制处理：**
1. Auto-compaction（接近 limit 时自动触发）
2. Manual compaction（`/compact`）
3. Memory flush（compaction 前自动保存记忆）

**Overflow 识别：**
- `request_too_large`
- `context length exceeded`
- `input exceeds the maximum number of tokens`

---

### Lesson 19：Skills 系统
**文件：** `week3/lesson19.md`

**Skill 加载优先级：**
```
Workspace skills (> .agents/skills > ~/.agents/skills
  > ~/.openclaw/skills > Bundled skills > extraDirs)
```

**Skill 结构：**
```
skills/<name>/
├── SKILL.md       # YAML frontmatter + 指令
└── (其他资源文件)
```

**Allowlist 配置：**
```json
agents.defaults.skills: ["github", "weather"]
agents.list: [{ id: "docs", skills: ["docs-search"] }]  // 替换，非合并
```

---

## 🔌 六、插件与扩展生态（Module 5）

### Lesson 20：Skills 系统深度
**文件：** `week4/lesson20.md`

**核心概念：**
- Skill = 一组打包的指令（SKILL.md）
- 按需加载（不需要每次都注入）
- 覆盖机制：同名校内，高优先级胜出

---

### Lesson 21：ACP Agents
**文件：** `week4/lesson21.md`

**ACP（Agent Communication Protocol）：**
- Subagent 之间的通信协议
- 支持 `sessions_spawn` 创建子任务
- 结果通过 `sessions_send` 回传

---

### Lesson 22：子 Agent 系统
**文件：** `week4/lesson22.md`

**Subagent 创建：**
```typescript
sessions_spawn({
  task: "具体任务",
  mode: "run",
  runtime: "subagent"
})
```

**隔离模式：**
- `context: "isolated"`：干净子会话（默认）
- `context: "fork"`：复制当前 transcript

---

### Lesson 23：消息路由与 Channel 集成
**文件：** `week4/lesson23.md`

**Channel 支持矩阵：**
| Channel | Block Streaming | Preview | 特殊能力 |
|---------|----------------|---------|---------|
| Telegram | ✅ | ✅ | 编辑草稿 |
| Discord | ✅ | ✅ | 可编辑消息 |
| Slack | ✅ | ✅ | 原生传输 |
| WhatsApp | ⚠️ | ❌ | 基础支持 |

---

## 🛡️ 七、故障转移与高可用（Module 6）

### Lesson 24：Retry Policy
**文件：** `week5/lesson24.md`

**默认配置：**
```json
{
  attempts: 3,
  maxDelayMs: 30000,
  jitter: 0.1
}
```

**Provider 特定：**
| Provider | Min Delay |
|----------|-----------|
| Telegram | 400ms |
| Discord | 500ms |

**x-should-retry: false 注入：**
- 当 `retry-after` > 60s 时
- 让 SDK 立即 surface error
- 触发 model failover

---

### Lesson 25：Sandbox 模式与范围
**文件：** `week5/lesson25.md`

**Mode：**
| Mode | 行为 |
|------|------|
| `off` | 不沙箱化 |
| `non-main` | 非 main session 沙箱化（默认） |
| `all` | 所有 session 沙箱化 |

**Scope：**
| Scope | 行为 |
|-------|------|
| `agent` | 每个 agent 一个容器（默认） |
| `session` | 每个 session 一个容器 |
| `shared` | 所有沙箱 session 共享一个容器 |

**Backend：**
| Backend | 说明 |
|---------|------|
| `docker` | 本地容器（默认） |
| `ssh` | 远程 SSH |
| `openshell` | OpenShell 管理 |

---

### Lesson 26：Workspace Access 与后端
**文件：** `week5/lesson26.md`

**Workspace Access：**
| Mode | 可见性 |
|------|-------|
| `none` | 隔离沙箱目录（默认） |
| `ro` | 只读挂载 `/agent` |
| `rw` | 读写挂载 `/workspace` |

**OpenShell Mirror vs Remote：**
| Mode |Canonical| 同步 |
|------|---------|------|
| `mirror` | Local | 每次 exec 前后同步 |
| `remote` | Remote | 初始化后不自动同步 |

---

### Lesson 27：沙箱安全深层机制
**文件：** `week5/lesson27.md`

**风险矩阵：**
| 风险 | 防御机制 |
|------|---------|
| DooD 路径问题 | 配置 host 绝对路径 + identical volume map |
| 路径遍历 | 沙箱重定向，禁止 `../` |
| 权限提升 | elevated exec 独立 escape path |

---

## 🛠️ 八、演练场二：Plugin 开发实战（Lab 2）

### Lesson 28：Plugin 概述
**文件：** `week4/lesson28.md`

**三类 Plugin：**
| 类型 | 功能 |
|------|------|
| Channel | 接入新消息平台（WhatsApp/Telegram/Discord） |
| Provider | 接入新 AI 模型（OpenAI/Anthropic/Ollama） |
| Tool | 扩展工具能力（browser/file/exec） |

---

### Lesson 29：Plugin Manifest
**文件：** `week4/lesson29.md`

**Manifest 核心字段：**
```typescript
{
  id: string,
  name: string,
  version: string,
  contracts: {
    tools?: { name: string, schema: object }[]
  },
  configSchema?: object
}
```

---

### Lesson 30：Plugin Hooks 守卫语义
**文件：** `week4/lesson30.md`

**守卫决策表（重述）：**
| Hook | Terminal Signal | 效果 |
|------|----------------|------|
| `before_tool_call` | `block: true` | 停止工具执行 |
| `before_install` | `block: true` | 停止安装 |
| `message_sending` | `cancel: true` | 取消发送 |

---

### Lesson 31：Plugin 快速开始
**文件：** `week4/lesson31.md`

**最小 Plugin 模板：**
```typescript
import { definePluginEntry } from "openclaw/plugin-sdk/plugin-entry"

export default definePluginEntry({
  id: "my-plugin",
  name: "My Plugin",
  register(api) {
    // 注册 hooks / tools / channels
  }
})
```

---

### Lesson 32：Beta 测试流程
**文件：** `week4/lesson32.md`

**Beta 检查清单：**
- [ ] 功能测试覆盖所有主要路径
- [ ] 错误处理测试
- [ ] 边界条件测试
- [ ] 与其他 Plugin 的交互测试

---

## 🔒 九、演练场三：安全实战（Lab 3）

### Lesson 33：沙箱安全配置
**文件：** `week5/lesson33.md`

**最小安全配置：**
```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        scope: "agent",
        backend: "docker"
      }
    }
  }
}
```

**OpenShell 生产配置：**
```json
{
  plugins: {
    entries: {
      openshell: {
        enabled: true,
        config: {
          mode: "remote",
          remoteWorkspaceDir: "/sandbox"
        }
      }
    }
  }
}
```

---

### Lesson 34：Plugin 调试
**文件：** `week5/lesson34.md`

**调试命令：**
```bash
openclaw plugins list
openclaw plugins validate <plugin-id>
openclaw sandbox status
```

---

### Lesson 35：隔离与 Bind 防护
**文件：** `week5/lesson35.md`

**路径验证深层机制：**
- 沙箱重定向：`/workspace` → `~/.openclaw/sandboxes/<id>/`
- 禁止路径遍历：`..` 检测
- Bind mount 白名单：仅允许配置的 host 路径

---

### Lesson 36：工具策略与沙箱交互
**文件：** `week5/lesson36.md`

**执行顺序：**
```
工具调用请求
  → Hook: before_tool_call (优先级链)
  → Sandbox Policy Check
  → Tool Execution
  → Hook: after_tool_call
  → Result 写回 transcript
```

**Elevated Escape Path：**
- `tools.elevated` 配置的工具有独立 escape path
- 绕过沙箱，直接在 host 执行
- 仅用于 Gateway 自身需要的操作

---

## 🎯 十、自我测试题库

### 架构相关（Lesson 01-02）
1. Gateway 默认 WebSocket 端口？ → `127.0.0.1:18789`
2. Wire Protocol 首帧必须是什么？ → `connect`
3. Canvas Host 两个路径？ → `/__openclaw__/canvas/` + `/__openclaw__/a2ui/`

### Agent Loop（Lesson 03-07）
4. `before_tool_call` 返回 `block: true` 会怎样？ → 终端执行，停止低优先级
5. Auto-compaction 前自动触发什么？ → Memory flush
6. Block streaming 的 `text_end` vs `message_end`？ → 实时发送 vs 结束后 flush
7. Hook 优先级默认注册顺序？ → 高优先级先执行；同优先级按注册顺序

### Memory（Lesson 08-11）
8. 三层记忆架构？ → MEMORY.md + memory/YYYY-MM-DD + DREAMS.md
9. `memory_search` 使用哪两种检索？ → Vector search + BM25 keyword search
10. Dreaming 三个 Phase？ → Light（不写）+ Deep（写MEMORY）+ REM（不写）
11. Deep ranking 权重最高的信号？ → Relevance（0.30）

### 路由与协作（Lesson 12-15）
12. Binding 匹配优先级第一位？ → peer exact match
13. 多 Agent 路径？ → `~/.openclaw/agents/<agentId>/`
14. Tier 3 Delegate 的核心要求？ → 自主行动 + 硬性约束先行

### 插件生态（Lesson 20-23）
15. Skill 加载优先级最高？ → Workspace skills
16. ACP 是什么？ → Agent Communication Protocol
17. Subagent 隔离模式默认？ → `context: "isolated"`

### 故障转移（Lesson 24-27）
18. Retry 默认 attempts？ → 3
19. Telegram min delay？ → 400ms
20. Sandbox mode 三种？ → off / non-main / all
21. `workspaceAccess: "rw"` 的挂载点？ → `/workspace`

### 安全实战（Lesson 33-36）
22. DooD 约束的核心要求？ → host 绝对路径 + identical volume map
23. Elevated exec escape path？ → `gateway`（或 `node`）
24. 路径遍历防护？ → `..` 检测 + 沙箱重定向

---

## 📊 十一、总结

**36 课核心地图：**

```
OpenClaw Agent Builder 知识图谱

[架构层] Gateway ─ WebSocket ─ 消息面（Channel）
    │
[引擎层] Agent Loop ─ Hooks ─ Session ─ Compaction
    │
[记忆层] MEMORY.md ─ memory/ ─ Dreaming ─ Search
    │
[扩展层] Skills ─ Plugins ─ Provider ─ Channel
    │
[安全层] Sandbox ─ Retry ─ Failover ─ 工具策略
    │
[协作层] Multi-Agent ─ Delegate ─ Subagent ─ ACP
```

**成为 Agent Builder 的关键能力：**
1. ✅ 理解 Gateway 架构和消息流
2. ✅ 掌握 Agent Loop 和 Hook 机制
3. ✅ 熟练运用 Memory 系统
4. ✅ 开发 Plugin 和 Skill
5. ✅ 保障安全（Sandbox + 策略）

---

*最后更新：2026-05-19 12:00*  
*课程来源：《OpenClaw 核心原理与实战》36 课大纲*