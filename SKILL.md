---
name: "requirement-drift-auditor"
description: 'Generate a requirement-drift audit report comparing git diff against master/main to find changes that diverge from the stated requirement. Use only when the user explicitly asks for a requirement-drift audit, code drift audit, pre-release/pre-test audit report, checks for 夹带私货/off-scope changes, or asks to generate audit-report.md/audit-report.html; do not trigger merely because code was changed, tested, built, or is ready to test.'
allowed-tools: Bash(git:*) Bash(git分支:*) Bash(mkdir:*)
---

# 需求偏离审计 (Requirement Drift Auditor)

本 Skill 用于在需求开发完成、提测前，以原始需求为基准线，通过分析当前分支与 `master` 分支的 Git Diff，排查并揪出偏离需求的”存疑发散”修改（如隐式重构、未要求的逻辑变更），最终生成本地的 Markdown 和 HTML 审计报告。

## 触发时机
仅当用户**明确要求**执行需求偏离/需求漂移审计时触发，例如：

- “执行需求偏离审计”
- “检查一下有没有夹带私货”
- “生成审计报告 / audit-report.md / audit-report.html”
- “提测前帮我做一次需求漂移检查”
- “对照原始需求审一下这次 diff 有没有超范围修改”

不要因为以下情况自动触发：

- 用户只是要求实现、修改、修 bug、跑测试或构建；
- 用户说“准备提测”但没有要求审计/检查偏离；
- 你自己准备在测试前做验证。

如果用户只要求“测试/构建/验证”，直接测试或构建，不生成审计报告。

## 执行工作流 (严格按顺序执行)

1. **需求确认**：
   - 如果用户在触发本技能时，**没有提供**本次开发的核心需求描述，你必须**先停下来询问用户**：”请问本次开发的核心需求和预期目标是什么？”
   - 只有获取到明确的”原始需求（基准线）”后，才能进入下一步。

2. **获取纯净的逻辑 Diff**：
   - 自动获取当前所在的分支名。
   - 自动在终端执行以下 Git 命令，获取当前分支与 `master` 分支的差异：
     ```bash
     git diff -w -b master...HEAD
     ```
   - *注：`-w -b` 参数极其关键，它会严格过滤掉纯粹的空格、缩进、空行等格式化修改，确保你只聚焦于真实的业务逻辑变更。*
   - 如果项目主分支名为 `main`，请自动替换为 `main...HEAD`。

3. **三向对比与深度分析 (核心分类逻辑)**：
   - 结合【原始需求】与获取到的【Git Diff】，逐个文件分析代码变更。
   - 采用以下标准对每一处逻辑修改进行分类映射：
     - 🟢 **直接响应**：需求明确要求的修改。（**正常行为，无需在最终报告中体现**）
     - 🟡 **必要衍生**：需求中没提，但为了实现需求、跑通代码**必须**附带的修改（如：引入了新的依赖包、修改了接口签名导致调用方同步修改）。（**在报告中简述**）
     - 🔴 **存疑发散 (重点！)**：需求中没提，且非绝对必需的修改。**任何你不确定是否属于需求本身的修改，都必须保守地归类为存疑点。** 包括但不限于：顺手的代码重构、未要求的异常/边界处理、旁路业务逻辑的调整、全局配置的修改等。（**必须在报告中详细罗列！**）

4. **生成并写入本地报告 (Markdown & HTML)**：
   - 在项目根目录使用工具创建并写入 `audit-report.md` 文件。
   - 文件的内容必须严格遵循以下 Markdown 模板：

   ```markdown
   # 需求偏离审计报告 (Requirement Drift Audit Report)

   **审查时间**: [填写当前系统日期]
   **比对分支**: `master` -> `[当前分支]`
   **核心需求**: [简述用户的原始需求]

   ## 1. 🔍 存疑点排查 (重点核对)
   > ⚠️ 以下代码变更未在原始需求中提及，且非实现需求的必要条件，存在“过度发散”风险，请人工严格 Review。
   *(如果没有发现，请写：“✅ 未发现明显的无端发散变更”)*

   - **[文件路径:大致行号]**
     - **变更现象**: [一句话描述改了什么，例如“为原有的 getUser 函数增加了对 null 参数的拦截”]
     - **存疑原因**: [为什么认为它超出了需求范围，例如“需求仅要求修改登录UI，此底层逻辑修改可能导致依赖 null 报错的旧业务逻辑失效”]

   ## 2. 🔗 必要衍生变更清单
   > 以下变更为实现需求所产生的必要副作用，请简单确认。
   - **[文件路径]**: [简述衍生修改内容，如“因新增分页参数，同步更新了接口签名”]

   ## 3. 📝 审计总结
   [总结本次代码变更与原始需求的偏离度，给出风险评级（低/中/高）]
   ```
   - **同时，基于上述 Markdown 内容，生成一份同名的 `audit-report.html` 文件**。
   - HTML 报告需要包含基础的 CSS 样式（如：使用系统无衬线字体，适当的行高、内外边距，并对“存疑点”等重要信息给予醒目的颜色高亮），以确保在浏览器中打开时具备良好的阅读体验。

5. **完成通知**：
   - 报告文件写入完成后，在对话框中通知用户：“审计报告已生成至本地 `audit-report.md` 和 `audit-report.html`，您可以直接在浏览器中打开 HTML 文件获得更好的阅读体验，并请重点核对【存疑点排查】部分。”
   - 如果发现了高风险的存疑发散点，请在对话中向用户简要预警。
