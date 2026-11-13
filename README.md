# CarrorOS

**给 AI 编码 agent 加一层运行时治理：不让它改自己的规则。**

这是发布仓，只放**能直接用的安装包**与说明。源码仓保持私有。

---

## 它解决什么

你在用 Claude Code / 类似的 agent 写代码，它会在你的仓库里执行命令、改文件。问题是：

```
· 它能改 AGENTS.md、.claude/gates、CI 配置——也就是"管着它自己的那些规则"
· 它说"做完了"，你没法确认它到底改了什么、改的是哪个版本
· 出了问题，找不到是谁（哪个 agent、哪次执行）改的
```

CarrorOS 在这些动作发生**之前**拦一道，并留下记录。

## 装什么

```
.claude/gates/           规则判定（路径归一、写通道识别、fail-closed）
.claude/hooks/           PreToolUse 钩子，接进 Claude Code
.claude/tasks/ .claude/verify/   任务与验收记录
AGENTS.md / CLAUDE.md    工作流约定（可被你的项目覆盖）
```

**不动你的代码、依赖和 .gitignore。**

## 安装

```bash
# 下载后
bash carroros-delivery-1.0.0.sh /path/to/your-project
```

装到已有项目里是设计意图，不是副作用。若目标目录已有 `.claude/`，需要 `--force` 才覆盖。

```bash
bash carroros-delivery-1.0.0.sh /path/to/your-project --force
```

## 装完怎么验

```bash
cd /path/to/your-project
# 1) 钩子生效：试着让 agent 改治理文件，应被拦
echo '{"tool_name":"Bash","tool_input":{"command":"cp /tmp/x .claude/gates/rules.py"}}' \
  | python3 .claude/hooks/pretool-gate.py

# 2) 期望输出含 deny / ASK_USER，并且 target 是归一后的路径
```

## 边界（先说清楚）

```
✓ 拦住常见写通道：重定向 / tee / sed -i / cp / mv / ln / dd / python open(w)
✓ 路径归一：折叠 ./ // ..、解软链、大小写不敏感
✓ fail-closed：解析失败、模块异常都按拒绝

✗ 拦不住 python -c 里用变量拼路径再 open(p,'w')
✗ 拦不住有 shell 权限、且会读 gate 源码找洞的 agent
✗ 不是安全隔离，是防手滑
```

**真要隔离，得把治理文件只读挂载，修改走人工。**

## 版本

| 版本 | 内容 |
|---|---|
| 1.0.0 | 首个发布包：gates + hooks + 任务/验收记录，173 个文件 |

## 许可

MIT
