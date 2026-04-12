---
name: autopku-task-write-notes
description: 读取课程slides，使用Agent Team撰写精简数学笔记，去除噪声内容
---

# 任务：撰写课程笔记

## 前置询问（必须）

在执行前，使用 `AskUserQuestion` 询问用户：

```python
AskUserQuestion({
    "questions": [
        {
            "question": "选择要撰写笔记的课件：",
            "options": [
                {"label": "全部课件", "value": "all"},
                {"label": "指定范围", "value": "range"}
            ],
            "multiSelect": False
        },
        {
            "question": "笔记详细程度：",
            "options": [
                {"label": "精简（只保留核心定义和定理）", "value": "minimal"},
                {"label": "标准（包含证明思路）", "value": "standard"},
                {"label": "详细（完整推导过程）", "value": "detailed"}
            ],
            "multiSelect": False
        },
        {
            "question": "额外要求（多选）：",
            "options": [
                {"label": "添加LaTeX公式编号", "value": "numbered_eq"},
                {"label": "添加概念之间的关联图", "value": "concept_map"},
                {"label": "添加例题（如有）", "value": "examples"},
                {"label": "生成Anki卡片", "value": "anki"}
            ],
            "multiSelect": True
        }
    ]
})
```

## 执行流程

1. **发现课件**: 扫描 `lectures/` 目录，列出所有 PDF
2. **用户确认**: 询问笔记范围和详细程度
3. **并行处理**: 为每个 PDF 创建 agent，引用 `pdf-reader` 解析
4. **汇总索引**: 生成 `notes/README.md` 索引文件

## Agent 工作流

### Coordinator

```
你是 Coordinator，协调笔记撰写任务。

输入目录：{lectures_dir}
输出目录：{notes_dir}

任务列表：{pdf_list}

执行：
1. 为每个 PDF 并行创建 writer agent
2. 收集所有 agent 完成报告
3. 生成 notes/README.md 索引

返回：
- 处理了多少个 PDF
- 生成了多少个笔记文件
- 索引文件路径
```

### Writer Agent（每个 PDF 一个）

```
你是笔记撰写专家，从课件中提取数学核心内容。

输入：{pdf_path}
输出：{notes_dir}/{lecture_name}.md

## 引用工具
引用: `sub-skills/tools/pdf-reader.md`

使用 PyMuPDF 读取 PDF：
```python
import fitz
doc = fitz.open("{pdf_path}")
text = "\n\n".join([page.get_text() for page in doc])
doc.close()
```

## 内容筛选原则（重要）

### ✅ 保留内容
- **Motivation**: 为什么要研究这个问题？核心问题是什么？
- **定义**: 形式化定义、符号表示
- **定理/命题**: 精确陈述，编号
- **证明**: 关键证明步骤、核心技巧
- **结论**: 主要结果、推论
- **技术工具**: 关键引理、构造方法

### ❌ 去除内容
- **历史背景**: 谁发明的、发展历程
- **故事/轶事**: 麻雀、哲学家轶事等
- **日常用例**: 用自然语言解释的例子（除非是必要的直觉）
- **复杂无关概念**: 用更复杂的概念解释简单概念
- **重复性内容**: 多处出现的相同解释
- **装饰性语言**: "让我们来看看"、"有趣的是"等

## 笔记格式

```markdown
# {Lecture 标题}

## 核心问题/Motivation
- 本节要解决的中心问题
- 与前文的关系（如有）

## 定义

### 定义 X.X （概念名）
**陈述**: 形式化定义

**符号**: $...$

## 定理与命题

### 定理 X.X （定理名）
**陈述**: 精确数学陈述

**证明**:
1. 关键步骤...
2. 核心技巧...
3. ...

**直观**（可选，一句话）: ...

## 技术工具/引理

### 引理 X.X
...

## 结论
- 本节主要结果总结
- 关键公式/事实

## 记号速查
| 符号 | 含义 |
|-----|------|
| $...$ | ... |
```

## 约束
- 笔记必须可渲染（标准 Markdown）
- 数学公式使用 LaTeX（$...$ 行内，$$...$$ 行间）
- 保留原始课件的结构层次（章节编号）
- 如某节无数学内容（纯故事/历史），标注"本节为导言/背景，略"

返回：
- 处理页数
- 提取的定义数、定理数
- 输出文件路径
- 备注（如有难以处理的内容）
```

## 输出文件

### 单个笔记文件
`notes/{lecture_name}.md`

### 索引文件
`notes/README.md`

```markdown
# 逻辑导论 笔记索引

生成时间: {timestamp}

## 课程概览
{课程简介}

## 章节索引

| 序号 | 标题 | 核心内容 | 文件 |
|-----|------|---------|------|
| 1 | 导言 | ... | [01.导言.md](01.导言.md) |
| 2 | 一只麻雀... | ... | [02.一只麻雀...](...) |
| ... | ... | ... | ... |

## 关键概念图谱
{概念之间的依赖关系}

## 公式速查
{重要公式列表}
```

## 使用示例

```bash
skill: autopku notes 逻辑导论
```

或在其他目录使用：

```bash
skill: autopku notes /path/to/lectures /path/to/notes
```
