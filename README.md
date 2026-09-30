# ximo-plugin (多 Agent 统一网关中继配置工具)

<div align="center">

![Go Version](https://img.shields.io/badge/Go-%3E%3D%201.22-00ADD8?style=flat-square&logo=go)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20macOS-blue?style=flat-square)
![Architecture](https://img.shields.io/badge/Architecture-Data--Driven%20Specs-brightgreen?style=flat-square)
![UI](https://img.shields.io/badge/WebUI-Embedded%20Offline-orange?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-purple?style=flat-square)

<p align="center">
  <b>把任意 OpenAI / Anthropic 兼容中转站，零侵入无缝接入本机各类 Agent 运行时</b><br>
  单个静态二进制 · 纯数据驱动拓展 · 敏感凭据 0600 原子存储 · 内置离线回环安全 WebUI
</p>

</div>

---

## 💡 核心设计与定位

`ximo-plugin` 是一个完全独立的轻量级 Go 模块（`module github.com/1535273240sch-droid/ximo-plugin`），专注于在本地机器上建立清晰、可解释、零污染的模型接入桥梁：

1. **精准对齐配置形态**：根据不同 Agent 的专属规范，自动将中转站的 `base_url`、`api_key` 与 `model_name` 写入本地配置文件，或生成标准的系统环境变量导出语句；
2. **全流程透明自检**：提供 `detect`（环境感知探测）与 `doctor`（深度健康体检）命令，让所有的配置变更与运行状态 100% 可见、可审计、可回滚；
3. **极简轻量**：**不代理实际网络流量、不篡改上游配额**，仅负责在本地终端建立准确的配置映射。

---

## 🌟 核心特性一览

- 🔌 **9 款开箱即用内置适配器（纯数据驱动）**：
  - `claude-code`、`codex-cli`、`continue`、`cursor`、`aider`、`env-openai`、`env-anthropic`、`openai-compatible-generic`、`ximo-agent`。
- 🧩 **零编译扩展机制**：新增任意私有 Agent 无需编写 Go 代码或重新构建，只需在 `~/.ximo-plugin/specs/` 丢入一份标准化 JSON 规则即可即时生效。
- 🖥️ **本地安全 Web 控制台**：
  - 运行 `ximo-plugin ui` 即可在本地环回地址拉起可视化界面；
  - 页面资源基于 `//go:embed` 打包内置于单个二进制，运行时完全零外部网络请求；
  - API 强制启用基于会话的随机令牌（`X-UI-Token`）防护，仅绑定 `127.0.0.1`。
- 🛡️ **严格的安全与防御性设计**：
  - **凭据原子隔离**：本地存储采用 `0600` 权限原子落盘，所有终端日志与前端界面强制执行字段掩码（如 `gwa_****2c9e`）；
  - **自动安全备份**：任何外部配置文件在修改前均会自动创建 `<path>.bak-<unix>` 备份，支持 `--dry-run` 预览差异，严禁篡改未知配置项。

---

## 🚀 安装与编译

### 环境要求
- **Go 1.22+**（推荐 Go 1.23+）

### 源码编译
```bash
git clone https://github.com/1535273240sch-droid/ximo-plugin.git
cd ximo-plugin

# 1. 运行模块自检与单测
go test ./... -count=1

# 2. 编译当前平台可执行文件
go build -o ximo-plugin ./cmd/ximo-plugin   # Windows 平台生成 ximo-plugin.exe
```

### 多平台交叉编译 (无 CGO 纯静态二进制)
```bash
# Linux amd64
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s -w" -o dist/ximo-plugin-linux-amd64 ./cmd/ximo-plugin

# macOS arm64 (Apple Silicon)
CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 go build -trimpath -ldflags "-s -w" -o dist/ximo-plugin-darwin-arm64 ./cmd/ximo-plugin

# Windows amd64
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -trimpath -ldflags "-s -w" -o dist/ximo-plugin-windows-amd64.exe ./cmd/ximo-plugin
```

---

## 📖 快速上手

标准工作流：`detect (探测) → login (凭据登记) → models (模型同步) → apply --dry-run (预检) → apply --yes (生效) → doctor (体检)`

### 1. 命令行 CLI 交互

```bash
# 1. 检测本机已安装的 Agent 及对应配置路径
ximo-plugin detect

# 2. 绑定中转站网关凭据 (安全落盘至 0600 凭据库)
ximo-plugin login --gateway http://127.0.0.1:8600

# 3. 列出当前网关支持的模型清单
ximo-plugin models --gateway http://127.0.0.1:8600

# 4. 差异预览 (不修改任何文件，仅在控制台打印对比)
ximo-plugin apply --gateway http://127.0.0.1:8600 --all --dry-run

# 5. 确认写入配置 (自动创建备份)
ximo-plugin apply --gateway http://127.0.0.1:8600 --all --yes

# 6. 系统健康自检 (检查连通性与配置完整度)
ximo-plugin doctor --gateway http://127.0.0.1:8600

# 7. 导出环境变量 (适合终端临时会话)
ximo-plugin print-env --gateway http://127.0.0.1:8600 --spec claude-code
```

### 2. 本地图形化控制台 (WebUI)

```bash
# 启动本地控制台并自动打开浏览器 (默认端口 8787，生成一次性认证令牌)
ximo-plugin ui

# 指定特定端口启动
ximo-plugin ui --port 8899 --no-open
```

---

## 🧩 如何添加一个自定义 Agent 规则

只需在 `~/.ximo-plugin/specs/<id>.json` 新建一份规格文件即可：

```json
{
  "id": "my-cli",
  "name": "My Custom Agent",
  "description": "自定义 Agent 配置规则示例",
  "detect": { 
    "binaries": ["my-cli"], 
    "paths": ["~/.my-cli"] 
  },
  "env": { 
    "base_url": "OPENAI_BASE_URL", 
    "api_key": "OPENAI_API_KEY", 
    "style": "export" 
  },
  "restart_note": "配置完成后请重启当前终端窗口以加载环境变量。",
  "protocols": ["openai-chat"]
}
```

验证规则是否生效：
```bash
ximo-plugin adapters list
ximo-plugin detect
```

---

## 📚 详细技术文档

| 文档导航 | 主要内容 |
|:---|:---|
| 📖 [完整技术手册 (docs/README.md)](docs/README.md) | 逐命令实操输出、本地 WebUI 安全拓扑、底层边界说明 |
| ⚡ [一页速查 (docs/QUICKSTART.md)](docs/QUICKSTART.md) | 编译、初始化与常用命令速查手册 |
| ⚙️ [规格规范全表 (specs/README.md)](specs/README.md) | 适配器 JSON Schema 规范与字段详解 |

---

## 📄 开源许可证

本项目基于 [MIT License](LICENSE) 协议开源。
