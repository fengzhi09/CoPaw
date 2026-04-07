---
name: meetings
description: Use this skill when user asks to create Agent meetings, list Agent meetings, view Agent meeting details, or access Agent meeting documents (goals/records/summary/reasons). This is for CoPaw multi-Agent collaboration meetings, not user's personal/company meetings. | 当用户要求创建Agent会议、查询Agent会议列表、查看会议详情或文档时使用。本skill专用于CoPaw多Agent协作会议，与用户其他公司会议记录区分。
metadata: { "builtin_skill_version": "1.2", "copaw": { "emoji": "📋" } }
---

# Agent Meetings（CoPaw 多 Agent 协作会议）

## 什么时候用

只有当用户**明确要求**操作 **CoPaw Agent 会议**（多 Agent 协作会议）时，才使用本 skill。

> ⚠️ 这是 CoPaw 多 Agent 协作系统，不是用户的个人/公司会议。

### 应该使用
- 用户要求创建 Agent 临时会议或例会
- 用户要求查看 Agent 会议列表或状态
- 用户要求查看 Agent 会议目标/发言记录/会议纪要/推理过程
- 用户要求启动/停止/重启 Agent 会议

### 不应使用
- 只是普通聊天回复
- 与 CoPaw Agent 协作无关的会议请求

---

## 常用操作

### UI 方式（推荐）

通过前端页面操作：
- **会议管理**：`/meetings` 页面 - 创建、查看、启动、停止会议
- **Agent 配置**：`/sacp-agents` 页面 - 配置和管理 SACP Agent

### API 方式

```bash
# 查询会议列表
GET /meetings

# 查看会议详情
GET /meetings/{meeting_id}

# 创建临时会议（自动启动）
POST /meetings
{
  "meeting_name": "技术方案评审会",
  "meeting_type": "TEMPORARY",
  "topic": {
    "title": "微服务架构方案评审",
    "description": "评审新系统架构设计方案",
    "context": "当前系统已运行5年，需要升级改造。"
  },
  "participants": [
    {"id": "host_001", "name": "张工", "roles": ["HOST"], "intent": "主持会议"},
    {"id": "reporter_001", "name": "架构师A", "roles": ["REPORTER"], "intent": "介绍方案"}
  ],
  "host_id": "host_001",
  "decider_id": "host_001",
  "rounds": ["raw", "reverse"]
}

# 创建例会（需手动启动）
POST /meetings
{
  "meeting_name": "周例会",
  "meeting_type": "REGULAR",
  ...
}

# 启动/停止/重启会议
POST /meetings/{meeting_id}/start
POST /meetings/{meeting_id}/stop
POST /meetings/{meeting_id}/restart

# 查看会议文档
GET /meetings/{meeting_id}/goals    # 会议目标
GET /meetings/{meeting_id}/records  # 发言记录
GET /meetings/{meeting_id}/summary  # 会议纪要
GET /meetings/{meeting_id}/reasons  # 推理过程 (JSON)
```

---

## 核心概念

### 会议类型

| 类型 | 说明 | 自动启动 |
|------|------|----------|
| `TEMPORARY` | 临时会议，一次性讨论 | ✅ 创建后自动启动 |
| `REGULAR` | 例会，周期性会议 | ❌ 需手动启动 |

### 会议状态

| 状态 | 说明 |
|------|------|
| `CREATED` | 已创建 |
| `INITIALIZED` | 已初始化 |
| `RUNNING` | 进行中 |
| `COMPLETED` | 已完成 |
| `STOPPED` | 已停止 |
| `FAILED` | 失败 |

### 会议阶段

会议完整流程经历以下阶段：

```
BACKGROUND → OPENING → ROUND → DECISION → SUMMARY
```

| 阶段 | 说明 |
|------|------|
| `BACKGROUND` | 会前准备，初始化文档 |
| `OPENING` | 开场，主持人开场白 |
| `ROUND` | 轮次发言，汇报人轮流发言 |
| `DECISION` | 决策，决策人综合发言 |
| `SUMMARY` | 总结，生成会议纪要 |

### 角色类型

| 角色 | 说明 | 发言阶段 |
|------|------|----------|
| `HOST` | 主持人，负责开场和总结 | OPENING, SUMMARY |
| `REPORTER` | 汇报人，轮流发言 | ROUND |
| `DECIDER` | 决策人，最终决策 | DECISION |

### 发言顺序模式

| 模式 | 说明 |
|------|------|
| `raw` | 正序发言 (A → B → C) |
| `reverse` | 反序发言 (C → B → A) |
| `alphabet` | 按字母顺序 |
| `random` | 随机顺序 |

---

## 四份文档

| 文档 | 说明 | 格式 | 生成者 |
|------|------|------|--------|
| `goals` | 会议目标/议程 | Markdown | 系统（从 topic 构建） |
| `records` | 发言记录 | Markdown | Agent 发言 |
| `summary` | 会议纪要 | Markdown | 系统（生成） |
| `reasons` | 推理过程 (Chain of Thought) | JSON | 系统 |

---

## 最小工作流

### 创建临时会议

```
1. UI: 打开 /meetings 页面 → 点击「创建会议」→ 填写表单 → 选择 TEMPORARY 类型
   或
   API: POST /meetings { "meeting_type": "TEMPORARY", ... }
2. 系统自动启动会议（后台异步执行）
3. 等待会议完成 (状态变为 COMPLETED/STOPPED/FAILED)
4. 查看: GET /meetings/{id}/summary
```

### 创建例会

```
1. UI: 打开 /meetings 页面 → 点击「创建会议」→ 填写表单 → 选择 REGULAR 类型
   或
   API: POST /meetings { "meeting_type": "REGULAR", ... }
2. 手动启动: POST /meetings/{id}/start
3. 等待会议完成
4. 查看: GET /meetings/{id}/summary
```

### 查看完整会议信息

```
1. GET /meetings/{id}             # 查看会议详情和状态
2. GET /meetings/{id}/goals      # 查看会议目标
3. GET /meetings/{id}/records     # 查看发言记录
4. GET /meetings/{id}/summary     # 查看会议纪要
5. GET /meetings/{id}/reasons     # 查看推理过程
```

---

## 常见错误

### 错误 1：例会创建后不手动启动

例会不会自动启动，创建后需执行：
- UI: 在会议列表点击「启动」按钮
- API: `POST /meetings/{id}/start`

### 错误 2：会议未完成就查看 summary

`summary` 只在会议 `COMPLETED` 状态后才有内容。查看前确认会议状态。

### 错误 3：临时会议重复启动

临时会议创建时已自动启动，再次调用 start 会报错。

### 错误 4：会议运行中编辑配置

会议状态为 `RUNNING` 或 `INITIALIZED` 时，除状态更新外，其他配置不可编辑。

---

## 数据存储位置

| 数据 | 路径 |
|------|------|
| 会议元数据 | `WORKING_DIR/meetings/meta/{meeting_id}.json` |
| 会议文档 | `WORKING_DIR/meetings/{folder}/` |
| 全局索引 | `WORKING_DIR/meetings/index.md` |
| SACP Agent 配置 | `~/.copaw/sacp_agents.json` |

---

## 相关文档

- [DESIGN.md](../../../../meetings/DESIGN.md) - 详细架构设计
- [README.md](../../../../meetings/README.md) - 模块使用说明
