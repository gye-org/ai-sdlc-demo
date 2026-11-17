---
name: Idea Analysis
description: "验证 Idea Issue 自动触发 Custom Agent，并将需求分析写回原 Issue。"
on:
  issues:
    types: [opened, labeled]
  roles: [admin, maintain, write]
if: >-
  contains(github.event.issue.labels.*.name, 'stage:idea') &&
  (github.event.action == 'opened' || github.event.label.name == 'stage:idea')
engine:
  id: copilot
  agent: idea-analyst
imports:
  - .github/agents/idea-analyst.agent.md
permissions:
  contents: read
  issues: read
  copilot-requests: write
tools:
  github:
    toolsets: [issues]
safe-outputs:
  add-comment:
    max: 1
timeout-minutes: 10
max-turns: 12
---

# Idea 入口冒烟测试

仅分析当前事件触发的 Issue #${{ github.event.issue.number }}。
使用 GitHub 只读工具获取该 Issue 的标题、正文和已有评论，
不要扫描其他 Issue 或读取与本次需求无关的仓库内容。

如果已有评论包含 `<!-- idea-analysis:v1 -->`，使用 noop 说明已有分析，不重复评论。
否则严格按照 `idea-analyst` Custom Agent 输出规范，
使用 `add_comment` safe output 将一条完整需求分析报告写回当前 Issue。

不要执行实现、部署、正式阶段流转或文件修改。
读取 Issue 失败或写回工具不可用时，明确报告失败，不得声称已完成分析。
