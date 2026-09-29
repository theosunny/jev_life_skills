# jev-boss-cli

在终端使用 **BOSS 直聘**（boss.zhipin.com）的命令行工具 —— 搜索职位、查看推荐、管理投递、与 HR 互动、批量打招呼。

> 📌 这是 [jackwener/boss-cli](https://github.com/jackwener/boss-cli)（Apache-2.0）的 **fork**，保留全部原有能力，并修复了登录态校验问题，使浏览器 Cookie 登录开箱即用。

---

## ✨ 本 fork 相较原版的关键改动

原版 `kabi-boss-cli` 把 `__zp_stoken__` 列入「必需 Cookie」，但 `__zp_stoken__` 是 zhipin.com 网页端 JS 在**客户端生成**的反爬令牌，服务端永远不会通过 `Set-Cookie` 下发。这带来两个坑：

1. **二维码登录拿不到 `__zp_stoken__`** → 启动时 `load_credential()` 判定缺少关键 Cookie，**主动删掉**已保存的凭证；搜索接口直接返回 `code:37 您的环境存在异常`。
2. **浏览器 Cookie 登录（`--cookie-source`）虽自带 `__zp_stoken__`，也会被误判**。

本 fork（v0.3.7）把 `__zp_stoken__` 从 `REQUIRED_COOKIES` 中移除，登录态校验只依赖真正的会话 Cookie（`wt2` / `wbg` / `zp_at`）。

**推荐登录方式**（在浏览器登录 zhipin.com 后提取，天然带完整登录态）：

```bash
jevboss login --cookie-source chrome
```

> 注：原版自带的 `browser_login.py`（用 camoufox 无头浏览器补全 `__zp_stoken__`）在本 fork 中一并保留，但未默认接线；日常使用浏览器 Cookie 提取即可稳定可用。

---

## 📦 安装

需要 Python 3.10+。

```bash
# 方式一：从 PyPI（如已发布）
pip install jev-boss-cli

# 方式二：从本仓库源码安装
git clone https://github.com/theosunny/jev_life_skills.git
cd jev_life_skills/boss-cli
pip install -e .

# 开发依赖（跑测试 / lint）
pip install -e ".[dev]"
```

命令名已重命名为 **`jevboss`**（避免与原版 `kabi-boss-cli` 的 `boss` 命令冲突）。

---

## 🔐 登录

先在任意支持的浏览器（Chrome / Firefox / Edge / Brave / Arc / Safari 等）登录 [www.zhipin.com](https://www.zhipin.com)，然后：

```bash
jevboss login                          # 自动检测有有效 Cookie 的浏览器
jevboss login --cookie-source chrome   # 显式指定浏览器（推荐）
jevboss login --qrcode                 # 二维码登录（终端输出二维码，用 Boss 直聘 APP 扫码）
```

验证：

```bash
jevboss status
jevboss me --json | jq '.data.name'
```

---

## 🚀 常用命令

### 搜索与浏览

| 命令 | 说明 | 示例 |
|------|------|------|
| `jevboss search <关键词>` | 按关键词搜职位（支持筛选） | `jevboss search "CTO" -c 上海` |
| `jevboss show <序号>` | 查看上一次搜索的第 N 条 | `jevboss show 3` |
| `jevboss detail <securityId>` | 查看职位完整详情 | `jevboss detail abc123 --json` |
| `jevboss export <关键词>` | 导出 CSV/JSON | `jevboss export "Python" -n 50 -o jobs.csv` |
| `jevboss recommend` | 个性化推荐 | `jevboss recommend -p 2 --json` |
| `jevboss history` | 浏览历史 | `jevboss history --json` |
| `jevboss cities` | 支持的城市 | `jevboss cities` |

### 个人中心

| 命令 | 说明 |
|------|------|
| `jevboss me` | 个人资料 |
| `jevboss applied` | 已投递职位 |
| `jevboss interviews` | 面试邀请 |
| `jevboss chat` | 沟通过的 HR |

### 互动

| 命令 | 说明 |
|------|------|
| `jevboss greet <securityId>` | 向 HR 打招呼 / 投递 |
| `jevboss batch-greet <关键词>` | 批量打招呼（内置 1.5s 限流间隔） |
| `jevboss batch-greet <关键词> --dry-run` | 预览（不发送） |

### 账号

| 命令 | 说明 |
|------|------|
| `jevboss login` | 提取浏览器 Cookie / 二维码登录 |
| `jevboss status` | 检查登录状态 |
| `jevboss logout` | 清除凭证 |

---

## 🔎 搜索筛选参数

| 筛选 | 参数 | 可选值 |
|------|------|--------|
| 城市 | `--city` / `-c` | 北京, 上海, 杭州, 深圳 等 |
| 薪资 | `--salary` | 3K以下, 3-5K, 5-10K, 10-15K, 15-20K, 20-30K, 30-50K, 50K以上 |
| 经验 | `--exp` | 不限, 在校/应届, 1年以内, 1-3年, 3-5年, 5-10年, 10年以上 |
| 学历 | `--degree` | 不限, 大专, 本科, 硕士, 博士 |
| 行业 | `--industry` | 互联网, 人工智能, 金融, 教育培训, 医疗健康 等 |
| 规模 | `--scale` | 0-20人, 20-99人, 100-499人, 500-999人, 1000-9999人, 10000人以上 |
| 阶段 | `--stage` | 未融资, 天使轮, A轮, B轮, C轮, D轮及以上, 已上市, 不需要融资 |
| 类型 | `--job-type` | 全职, 兼职, 实习 |

分页：`jevboss search "CTO" -c 上海 -p 2`

---

## 🧪 示例

```bash
# 搜上海 CTO，导出前 50 条到 CSV
jevboss search "CTO" -c 上海 -n 50 -o cto_shanghai.csv

# 搜杭州 golang，看 JSON 再取第一条详情
jevboss search "golang" --city 杭州 --json | jq '.data.jobList[0].securityId'
jevboss detail <securityId> --json | jq '.data.jobInfo | {jobName, salaryDesc, skills}'

# 批量打招呼前先预览
jevboss batch-greet "Python" --city 杭州 --dry-run
jevboss batch-greet "Python" --city 杭州 -n 10
```

---

## ⚠️ 限制

- 不能发送聊天消息（需 MQTT/Protobuf）
- 不能编辑简历
- 公司页返回 HTML（依赖 `__zp_stoken__`，部分场景受限）
- 同一时间只支持一套 Cookie
- 批量打招呼内置限流，请勿绕过

---

## 📄 许可证与致谢

- 基于 [jackwener/boss-cli](https://github.com/jackwener/boss-cli) **fork**，原版 Apache-2.0 许可证，原作者 jackwener。
- 本 fork 由 theosunny 维护，修改仅涉及登录态校验修复与命令重命名，能力集合与原版一致。
- 免责声明：本工具仅供个人学习/求职效率使用，请遵守 BOSS 直聘服务条款，勿用于滥用或爬虫攻击。
