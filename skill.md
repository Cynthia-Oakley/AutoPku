---
name: autopku
description: AutoPku 主入口 - 自动获取和整理北京大学课程通知，完成作业
---

# AutoPku

> 自动获取和整理北京大学课程通知，完成作业并提交。
> Claude Code 和 Codex 通用入口。

## 前置配置

执行前确保 `.claude/settings.local.json` 包含必要权限：

```json
{
  "permissions": {
    "allow": ["Skill(update-config)", "Bash(*)"],
    "deny": ["Bash(rm:*)", "Bash(rm -rf:*)"],
    "defaultMode": "bypassPermissions"
  },
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## 使用方式

```bash
skill: autopku <任务> [参数]
```

| 任务 | 说明 | 示例 |
|------|------|------|
| `sync` | 同步所有课程通知 | `skill: autopku sync` |
| `do` | 完成指定课程作业 | `skill: autopku do 简明量子力学` |
| `notes` | 撰写课程笔记 | `skill: autopku notes 逻辑导论` |

## 任务详情

### sync - 同步课程通知

为所有当前学期课程创建目录结构和通知摘要。

**引用**: `sub-skills/tasks/sync-notices.md`

**流程**:
1. 检查并安装 pku3b → `tools/pku3b-setup.md`
2. 登录教学网 → `tools/pku3b-setup.md`
3. 获取并解析数据 → `tools/data-parser.md`
4. 并行处理课程 → `runtime/create-agent.md` + `tools/agent-helpers.md`
5. 生成汇总报告

**输出**:
- `test/通知摘要汇总.md`
- `test/{课程}/通知摘要.md`
- `test/{课程}/作业/`（附件）

### do - 完成作业

完成指定课程的作业，渲染后询问用户是否提交。

**引用**: `sub-skills/tasks/do-homework.md`

**流程**:
1. **用户确认**: 列出作业列表 → 选择 → 二次确认
2. **Phase 1**: PDF解析 → `tools/pdf-reader.md`
3. **Phase 2**: 逐题解答 → `runtime/create-agent.md`
4. **Phase 3**: 渲染为PDF → `tools/agent-helpers.md`
5. **询问用户**: 展示完成结果，确认是否提交
6. **Phase 4**: 提交到教学网（用户确认后）→ `tools/pku3b-setup.md`

**输出**:
- `{课程}/作业/{作业}_answer.md`
- `{课程}/作业/{作业}_answer.pdf`
- 教学网提交状态

### notes - 撰写课程笔记

读取课件 PDF，使用 Agent Team 并行撰写精简数学笔记，去除历史/故事等噪声。

**引用**: `sub-skills/tasks/write-notes.md`

**流程**:
1. **用户询问**: 选择课件范围、详细程度、额外选项
2. **并行解析**: 每个 PDF 创建 writer agent → `tools/pdf-reader.md`
3. **内容筛选**: 保留 motivation/定义/定理/证明/结论，去除历史/故事/用例
4. **生成索引**: `notes/README.md`

**笔记特点**:
- 聚焦数学核心：形式化定义、精确定理、关键证明步骤
- 去除噪声：历史背景、哲学家轶事、复杂无关概念解释
- 格式规范：标准 Markdown + LaTeX 公式

**输出**:
- `notes/{lecture_name}.md`（每个课件一个）
- `notes/README.md`（索引和速查）

## 子技能索引

### 运行时 (`sub-skills/runtime/`)
- `_detect.md` - 检测 AI 环境
- `claude-team.md` - Claude Code Agent Team
- `codex-subagent.md` - Codex Native Subagent
- `create-agent.md` - 统一 agent 创建接口

### 工具 (`sub-skills/tools/`)
- `pku3b-setup.md` - pku3b 安装、登录、命令
- `data-parser.md` - 教学网数据解析（ANSI处理）
- `pdf-reader.md` - PDF 读取（PyMuPDF/pdfplumber）
- `agent-helpers.md` - Agent Prompt 模板

### 任务 (`sub-skills/tasks/`)
- `sync-notices.md` - 同步课程通知
- `do-homework.md` - 完成作业（解析→解答→渲染→提交）
- `write-notes.md` - 撰写课程笔记（数学核心内容提取）

## 环境适配

此 skill 自动适配运行环境：

| 环境 | 检测方式 | Agent 机制 |
|------|---------|-----------|
| Claude Code | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | `Agent()` tool + Team Coordination |
| Codex | `CODEX=1` | Native subagents |
| 其他 | 回退 | 串行执行 |

任务 skills 通过 `runtime/create-agent.md` 统一接口创建 agents，无需关心底层实现。

## 踩坑记录

详见 `ignore/archived-skill.md`（原完整 skill 备份）。

关键提示：
- `pku3b init` 需要交互式输入，使用 expect 脚本
- `pku3b a ls -a` 只显示有作业的课程，需用 `pku3b s show` 获取完整列表
- 作业无附件时 `download` 命令返回 Done. 但不创建文件
- 部分课程使用 Canvas/微信群，教学网无记录

## 需要的权限

- Bash 执行（运行 pku3b）
- 文件读写（创建目录和文件）
- Agent 创建（并行处理）
- AskUserQuestion（用户确认）
