# 签到仓库搭建完成 · 操作速查（huickdo/tg-autosign）

> 本仓库 Fork 自 [JYBWOB/tg-singer-actions](https://github.com/JYBWOB/tg-singer-actions)，
> 通过 GitHub Actions + [tg-signer](https://github.com/amchii/tg-signer) 每天自动签到。
> 搭建日期：2026-11-16

---

## ✅ 已完成（我这边）

- Fork 到你的账号：`https://github.com/huickdo/tg-autosign`（保留上游更新能力）
- 工作流、配置模板、一键脚本均已就位
- 默认签到时间：cron `30 2 * * *` = **北京时间每天 10:30**（UTC 02:30）

## ⚠️ 需要你亲自做的两步（我做不了，原因见后）

### 第 1 步：启用 Actions（网页点一下，1 分钟）

Fork 出来的仓库 Actions 默认是**禁用**的，且当前 PAT 没有 Administration 权限，无法通过 API 启用。

> 打开 https://github.com/huickdo/tg-autosign/actions
> 页面会显示 "Workflows aren't being triggered on this repository" 之类的提示 →
> 点击 **"I understand my workflows, go ahead and enable them"** 即可。

### 第 2 步：本地登录生成 Secret（需要手机验证码）

`session_string` 是 Telegram 账号登录凭据，必须**真人收验证码**登录一次才能生成，我无法代劳。

在一台能访问 Telegram、能收验证码的电脑上：

```bash
git clone https://github.com/huickdo/tg-autosign.git
cd tg-autosign
bash scripts/setup.sh            # 交互：选填代理 → 手机号+验证码登录 → 配置签到任务
```

脚本会自动：建 Python 环境 → 装 tg-signer 0.9.2 → 登录 → 引导配置 → 导出 JSON → 若已登录 `gh` CLI，询问是否把两个 Secret 上传到仓库。

**手动上传 Secret**（脚本没连 `gh` 时）：仓库 → Settings → Secrets and variables → Actions → New repository secret：

| Secret 名 | 内容 | 来源 |
|---|---|---|
| `TG_SESSION_STRING` | 登录生成的 `my_account.session_string` 全文 | 脚本输出路径 `~/.local/share/tg-singer-actions/session/` |
| `TG_SIGNER_CONFIG` | 导出的 `sign-config.json` 全文 | 脚本输出路径 `~/.local/share/tg-singer-actions/` |

> 上传后到 Actions 页手动跑一次 `workflow_dispatch`（或 `gh workflow run main.yml`）验证。
> 日志默认隐藏（含对话名/消息，防隐私泄露），只看成功/失败即可。

---

## 🔧 常用修改

| 想改什么 | 改哪里 |
|---|---|
| 签到时间 | `.github/workflows/main.yml` 的 `cron`（**UTC**！北京时 = UTC+8，如 08:00 写 `0 0 * * *`） |
| 加/改签到机器人 | 本地重跑 `bash scripts/setup.sh`，重新导出配置并更新 `TG_SIGNER_CONFIG` Secret |
| 用代理 | 上传 `TG_PROXY` Secret，格式 `socks5://公网IP:端口`（`127.0.0.1` runner 访问不到） |
| 临时禁签到 | Actions 页 Disable workflow，或删掉 cron 那行 |
| 保持定时任务不被停用 | 已内置 `keepalive.yml`：每月 1/15 号空提交保活（需仓库有写权限；此步同样需网页确认 Actions 启用） |

## ⚠️ 六个坑（来自上游 README + 作者 issue）

1. cron 是 **UTC**，且 GitHub 高峰期**不准点**——避开整点。
2. 公共仓库长期无活动会**停定时任务**——keepalive 已配置。
3. **必须先本地登录**拿 session，runner 只复用不能现场登录。
4. session **会过期**（改密码/登出/长期异地）——失效就重跑 setup 更新 Secret。
5. runner 是**数据中心 IP**，可能触发 Telegram 风控/验证码——敏感账号配公网代理。
6. **日志含隐私**，别把完整日志贴公开 Issue。

## 📌 Secret 与文件对照

- 配置模板：`SIGN_CONFIG_DEMO.json`（一个 `chats` 数组可配多个机器人；`actions` 里 `action:1`=发文本、`action:3`=点按钮、`action:2`=骰子、`action:4`=AI识图、`action:5`=答题）
- 一键脚本：`scripts/setup.sh`（登录 + 配置 + 上传 Secret）
- 签到工作流：`.github/workflows/main.yml`
- 保活工作流：`.github/workflows/keepalive.yml`

> 安全：`session_string` 等同账号密码，只存 Secret，绝不进仓库 / 不贴公开 Issue。
