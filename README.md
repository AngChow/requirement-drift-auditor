# Requirement Drift Auditor (需求偏离审计)

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

一个 Claude Code Skill，用于在需求开发完成后、提测前，以原始需求为基准线审计代码变更，排查偏离需求的"存疑发散"修改，生成本地 Markdown 和 HTML 审计报告供人工审核。

## 背景与动机

在使用 AI 辅助编码时，模型可能在完成需求的同时夹带未要求的修改——顺手重构、多余的错误处理、旁路逻辑调整等。这些"发散变更"轻则增加 Review 负担，重则引入线上风险。本 Skill 将变更严格分为三类，帮助工程师快速定位需要人工核对的存疑点。

## 功能特性

- 以原始需求为基准线，对比当前分支与主分支的 Git Diff
- 自动过滤纯格式化修改（空格、缩进、空行），只聚焦逻辑变更
- 三级分类体系：直接响应 / 必要衍生 / 存疑发散
- 生成本地 `audit-report.md` 和 `audit-report.html` 两份报告
- HTML 报告带 CSS 样式，存疑点醒目高亮，浏览器直接打开即可阅读

## 安装

### 方式一：在 Claude Code 中直接安装（推荐）

打开 Claude Code，直接说：

```
帮我安装这个 skill https://github.com/AngChow/requirement-drift-auditor.git
```

Claude 会自动完成克隆和注册。

### 方式二：手动安装

```bash
git clone https://github.com/AngChow/requirement-drift-auditor.git \
  ~/.claude/skills/requirement-drift-auditor
```

## 使用方法

在 Claude Code 中，当开发完成准备提测时，使用以下任一方式触发：

```
/requirement-drift-auditor 本次需求：新增用户导出功能
```

或自然语言触发：

```
开发完成了，帮我审计一下代码有没有夹带私货，需求是新增用户导出功能
```

### 触发关键词

- "开发完成，准备提测"
- "检查一下代码有没有夹带私货"
- "执行审计"
- "生成代码审查报告"

### 前置条件

- 当前工作目录必须是 Git 仓库
- 存在主分支（`master` 或 `main`）
- 已提交代码变更（未提交的变更不会被 `git diff` 捕获）

## 分类标准

Skill 对每处逻辑变更进行三级分类：

| 级别 | 分类 | 定义 | 报告中的处理 |
|------|------|------|-------------|
| 🟢 | 直接响应 | 需求明确要求的修改 | 不在报告中体现 |
| 🟡 | 必要衍生 | 需求未提，但实现需求必须附带的修改 | 简述 |
| 🔴 | 存疑发散 | 需求未提，且非绝对必需的修改 | **详细罗列，重点核对** |

**分类原则**：任何不确定是否属于需求本身的修改，一律保守归类为存疑点。

## 报告格式

生成的 `audit-report.md` 包含三个部分：

### 1. 存疑点排查（重点核对）

列出所有偏离需求的变更，每项包含：
- 文件路径与大致行号
- 变更现象（改了什么）
- 存疑原因（为什么认为超出需求范围）

### 2. 必要衍生变更清单

列出实现需求所产生的必要副作用，供简单确认。

### 3. 审计总结

总结偏离度，给出风险评级（低/中/高）。

## 项目结构

```
requirement-drift-auditor/
├── SKILL.md          # Skill 定义文件（Claude Skill 格式，含触发条件和工作流）
├── README.md         # 本文件
├── LICENSE           # MIT 许可证
└── .gitignore
```

## Git Diff 参数说明

Skill 使用 `git diff -w -b master...HEAD` 获取差异：

| 参数 | 作用 |
|------|------|
| `-w` | 忽略所有空白字符变更 |
| `-b` | 忽略空白字符数量的变更 |
| `master...HEAD` | 三点语法，只显示当前分支相对于 master 的独有变更 |

## 注意事项

1. **Diff 范围**：仅审计已提交的变更，未暂存或未提交的修改不在审计范围内
2. **主分支名**：自动检测 `master` 或 `main`，无需手动指定
3. **报告位置**：报告生成在项目根目录，建议在 `.gitignore` 中添加 `audit-report.*`
4. **纯指令型 Skill**：无需安装任何依赖，不包含脚本文件

## 许可证

[MIT](LICENSE)
