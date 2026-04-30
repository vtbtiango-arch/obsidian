# 中文版

这个目录包含 ScholarSkill 的完整简体中文版本。

如果你准备使用中文版工作流、中文版模板和中文版说明，请从这个目录开始。初始化时建议直接运行中文包里的 `configure.py`，并传入 `--vault-path`，这样会明确安装到你指定的 Obsidian vault，并把模板同步到 `9-Templates/zh-CN/`。

快速开始：

```bash
git clone https://github.com/EESJGong/scholar-skill.git
cd scholar-skill
```

然后在 OpenClaw 中告诉 agent：

```text
请把这个仓库安装成 OpenClaw skill，使用中文版，并配置到我的 Obsidian vault：/你的/Obsidian/Vault
```

从这里开始：

- [SKILL.md](SKILL.md)
- [docs/CONFIGURATION.md](docs/CONFIGURATION.md)
- [docs/DURABLE-INTEGRATION.md](docs/DURABLE-INTEGRATION.md)
