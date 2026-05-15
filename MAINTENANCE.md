# Maintenance

`python-conventions` plugin 的维护手册。涵盖日常改动到发版的完整操作流程。

> **核心原则**：每次有意义改动 → 提交 → bump version → 打 tag → 发 release。光 commit 不打 tag，用户的 `/plugin update` 不会拉到新版。

---

## 仓库基本信息

| 项 | 值 |
|---|---|
| 仓库 | https://github.com/wfj6ccff/python-conventions |
| 默认分支 | `main` |
| 本地路径 | `/Users/66ccff/Downloads/python-skills-perplexity-aligned` |
| 维护工具 | `git`、`gh` CLI |

---

## 标准发版流程（5 步）

每次发版都按这 5 步走，**全部命令都在仓库根目录跑**。

```bash
cd /Users/66ccff/Downloads/python-skills-perplexity-aligned
```

### 步骤 1：改文件

按需改 `skills/*/SKILL.md`、`skills/*/references/*.md` 等。常见改动场景见下方 [常见操作 cookbook](#常见操作-cookbook)。

### 步骤 2：决定新版本号

按 [Semantic Versioning](https://semver.org/lang/zh-CN/)：

| 改动类型 | 版本变化 | 示例 |
|---|---|---|
| 改措辞 / 修笔误 / 调阈值 / 删可选推荐 | **patch**：`0.1.1 → 0.1.2` | 移除 `pydantic-ai` 推荐 |
| 加新 reference / 加新 skill / 加默认参数 | **minor**：`0.1.x → 0.2.0` | 新增 `db-migrations.md` |
| 删 skill / 改 description 路由语义 / 改 plugin name | **major**：`0.x.x → 1.0.0` | 把 `llm` skill 拆成两个 |

> 第一个稳定版本之前都是 `0.x.x`。`1.0.0` 之后 major 才有破坏性含义。

### 步骤 3：同步三处版本号

**必须三处同时改**，否则会出现 plugin.json 与 marketplace.json 不一致：

```bash
# 假设 NEW_VERSION="0.1.2"
NEW_VERSION="0.1.2"

# 三处用 sed 改（macOS BSD sed 用 -i ''）
sed -i '' "s/\"version\": \"[0-9.]*\"/\"version\": \"${NEW_VERSION}\"/" .claude-plugin/plugin.json
sed -i '' "s/\"version\": \"[0-9.]*\"/\"version\": \"${NEW_VERSION}\"/" .claude-plugin/marketplace.json
```

然后**手动**在 `CHANGELOG.md` 顶部加一段：

```markdown
## [0.1.2] - 2026-05-15

### Changed
- 简短描述本次改动

### Removed / Added / Fixed
- ……
```

**校验三处一致**：

```bash
echo "=== 版本一致性检查 ==="
grep '"version"' .claude-plugin/plugin.json
grep '"version"' .claude-plugin/marketplace.json
grep -E "^## " CHANGELOG.md | head -3
```

三处都应该是同一个新版本号。

### 步骤 4：commit + tag + push（一次性）

```bash
NEW_VERSION="0.1.2"
SUMMARY="简短描述本次改动"

git add -A
git status --short          # 检查 staged 文件

git commit -m "v${NEW_VERSION}: ${SUMMARY}

详细说明改了什么、为什么。
可以多行。"

git tag -a "v${NEW_VERSION}" -m "v${NEW_VERSION}: ${SUMMARY}"

git push origin main
git push origin "v${NEW_VERSION}"
```

### 步骤 5：发 GitHub release

```bash
NEW_VERSION="0.1.2"

gh release create "v${NEW_VERSION}" \
  --title "v${NEW_VERSION} — 标题" \
  --notes "## What changed

- 改动点 1
- 改动点 2

## Install / Update

\`\`\`
/plugin marketplace update wfj6ccff
/plugin update python-conventions@wfj6ccff
\`\`\`"
```

完成后访问 `https://github.com/wfj6ccff/python-conventions/releases` 确认新版已是 Latest。

---

## 完整发版脚本（一键）

复制粘贴可用。改 `NEW_VERSION` 和 `SUMMARY` 即可。

```bash
#!/usr/bin/env bash
set -euo pipefail

cd /Users/66ccff/Downloads/python-skills-perplexity-aligned

# === 改这两行 ===
NEW_VERSION="0.1.2"
SUMMARY="简短描述本次改动"
# ==================

# 1. 改三处版本号（CHANGELOG 仍需手动加段落）
sed -i '' "s/\"version\": \"[0-9.]*\"/\"version\": \"${NEW_VERSION}\"/" .claude-plugin/plugin.json
sed -i '' "s/\"version\": \"[0-9.]*\"/\"version\": \"${NEW_VERSION}\"/" .claude-plugin/marketplace.json

echo "⚠️  请手动在 CHANGELOG.md 顶部加 [${NEW_VERSION}] 段落，然后回车继续……"
read -r

# 2. 校验
echo "=== 版本一致性 ==="
grep '"version"' .claude-plugin/plugin.json
grep '"version"' .claude-plugin/marketplace.json
grep -E "^## " CHANGELOG.md | head -3

# 3. commit + tag
git add -A
git commit -m "v${NEW_VERSION}: ${SUMMARY}"
git tag -a "v${NEW_VERSION}" -m "v${NEW_VERSION}: ${SUMMARY}"

# 4. push
git push origin main
git push origin "v${NEW_VERSION}"

# 5. release
gh release create "v${NEW_VERSION}" \
  --title "v${NEW_VERSION} — ${SUMMARY}" \
  --notes "## What changed

见 [CHANGELOG](./CHANGELOG.md#$(echo ${NEW_VERSION} | tr -d .))。

## Update

\`\`\`
/plugin marketplace update wfj6ccff
/plugin update python-conventions@wfj6ccff
\`\`\`"

echo "✅ 发布完成：https://github.com/wfj6ccff/python-conventions/releases/tag/v${NEW_VERSION}"
```

---

## 常见操作 cookbook

### A. 改某个 skill 的 description（路由语义）

⚠️ 这是 **major** 改动，会影响 Claude 是否加载该 skill。

```bash
# 编辑 SKILL.md 顶部 frontmatter 的 description: 行
$EDITOR skills/llm/SKILL.md
```

发版：bump major（`0.x.x → 1.0.0`）或在 0.x 阶段 bump minor。

### B. 加一个新 reference

```bash
# 1. 写新文件
$EDITOR skills/llm/references/new-topic.md

# 2. 在 SKILL.md 的 "Reference loading rules" 段加一行
$EDITOR skills/llm/SKILL.md
```

发版：minor。

### C. 删一段过时推荐（推荐位瘦身）

像 v0.1.1 那种，删 `pydantic-ai` / `langfuse` / `ragas`：

```bash
# 直接编辑相关 references 删掉对应行
# 然后全仓 grep 确认无残留
grep -RIn -E "pydantic-ai|logfire|langfuse|ragas" --include="*.md"
```

发版：patch。

### D. 加一个新 skill

```bash
# 1. 建目录
mkdir -p skills/new-skill/references

# 2. 写 SKILL.md（参考现有任意 SKILL.md 格式）
$EDITOR skills/new-skill/SKILL.md

# 3. 写 references
$EDITOR skills/new-skill/references/xxx.md
```

`SKILL.md` frontmatter 必须有：

```markdown
---
name: new-skill
description: Load when user ...
---
```

发版：minor。

### E. 加一个 slash command

```bash
mkdir -p commands
$EDITOR commands/init-fastapi.md
```

文件本身就是一段 prompt。安装后用 `/python-conventions:init-fastapi` 调用。

需要在 `plugin.json` 里？**不需要**——默认会自动发现 `commands/` 目录。

发版：minor。

### F. 加 hook（自动化）

```bash
mkdir -p hooks
cat > hooks/hooks.json <<'EOF'
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        { "type": "command", "command": "ruff format ${CLAUDE_PROJECT_DIR}" }
      ]
    }
  ]
}
EOF
```

发版：minor（首次加 hook 是新功能）。

---

## 本地验证（push 前）

不想每次改完直接 push，可以先在本机临时挂载验证：

```bash
# 在另一个终端开 Claude Code，挂载本地 plugin
claude --plugin-dir /Users/66ccff/Downloads/python-skills-perplexity-aligned
```

这种方式：

- 不进缓存，直接读源文件
- 改完 `SKILL.md` 重启 Claude Code 就生效
- 完全绕过 marketplace，适合快速迭代

验证 OK 后再走标准发版流程。

---

## 用户侧使用命令速查

发给装这个 plugin 的人（包括你自己）：

### 首次安装

```text
/plugin marketplace add wfj6ccff/python-conventions
/plugin install python-conventions@wfj6ccff
```

### 拿到新版

```text
/plugin marketplace update wfj6ccff
/plugin update python-conventions@wfj6ccff
```

### 卸载

```text
/plugin uninstall python-conventions@wfj6ccff
/plugin marketplace remove wfj6ccff
```

### 查看已装 plugin

```text
/plugin list
```

---

## 故障排除

### 发版后用户说 "/plugin update 没拉到新版"

最常见原因：**没打 tag**。

```bash
# 检查是否有 tag
git ls-remote --tags origin

# 检查 release 是否存在
gh release list --limit 5
```

如果 tag 缺了，补打：

```bash
git tag -a v0.1.2 -m "v0.1.2"
git push origin v0.1.2
gh release create v0.1.2 --title "v0.1.2" --notes "补发"
```

### plugin.json 与 marketplace.json 版本不一致

```bash
grep '"version"' .claude-plugin/plugin.json .claude-plugin/marketplace.json
```

不一致的话，按 `plugin.json` 优先（[官方规则](https://code.claude.com/docs/en/plugins-reference)：plugin.json wins），改 marketplace.json 对齐它，再 commit 一个 patch 版本。

### gh auth 失效（token 过期）

```bash
gh auth status         # 看是否还登录着
gh auth refresh        # 刷新 token
gh auth login          # 实在不行重新走一次浏览器登录
```

### push 被拒（远端有新 commit）

如果你在 GitHub 网页上直接编辑了文件，本地会落后：

```bash
git pull --rebase origin main
# 解决冲突（如果有）
git push origin main
```

### 误打错版本号的 tag

```bash
# 删本地
git tag -d v0.1.x

# 删远端
git push origin :refs/tags/v0.1.x

# 删 release
gh release delete v0.1.x --yes
```

然后重新走步骤 4 + 5。

---

## 回滚发布版本

如果 v0.1.2 有问题，想回滚到 v0.1.1：

### 方案 A：发一个 v0.1.3，内容回到 v0.1.1

推荐这个，符合 semver 哲学：

```bash
# 把 v0.1.1 的内容拉出来覆盖到当前
git checkout v0.1.1 -- skills/

# 改三处版本号到 0.1.3，CHANGELOG 加段落
# 然后走标准发版流程
```

### 方案 B：硬删 v0.1.2 release 和 tag（不推荐）

会让已经更新到 v0.1.2 的用户陷入困惑。除非 v0.1.2 还没人装过。

```bash
gh release delete v0.1.2 --yes
git push origin :refs/tags/v0.1.2
git tag -d v0.1.2
git revert <v0.1.2-commit-sha>
git push origin main
```

---

## gh CLI 速查

| 操作 | 命令 |
|---|---|
| 看仓库基本信息 | `gh repo view wfj6ccff/python-conventions` |
| 在浏览器打开仓库 | `gh repo view --web` |
| 看 releases | `gh release list` |
| 看某个 release | `gh release view v0.1.1` |
| 在浏览器打开 release | `gh release view v0.1.1 --web` |
| 下载某版本源码包 | `gh release download v0.1.1` |
| 看最新 workflow run | `gh run list --limit 5` |
| 看仓库 issues | `gh issue list` |
| 看 auth 状态 | `gh auth status` |

---

## git 速查

| 操作 | 命令 |
|---|---|
| 看本地 vs 远端差异 | `git diff origin/main` |
| 看本地未提交改动 | `git status --short && git diff` |
| 看 tag 列表 | `git tag -l` |
| 看远端 tag | `git ls-remote --tags origin` |
| 看 commit 历史 | `git log --oneline -10` |
| 撤销最后一次 commit（保留改动） | `git reset --soft HEAD~1` |
| 撤销某文件未 commit 的改动 | `git restore <file>` |

---

## 文件改动后必做的检查

任何改动后建议跑一遍：

```bash
# 1. 跨 skill 引用残留扫描（应只剩 skill 内部相对引用）
grep -RIn -E "skills/(api|cli|llm|project|worker|data-pipeline)/" \
  --include="*.md" --include="*.json"

# 2. SKILL.md frontmatter 与目录名一致
for f in skills/*/SKILL.md; do
  dir=$(dirname "$f" | xargs basename)
  name=$(grep "^name:" "$f" | sed 's/name: //')
  [ "$dir" = "$name" ] && echo "✓ $f" || echo "✗ $f → dir=$dir name=$name"
done

# 3. JSON 合法性
python3 -c "import json; json.load(open('.claude-plugin/plugin.json'))" && echo "✓ plugin.json"
python3 -c "import json; json.load(open('.claude-plugin/marketplace.json'))" && echo "✓ marketplace.json"

# 4. 禁用包是否被误推荐
grep -RIn -E "langchain|llamaindex|haystack|guidance|Celery|requests|tenacity" \
  --include="*.md" | grep -vE "(禁用|不要|不引入|不建议|❌|代替|替换|stamina)"
# 输出应只在反模式 / 禁用清单 / 反例题中出现
```

---

## 我下次想做什么

记下来，避免遗忘：

- [ ] 加 `commands/init-fastapi.md` —— `/python-conventions:init-fastapi` 一键生成符合栈的 FastAPI 骨架
- [ ] 加 `commands/init-llm-client.md` —— 一键生成 instructor + Qwen3 客户端
- [ ] 加 `agents/python-reviewer.md` —— 按本栈做 PR review 的 subagent
- [ ] 加 `hooks/hooks.json` —— Edit / Write 后自动 ruff format
- [ ] 写 GitHub Actions 校验 plugin.json / marketplace.json schema

---

## 参考链接

- [Claude Code Plugins 官方文档](https://code.claude.com/docs/en/plugins)
- [Plugin manifest 完整 schema](https://code.claude.com/docs/en/plugins-reference)
- [Plugin marketplace 创建指南](https://code.claude.com/docs/en/plugin-marketplaces)
- [Semantic Versioning](https://semver.org/lang/zh-CN/)
