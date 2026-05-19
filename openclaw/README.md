# 🤖 OpenClaw 核心原理与实战

[![Stars](https://img.shields.io/github/stars/adongwanai/aiguide?style=social)](https://github.com/adongwanai/aiguide)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Agent- blue?style=flat-square)](https://github.com/openclaw/openclaw)

> **VibeCoding 时代，人人都能成为 Agent Builder**  
> 36 课从理论到实战，系统掌握 OpenClaw 核心原理

---

## 📖 课程概览

| 属性 | 值 |
|------|-----|
| 总课时 | 36 课 |
| 理论模块 | 6 个（模块一～六） |
| 实战演练场 | 3 个（Lab 1～3） |
| 预计学习时间 | 6 周（每天 2 小时） |

---

## 🗺️ 学习路径

```
Week 1: 模块一（架构）+ 模块二（Agent Loop）
Week 2: 模块三（Memory 向量记忆系统）
Week 3: 模块四（消息路由）+ 演练场一（应用实战）
Week 4: 模块五（插件生态）+ 演练场二（开发实战）
Week 5: 模块六（故障转移）+ 演练场三（安全实战）
Week 6: 总复盘 + 知识图谱输出
```

---

## 📚 课程目录

### 模块一：OpenClaw 架构
| 课时 | 标题 | 文件 |
|------|------|------|
| L01 | OpenClaw 系统架构全景 | [01-Architecture/L01.md](./01-Architecture/L01.md) |
| L02 | Local-First 隐私哲学与数据主权 | [01-Architecture/L02.md](./01-Architecture/L02.md) |
| L03 | 统一消息模型与多平台适配 | [01-Architecture/L03.md](./01-Architecture/L03.md) |

### 模块二：Agent Loop 核心引擎
| 课时 | 标题 | 文件 |
|------|------|------|
| L04 | 三层架构与执行流程总览 | [02-AgentLoop/L04.md](./02-AgentLoop/L04.md) |
| L05 | 核心重试循环与七重容错策略 | [02-AgentLoop/L05.md](./02-AgentLoop/L05.md) |
| L06 | 单次 LLM 尝试的完整流程 | [02-AgentLoop/L06.md](./02-AgentLoop/L06.md) |
| L07 | 事件驱动的流式处理与状态管理 | [02-AgentLoop/L07.md](./02-AgentLoop/L07.md) |

### 模块三：Memory 向量记忆系统
| 课时 | 标题 | 文件 |
|------|------|------|
| L08 | 四层记忆架构设计 | [03-Memory/L08.md](./03-Memory/L08.md) |
| L09 | 混合搜索、MMR 去重与时间衰减 | [03-Memory/L09.md](./03-Memory/L09.md) |
| L10 | Embedding 存储与优化 | [03-Memory/L10.md](./03-Memory/L10.md) |
| L11 | Agent 集成与数据流全景 | [03-Memory/L11.md](./03-Memory/L11.md) |

### 模块四：消息路由与多 Agent 协作
| 课时 | 标题 | 文件 |
|------|------|------|
| L12 | 消息全链路与回复生成管线 | [04-Routing/L12.md](./04-Routing/L12.md) |
| L13 | 七层路由匹配引擎 | [04-Routing/L13.md](./04-Routing/L13.md) |
| L14 | Session Key 构建与会话隔离策略 | [04-Routing/L14.md](./04-Routing/L14.md) |
| L15 | 多 Agent 隔离与 Subagent 动态协作 | [04-Routing/L15.md](./04-Routing/L15.md) |

### 演练场一：OpenClaw 应用实战
| 课时 | 标题 | 文件 |
|------|------|------|
| L16 | 环境搭建与基础配置 | [04-Routing/L16.md](./04-Routing/L16.md) |
| L17 | 飞书/钉钉等机器人开发 | [04-Routing/L17.md](./04-Routing/L17.md) |
| L18 | 金融投研助手构建 | [04-Routing/L18.md](./04-Routing/L18.md) |
| L19 | 智能客服系统搭建 | [04-Routing/L19.md](./04-Routing/L19.md) |

### 模块五：插件与扩展生态
| 课时 | 标题 | 文件 |
|------|------|------|
| L20 | 双层扩展体系与 Plugin 全貌 | [05-Plugins/L20.md](./05-Plugins/L20.md) |
| L21 | Hook 系统深入 | [05-Plugins/L21.md](./05-Plugins/L21.md) |
| L22 | Provider、Skill 与 Channel 实战 | [05-Plugins/L22.md](./05-Plugins/L22.md) |
| L23 | Plugin SDK 分层与生态工程化 | [05-Plugins/L23.md](./05-Plugins/L23.md) |

### 模块六：故障转移与高可用
| 课时 | 标题 | 文件 |
|------|------|------|
| L24 | 五层递进式上下文防护 | [06-Security/L24.md](./06-Security/L24.md) |
| L25 | 三级故障转移链设计 | [06-Security/L25.md](./06-Security/L25.md) |
| L26 | 冷却期与指数退避 | [06-Security/L26.md](./06-Security/L26.md) |
| L27 | 从防护到实战 | [06-Security/L27.md](./06-Security/L27.md) |

### 演练场二：OpenClaw 开发实战
| 课时 | 标题 | 文件 |
|------|------|------|
| L28 | 消息生命周期追踪实战 | [Labs/L28.md](./Labs/L28.md) |
| L29 | 天气查询 Skill 开发实战 | [Labs/L29.md](./Labs/L29.md) |
| L30 | 数据库连接池 Plugin 开发 | [Labs/L30.md](./Labs/L30.md) |
| L31 | Token 成本优化开发 | [Labs/L31.md](./Labs/L31.md) |
| L32 | 多 Agent 协作系统构建 | [Labs/L32.md](./Labs/L32.md) |

### 演练场三：OpenClaw 安全实战
| 课时 | 标题 | 文件 |
|------|------|------|
| L33 | 威胁模型与安全基线 | [06-Security/L33.md](./06-Security/L33.md) |
| L34 | 工具策略管道与权限分级 | [06-Security/L34.md](./06-Security/L34.md) |
| L35 | 循环检测与注入防御 | [06-Security/L35.md](./06-Security/L35.md) |
| L36 | Gateway、沙箱与安全审计 | [06-Security/L36.md](./06-Security/L36.md) |

---

## 🚀 快速开始

### 1. 安装 OpenClaw
```bash
npm install -g openclaw
openclaw gateway start
```

### 2. 学习路线
```bash
# 第一周：架构 + Agent Loop
openclaw/learn --course openclaw --week 1

# 查看架构文档
cat openclaw/01-Architecture/L01.md
```

### 3. 实战演练
```bash
# 进入演练场
cd openclaw/Labs

# 跟着教程做
cat L28.md  # 消息生命周期追踪
```

---

## 📂 目录结构

```
openclaw/
├── README.md              ← 本课程入口
├── 01-Architecture/       ← 模块一：OpenClaw 架构（L01-L03）
├── 02-AgentLoop/         ← 模块二：Agent Loop（L04-L07）
├── 03-Memory/            ← 模块三：Memory（L08-L11）
├── 04-Routing/           ← 模块四：路由 + 演练场一（L12-L19）
├── 05-Plugins/           ← 模块五：插件生态（L20-L23）
├── 06-Security/          ← 模块六：安全 + 演练场三（L24-L36）
└── Labs/                 ← 演练场二：开发实战（L28-L32）
```

---

## 🎯 核心产出

学完本课程后，你将掌握：
- ✅ OpenClaw 架构设计（Local-First、高内聚低耦合）
- ✅ Agent Loop 执行引擎（Hook 系统、消息流）
- ✅ Memory 向量系统（RAG、混合搜索、Dreaming）
- ✅ 多 Agent 协作（路由、隔离、Subagent）
- ✅ 插件开发（Skill、Plugin、Channel）
- ✅ 安全部署（Sandbox、Retry、Failover）

---

## 🔗 相关资源

- [OpenClaw 官方文档](https://docs.openclaw.ai)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [AIGuide 主仓库](https://github.com/adongwanai/aiguide)

---

*最后更新：2026-05-19*  
*欢迎 ⭐ Star，支持本课程！*
