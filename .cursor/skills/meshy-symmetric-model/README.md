# Meshy Symmetric Model — Cursor Agent Skill

通过 Meshy.ai API 生成对称低模 3D 角色的 Cursor Agent Skill。

AI Agent 会根据参考图片自动完成：生成模型 → 减面 → 对称处理 → 纹理压缩 → 导出。

## 环境要求

| 依赖 | 版本 | 说明 |
|------|------|------|
| [Cursor](https://cursor.com) | 最新 | AI IDE，运行此 Skill 的宿主 |
| [Blender](https://www.blender.org/download/) | 4.0+ | 3D 建模软件 |
| [Blender MCP Addon](https://github.com/ahujasid/blender-mcp) | 最新 | Blender ↔ Cursor 的桥接插件 |
| [Meshy.ai 账号](https://app.meshy.ai/) | — | 需要 API Key 和 credits |

## 安装步骤

### 1. 安装 Blender MCP

1. 在 Blender 中：Edit → Preferences → Add-ons → Install
2. 安装 blender-mcp 插件并启用
3. 在插件面板点击 **Start MCP Server**

### 2. 配置 Cursor MCP

在 Cursor 设置中添加 Blender MCP Server（Settings → MCP），确保连接状态正常。

### 3. 安装 Skill

将 `SKILL.md` 复制到你的 Cursor 项目中：

```
你的项目/
├── .cursor/
│   └── skills/
│       └── meshy-symmetric-model/
│           └── SKILL.md          ← 放这里
├── .env                          ← API Key 配置
├── .env.example                  ← 模板（可选，用于分享）
└── ...
```

也可以放在用户全局目录，对所有项目生效：

```
~/.cursor/skills/meshy-symmetric-model/SKILL.md
```

### 4. 配置 API Key

```bash
# 复制模板
cp .env.example .env

# 编辑 .env，填入你的真实 API Key
# MESHY_API_KEY=msy_your_real_key_here
```

API Key 获取地址：https://app.meshy.ai/settings/api-keys

## 使用方式

在 Cursor 中直接对 AI 说：

- "用 horseman.png 生成一个对称低模角色"
- "调用 Meshy API 图生3D，目标 1500 面"
- "帮我批量生成 T-pose 对称角色模型"

Agent 会自动识别并调用此 Skill。

## 费用说明

每次生成消耗约 **30 Meshy credits**（生成 20 + 纹理 10）。

查询余额：https://app.meshy.ai/settings/billing

## 文件说明

| 文件 | 用途 |
|------|------|
| `SKILL.md` | Skill 主文件，Agent 读取并执行 |
| `.env` | 存放 API Key（**不要提交到 git**） |
| `.env.example` | API Key 模板，可安全分享 |

## 注意事项

- `.env` 文件包含敏感信息，**绝对不要**提交到 git
- 建议在 `.gitignore` 中添加 `.env`
- Meshy API 的 `symmetry_mode: "on"` 仅保证外形对称，拓扑对称由 Blender bisect+mirror 完成
- 减面建议交给 Meshy API（`should_remesh: true`），Blender 的 Decimate 对 AI 模型效果差
