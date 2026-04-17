# PR 示例：添加代码审查规则到文档

这是一个示例 Pull Request，用来演示如何使用项目的 PR 模板与代码审查流程。

示例分支：`example/pr-add-code-review`

示例 PR 标题：

```text
docs: add example PR linking code review rules
```

示例 PR 描述（模板化）：

- 变更说明：将 `docs/code-review.md` 添加为代码审查指南并在 README 中链接。
- 关联 issue：无
- 影响范围：文档，仅 docs/ 目录

检查清单：

- [x] 变更说明与目的清晰
- [x] 单元/集成测试无影响
- [x] 静态检查（linter/formatter）通过（文档无需运行）

如何本地创建并提交该 PR（示例命令）：

```bash
cd /home/project/yi/yi-workspace
# 创建并切换到新分支
git checkout -b example/pr-add-code-review

# 添加并提交更改
git add docs/pr-example.md docs/code-review.md README.md
git commit -m "docs: add PR example linking code review rules"

# 推送到远程并在 GitHub 上创建 PR
git push -u origin example/pr-add-code-review
# 然后在 GitHub 上打开 Pull Request，并使用项目的 PR 模板填写说明
```

如果需要，我可以为你自动创建本地分支并提交（不推送），或者生成一个包含实际变更的 patch 文件供你手动应用与推送。
