# Meshy Symmetric Model — Cursor Agent Skill

一个 [Cursor](https://cursor.com) Agent Skill，通过 [Meshy.ai](https://meshy.ai) API + [Blender](https://www.blender.org) 自动生成**拓扑对称的低模 3D 角色**。

```
参考图片 → Meshy API 生成+减面 → Blender bisect+mirror 拓扑对称 → 纹理压缩 → 导出 GLB
```

## 快速开始

### 1. 环境准备

| 依赖 | 版本 | 说明 |
|------|------|------|
| [Cursor](https://cursor.com) | 最新 | AI IDE |
| [Blender](https://www.blender.org/download/) | 4.0+ | 3D 建模 |
| [Blender MCP Addon](https://github.com/ahujasid/blender-mcp) | 最新 | Blender ↔ Cursor 桥接 |
| [Meshy.ai 账号](https://app.meshy.ai/) | — | 需 API Key（[获取](https://app.meshy.ai/settings/api-keys)） |

### 2. 安装

```bash
git clone https://github.com/silwings1986/meshy-symmetric-model-skill.git
cd meshy-symmetric-model-skill

# 配置 API Key
cp .env.example .env
# 编辑 .env，填入你的 Meshy API Key
```

将 `.cursor/skills/meshy-symmetric-model/` 目录复制到你的项目中，或放到全局目录 `~/.cursor/skills/` 对所有项目生效。

### 3. 启动 Blender MCP

1. 打开 Blender → Edit → Preferences → Add-ons → 启用 blender-mcp
2. 插件面板点击 **Start MCP Server**
3. Cursor 设置中确认 MCP 连接正常

### 4. 使用

在 Cursor 中对 AI 说：

> "用 horseman.png 生成一个对称低模角色，目标 1500 面"

Agent 会自动调用此 Skill 完成全流程。

## 仓库结构

```
├── .cursor/skills/meshy-symmetric-model/
│   └── SKILL.md           # Skill 主体（Agent 读取执行的指令文件）
├── .env.example            # API Key 模板
├── .gitignore
└── README.md
```

## 费用

每次生成约 **30 Meshy credits**（生成 20 + 纹理 10）。[查询余额](https://app.meshy.ai/settings/billing)

## License

MIT
