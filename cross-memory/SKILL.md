---
name: cross-memory
description: >-
  跨工作空间全局记忆检索与写入。当用户提到"查记忆"、"我记得"、"之前说过"、"我的偏好"、
  "你了解我什么"、"你知道我的情况吗"、"查一下记忆"、"跨工作空间"、"记忆中心"、"全局记忆"时触发。
  也适用于需要跨工作空间上下文（如投资空间需要健康信息配合决策），或对话开始时需要建立用户画像理解的场景。
  任何涉及用户个人信息、财务状况、健康状况、工作背景的对话前都应先加载此 Skill。
---

# cross-memory — 跨工作空间全局记忆检索

从全局记忆中心读取用户的完整个人上下文，任何工作空间均可使用。

## 前置条件

本 Skill 依赖以下基础设施（由 Dream Skill 或用户手动创建）：

1. **全局记忆中心**：`${HOME}/.workbuddy/memory/MEMORY.md` — 主数据源，包含用户的所有跨空间精炼记忆
2. **Dream Skill**（推荐）：自动每日蒸馏日报 → MEMORY.md，保持记忆新鲜
3. 如果 MEMORY.md 不存在，引导用户使用 Dream Skill 进行首次蒸馏初始化

## 核心文件路径（动态发现）

```
全局记忆中心（主数据源）
├── MEMORY.md          → ${HOME}/.workbuddy/memory/MEMORY.md
├── 日报目录           → ${HOME}/.workbuddy/memory/YYYY-MM-DD.md
├── 永久档案           → ${HOME}/.workbuddy/memory/dream-vault/ledger.md
├── 永久档案索引       → ${HOME}/.workbuddy/memory/dream-vault/ledger-index.json
└── 蒸馏日志           → ${HOME}/.workbuddy/memory/dream-vault/meta/review-YYYY-MM.md
```

> **路径说明**：`${HOME}` 是用户主目录（Windows: `C:\Users\<username>`，macOS/Linux: `~`）。
> 全局记忆中心推荐放置在用户级 `.workbuddy/memory/` 目录下，这样所有工作空间都能访问。

## 使用流程

### 第一步：发现记忆文件位置

执行以下命令发现全局记忆中心：

```bash
# Windows
Get-ChildItem -Path "$env:USERPROFILE" -Filter "MEMORY.md" -Recurse -Depth 5 -ErrorAction SilentlyContinue | Where-Object { $_.FullName -match "\.workbuddy[\\/]memory[\\/]" } | Select-Object -ExpandProperty FullName

# macOS/Linux
find ~ -name "MEMORY.md" -path "*/.workbuddy/memory/*" -maxdepth 6 2>/dev/null
```

如果找到多个，优先选择 `${HOME}/.workbuddy/memory/MEMORY.md`（用户级全局中心）。

如果找不到，检查 MEMORY.md 是否在当前工作空间的 `.workbuddy/memory/` 下。

### 第二步：读取全局记忆（必做）

使用 `read_file` 读取找到的 `MEMORY.md`，获取用户的完整上下文。

MEMORY.md 标准结构（如果存在）：
- **当前状态**：工作、健康、财务、投资操作待办、房产、AI 工具等
- **稳定认知**：个人画像、工作风格、技术偏好、兴趣、投资风格等
- **关系与背景**：团队、家族史、公司背景
- **跨工作空间记忆地图**：所有工作空间的路径和记忆焦点（如果有的话）
- **Dream**：蒸馏状态

### 第三步：扫描子工作空间（可选）

如果需要更详细的信息，扫描用户的其他工作空间：

```bash
# Windows — 找出所有包含 MEMORY.md 的工作空间
Get-ChildItem -Path "$env:USERPROFILE" -Filter "MEMORY.md" -Recurse -Depth 5 -ErrorAction SilentlyContinue | Where-Object { $_.FullName -match "\.workbuddy[\\/]memory[\\/]" } | Select-Object -ExpandProperty FullName

# macOS/Linux
find ~ -name "MEMORY.md" -path "*/.workbuddy/memory/*" -maxdepth 6 2>/dev/null
```

根据 MEMORY.md 中的"跨工作空间记忆地图"区块，按需读取对应工作空间的详细记忆。

### 第四步：写入记忆（可选）

如果当前对话产生了重要的新信息：
- **即时写入**（用户纠正/重要决策）→ 直接更新 MEMORY.md 对应区块
- **等待蒸馏**（日常信息）→ 写入当前工作空间的日报 `YYYY-MM-DD.md`
- 如果当前不在记忆中心工作空间，可跨工作空间写入（使用绝对路径）

## 使用原则

1. **读取优先**：任何涉及用户个人决策的对话，都应先读取全局记忆
2. **隐私保护**：记忆内容仅在对话中内部使用，不对外暴露
3. **按需深度**：MEMORY.md 已包含精华摘要，通常足够；只在需要详细数据时才深入读取子空间
4. **增量更新**：发现新信息时，追加到日报即可，不要频繁覆盖 MEMORY.md
5. **跨域关联**：注意跨领域的信息关联（如健康影响投资决策、工作影响时间安排）
6. **容错处理**：如果文件不存在或路径无效，优雅降级而非报错

## 初始化指南（给新用户的快速设置）

如果用户是首次使用此 Skill，引导他们完成以下步骤：

1. **选择记忆中心位置**：推荐 `${HOME}/.workbuddy/memory/MEMORY.md`
2. **创建目录**：`mkdir -p ${HOME}/.workbuddy/memory`
3. **初始化 MEMORY.md**：创建空白 MEMORY.md，填入基本信息（工作、健康、财务、兴趣等）
4. **安装 Dream Skill**（可选但推荐）：实现自动每日蒸馏
5. **配置跨工作空间索引**：在 MEMORY.md 中记录其他工作空间的路径和焦点

## MEMORY.md 模板

```markdown
# 全局记忆中心

## 当前状态

**工作**
- （填写工作信息）

**健康**
- （填写健康状况）

**财务**
- （填写财务概况）

## 稳定认知

**个人画像**
- （填写个人基本信息）

**工作风格**
- （填写工作偏好）

**技术偏好**
- （填写技术栈和工具偏好）

**兴趣**
- （填写兴趣爱好）

## 关系与背景
- （填写团队、家族、公司等背景）

## 跨工作空间记忆地图

| 工作空间 | 路径 | 记忆焦点 |
|---------|------|---------|
| 全局记忆 | ${HOME}/.workbuddy/memory/ | 综合记忆中心 |
| （添加更多） | | |

## Dream
- （蒸馏状态，由 Dream Skill 自动维护）
```
