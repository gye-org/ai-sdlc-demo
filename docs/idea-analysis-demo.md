# Idea 入口自动分析冒烟测试

## 范围

验证 `Issue → Agentic Workflow → Copilot Custom Agent → 原 Issue 评论`，
不涉及业务应用实现、Space 集成、飞书认证或正式 manifest 阶段流转。
原有 `pm_assistant` 需要交互式飞书 OAuth，因此本测试使用独立的
`.github/agents/idea-analyst.agent.md`，不改动原有 Agent。

## 文件

| 文件 | 用途 |
| --- | --- |
| `.github/ISSUE_TEMPLATE/idea.yml` | Idea 表单，自动添加 `stage:idea` |
| `.github/agents/idea-analyst.agent.md` | 非交互式需求分析角色 |
| `.github/workflows/idea-analysis.md` | 事件、角色限制、权限和报告写回 |
| `.github/workflows/idea-analysis.lock.yml` | `gh aw compile` 生成的 Actions 工作流，不手工编辑 |

## 前置条件

1. 仓库启用 Actions，组织策略允许编译产物中固定版本的 Actions。
2. 推理认证。当前配置声明 `copilot-requests: write`，使用 Actions 自带的
   `GITHUB_TOKEN` 推理，费用计入组织；前提是组织有 Copilot 订阅并开启集中计费。
   若组织不满足该前提，改为删除该权限并配置仓库或组织 Secret
   `COPILOT_GITHUB_TOKEN`：使用有 Copilot 权益账号的 fine-grained PAT，授予账户级
   **Copilot Requests: Read**。不要把 Token 写入代码、Issue 或聊天。
3. 仓库存在 `stage:idea` 标签。
4. 人工审核后，将上述配置及编译产物发布到默认分支。
   仅存在于功能分支时，`issues` 事件不会运行它，Issue 表单也不会出现在默认入口。

## 编译

在仓库根目录执行（初始配置使用 gh-aw v1.2.2）：

```powershell
gh extension install github/gh-aw
gh aw compile idea-analysis
```

已安装扩展时无需重复安装。编辑 Agent 或 Workflow 后重新编译，随源文件一起发布
生成的 lock 文件及编译器生成的依赖文件。

## 测试

1. 有仓库写权限的成员通过 Idea 表单创建 Issue，或为已有测试 Issue 添加
   `stage:idea` 标签。只响应创建和添加该标签的事件，不响应普通编辑或评论。
2. 在仓库 Actions 页查看 `Idea Analysis` 运行记录。
3. 确认运行成功，原 Issue 出现以 `<!-- idea-analysis:v1 -->` 标记的报告评论，
   包含需求摘要、MVP、至少三条 Given/When/Then 验收标准、复杂度与风险、待澄清与建议。
4. 移除后重新添加 `stage:idea`，确认不会重复报告。
5. 不带该标签的新 Issue 不应启动 AI 分析；非授权角色不应启动 AI 分析。

如果测试 Issue 在工作流发布前已创建，发布后必须移除并重新添加标签；
GitHub 不会重放此前的创建事件。不要将编译通过等同于云端端到端通过。

## 权限与限制

- 仅允许 `admin`、`maintain`、`write` 角色触发，避免公开仓库任意访客消耗推理额度。
- Agent 只有只读 GitHub 工具；评论由独立的 safe-output 任务写入，最多一条。
- 限制 12 轮与 10 分钟；推理仍会消耗所用账号的额度。
- 不读取 Space；不进行外网竞品验证；不能根据此报告宣称正式立项通过。
- 不自动提交、合并、部署或创建下一阶段任务。

参考：[Custom Agent 导入](https://github.github.com/gh-aw/reference/copilot-custom-agents/)、
[Copilot 配置](https://github.github.com/gh-aw/reference/engines/copilot/)、
[触发事件](https://github.github.com/gh-aw/reference/triggers/)。
