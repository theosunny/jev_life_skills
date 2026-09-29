---
name: jev-boss-cli
description: Use jev-boss-cli for ALL BOSS 直聘 operations — searching jobs, viewing recommendations, managing applications, chatting with recruiters, and batch greeting. Invoke whenever the user requests any job search or recruitment platform interaction on BOSS 直聘.
author: theosunny
version: "0.3.7"
tags:
  - boss
  - zhipin
  - boss直聘
  - job-search
  - recruitment
  - cli
---

# jev-boss-cli — BOSS 直聘 CLI Tool（fork with login fix）

**Binary:** `jevboss`
**Credentials:** browser cookies (auto-extracted from 10+ browsers) or QR code login (`--qrcode`)

> 本工具是 [jackwener/boss-cli](https://github.com/jackwener/boss-cli)（Apache-2.0）的 fork，保留全部原有能力，并修复了登录态校验问题。详见下方「登录修复说明」。

## 登录修复说明（本 fork 的关键改动）

原版 `kabi-boss-cli` 把 `__zp_stoken__` 列入 `REQUIRED_COOKIES`，但 `__zp_stoken__` 是 zhipin.com 网页 JS 在客户端生成的**反爬令牌**，服务端永远不会通过 `Set-Cookie` 下发。这导致两个问题：

1. **二维码登录拿不到 `__zp_stoken__`** → `load_credential()` 判定缺少关键 Cookie，会**主动删除**已保存的凭证，搜索接口也返回 `code:37 您的环境存在异常`。
2. **浏览器 Cookie 登录虽然包含 `__zp_stoken__`，但也被 `has_required_cookies` 误判**。

本 fork 把 `__zp_stoken__` 从 `REQUIRED_COOKIES` 中移除，登录态校验只依赖真正的会话 Cookie（`wt2` / `wbg` / `zp_at`）。

**结论 / 推荐登录方式**：在浏览器登录 www.zhipin.com 后，用浏览器 Cookie 提取即可开箱即用（该方式天然带 `__zp_stoken__`）：

```bash
jevboss login --cookie-source chrome   # 推荐：能拿到完整登录态（含 __zp_stoken__）
```

二维码登录（`jevboss login --qrcode`）现在也能保存会话并用于 recommend/chat/applied 等接口；但 `search` 接口因平台反爬，仍建议用上面的浏览器 Cookie 方式。

## Setup

```bash
# 从本仓库安装（需要 Python 3.10+）
pip install jev-boss-cli
# 或本地源码安装：
#   cd boss-cli && pip install -e .
```

## Authentication

**IMPORTANT FOR AGENTS**: 执行任何命令前，先检查是否已登录，不要假设 Cookie 已配置。

### Step 0: 检查是否已认证

```bash
jevboss status --json 2>/dev/null | jq -r '.authenticated' | grep -q true && echo "AUTH_OK" || echo "AUTH_NEEDED"
```

`AUTH_OK` 则跳过登录；`AUTH_NEEDED` 则进入 Step 1。

### Step 1: 引导用户认证

确保用户已在任意支持的浏览器（Chrome / Firefox / Edge / Brave / Arc / Safari 等）登录 zhipin.com，然后：

```bash
jevboss login                          # 自动检测有有效 Cookie 的浏览器
jevboss login --cookie-source chrome   # 显式指定浏览器（推荐）
jevboss login --qrcode                 # 二维码登录（用 Boss 直聘 APP 扫码）
```

验证：

```bash
jevboss status
jevboss me --json | jq '.data.name'
```

### Step 2: 常见认证问题

| 现象 | 处理 |
|------|------|
| `未登录` | 运行 `jevboss login --cookie-source chrome` |
| `code:37 您的环境存在异常` | 未携带有效登录态（缺 `__zp_stoken__`），改用浏览器 Cookie 登录 |
| Rate limited (code=9) | 内置自动冷却退避，等待后重试 |
| API 超时 | 检查网络后重试 |

## Agent Defaults

所有机器可读输出使用 [SCHEMA.md](./SCHEMA.md) 描述的信封格式，数据在 `.data` 下。

- 非 TTY stdout → 自动 YAML
- `--json` / `--yaml` → 显式格式
- Rich 输出 → **stderr**（管道安全：`jevboss search X --json | jq .data`）

## Command Reference

### Search & Browse

| Command | Description | Example |
|---------|-------------|---------|
| `jevboss search <keyword>` | 按关键词搜索职位（支持筛选） | `jevboss search "golang" --city 杭州 --salary 20-30K` |
| `jevboss show <index>` | 查看上一次搜索的第 N 条 | `jevboss show 3` |
| `jevboss detail <securityId>` | 查看职位完整详情 | `jevboss detail abc123 --json` |
| `jevboss export <keyword>` | 导出搜索结果为 CSV/JSON | `jevboss export "Python" -n 50 -o jobs.csv` |
| `jevboss recommend` | 个性化推荐职位 | `jevboss recommend -p 2 --json` |
| `jevboss history` | 浏览历史 | `jevboss history --json` |
| `jevboss cities` | 支持的城市列表 | `jevboss cities` |

### Personal Center

| Command | Description | Example |
|---------|-------------|---------|
| `jevboss me` | 查看个人资料 | `jevboss me --json` |
| `jevboss applied` | 已投递职位 | `jevboss applied -p 1 --json` |
| `jevboss interviews` | 面试邀请 | `jevboss interviews --json` |
| `jevboss chat` | 沟通过的 HR 列表 | `jevboss chat --json` |

### Actions

| Command | Description | Example |
|---------|-------------|---------|
| `jevboss greet <securityId>` | 向 HR 打招呼 / 投递 | `jevboss greet abc123 --json` |
| `jevboss batch-greet <keyword>` | 批量打招呼 | `jevboss batch-greet "Python" --city 杭州 -n 5` |
| `jevboss batch-greet <keyword> --dry-run` | 预览（不发送） | `jevboss batch-greet "golang" --dry-run` |

### Account

| Command | Description |
|---------|-------------|
| `jevboss login` | 从浏览器提取 Cookie（自动检测，失败回退二维码） |
| `jevboss login --cookie-source <browser>` | 从指定浏览器提取 |
| `jevboss login --qrcode` | 仅二维码登录（终端输出二维码） |
| `jevboss status` | 检查登录状态（显示 Cookie 名称） |
| `jevboss logout` | 清除已保存凭证 |

## Search Filter Options

| Filter | Flag | Values |
|--------|------|--------|
| City | `--city` | 北京, 上海, 杭州, 深圳 等（用 `jevboss cities` 看全量） |
| Salary | `--salary` | 3K以下, 3-5K, 5-10K, 10-15K, 15-20K, 20-30K, 30-50K, 50K以上 |
| Experience | `--exp` | 不限, 在校/应届, 1年以内, 1-3年, 3-5年, 5-10年, 10年以上 |
| Degree | `--degree` | 不限, 大专, 本科, 硕士, 博士 |
| Industry | `--industry` | 互联网, 电子商务, 游戏, 人工智能, 金融, 教育培训, 医疗健康 等 |
| Company Scale | `--scale` | 0-20人, 20-99人, 100-499人, 500-999人, 1000-9999人, 10000人以上 |
| Funding Stage | `--stage` | 未融资, 天使轮, A轮, B轮, C轮, D轮及以上, 已上市, 不需要融资 |
| Job Type | `--job-type` | 全职, 兼职, 实习 |

## Agent Workflow Examples

### 搜索 → 批量打招呼

```bash
jevboss batch-greet "golang" --city 杭州 --salary 20-30K --dry-run   # 先预览
jevboss batch-greet "golang" --city 杭州 --salary 20-30K -n 10       # 再执行
```

### 搜索 → 详情（结构化）

```bash
SEC_ID=$(jevboss search "golang" --city 杭州 --json | jq -r '.data.jobList[0].securityId')
jevboss detail "$SEC_ID" --json | jq '.data.jobInfo | {jobName, salaryDesc, skills}'
```

### 每日查岗

```bash
jevboss recommend --json | jq '.data.jobList | length'
jevboss search "Python" --city 杭州 --json
jevboss show 1
jevboss applied --json
jevboss interviews --json
```

## Error Codes

结构化错误码在 `error.code` 字段（见 [SCHEMA.md](./SCHEMA.md)）：

- `not_authenticated` — Cookie 过期或缺失
- `rate_limited` — 请求过多（内置自动冷却）
- `invalid_params` — 参数缺失或非法
- `api_error` — 上游 API 错误
- `unknown_error` — 未预期错误

## Limitations

- **不能发消息** — 无法发送聊天消息（需 MQTT/Protobuf）
- **不能编辑简历** — 无法在 CLI 编辑简历
- **公司搜索受限** — 公司页返回 HTML（依赖 `__zp_stoken__`）
- **单账号** — 同一时间一套 Cookie
- **限流** — 批量打招呼内置 1.5s 间隔

## Anti-Detection / Safety Notes

- **不要并行请求** — 内置高斯抖动延迟用于账号安全
- **限流自恢复**：出现 code=9 时自动退避（10s→20s→40s→60s）并重试一次
- **调试用 `-v`**：`jevboss -v search "Python"` 显示请求耗时
- **批量打招呼建议 ≤ 10 次/会话**，避免被风控
- **Cookie 自动刷新**：≥ 7 天旧时自动尝试浏览器提取
- 不要把原始 Cookie 值暴露在日志中；优先本地浏览器提取
