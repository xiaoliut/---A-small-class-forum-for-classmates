# 班级论坛（Class Forum）

一个面向中学班级的轻量论坛，开箱即用、零外部服务依赖。

**技术栈**：Node.js + Express + SQLite + EJS（服务端渲染，移动端自适应，浅色校园风）。

**核心能力**：审核制注册 · 帖子/评论 · 图片/GIF/表情/封面 · AI 辅助审核（离线规则 + 可选大模型，默认关闭）· 自定义头像 · 三级权限（超级管理员/管理员/同学）· 改名/销号审核 · 管理员后台 · 论坛规范可编辑。

---

## 功能亮点

- **审核制注册**：注册只提交「入班申请」（邮箱/手机号至少填一个），管理员通过后才能发帖评论。
- **帖子 / 评论**：支持分区、搜索、分页、置顶/公告、警告、软删除；评论违规自动隐藏；列表**整卡可点击**进入详情。
- **图片 / 表情 / 封面**：帖子和评论可上传图片（含 GIF 动图），自带表情面板，点击图片全屏放大；发帖可**自定义封面图**（图片/动图，不传则不显示封面）。
- **AI 辅助审核**：本地规则库（默认离线可用）+ 可选大模型 API，双引擎取更严结果；**默认关闭（纯人工审核）**，后台可开启并配置大模型（Base URL / API Key / 模型）。
- **自定义头像**：弹窗式圆形裁剪器，拖动图片、调裁剪大小，保存即生效。
- **用户联系方式**：注册必填邮箱/手机号，后台「用户管理」列表直接显示每个用户的联系方式。
- **三级权限**：`superadmin` 超级管理员 / `admin` 管理员 / `student` 同学。超级管理员可改站点所有内容、增删管理员；普通管理员只管业务，不能动管理员账号。
- **改名审核**：成员改真实姓名由管理员审核，管理员改名由超级管理员审核。
- **销号审核**：成员销号由管理员审核、管理员销号由超级管理员审核，通过后进入 7 天冷静期，到期自动注销（账号脱敏，帖子保留）。
- **论坛规范可编辑**：`/rules` 页面内容由超级管理员在后台直接编辑。
- **安全响应头**：内置 CSP、X-Frame-Options、nosniff、COOP、Referrer-Policy、Permissions-Policy、HSTS。

---

## 快速开始

```bash
cd class-forum
npm install --omit=dev     # 生产也这样装；--omit=dev 跳过部署工具依赖
cp .env.example .env       # 可选，按需修改
node init-db.js            # 建表 + 创建初始超级管理员
npm start                  # 前台启动，Ctrl+C 停止
```

访问 **http://127.0.0.1:3000**

### 内置账号

默认只创建 **1 个超级管理员**，管理员和同学由超级管理员登录后在后台添加：

| 角色 | 用户名 | 密码 |
| --- | --- | --- |
| 超级管理员 | `admin` | `admin123` |

> ⚠️ 部署上线前务必先修改密码，并设置 `SESSION_SECRET`。

### 环境要求

- Node.js ≥ 18。推荐 **Node 22+**（内置 `node:sqlite`，零原生编译；老版本自动回退 `better-sqlite3`）。
- Node 22/24 使用 `node:sqlite` 时会打印 ExperimentalWarning，属正常现象。

---

## 部署方法

### 本地 / 开发

```bash
npm start       # 前台运行
npm run serve   # 后台常驻（日志写 data/server.log）
npm run stop    # 停止后台服务
```

### Linux 服务器（生产，推荐）

三步跑起来，完整版见 [`DEPLOY.md`](DEPLOY.md)：

```bash
# 1. 装 Node 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs

# 2. 上传代码后装依赖 + 建库
cd /www/wwwroot/class-forum
npm install --omit=dev
node init-db.js

# 3. 用 systemd 常驻
cat > /etc/systemd/system/class-forum.service <<'EOF'
[Unit]
Description=Class Forum
After=network.target
[Service]
Type=simple
User=www-data
WorkingDirectory=/www/wwwroot/class-forum
ExecStart=/usr/bin/node app.js
Restart=always
Environment=NODE_ENV=production
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload && systemctl enable --now class-forum
```

然后配 Nginx 反向代理（80 → 3000）和 HTTPS，详见 `DEPLOY.md`。

---

## 可修改的配置

所有日常可改的内容集中在 `config/` 目录（后台「站点设置」可视化编辑，保存即生效并写回文件）：

| 文件 | 管什么 |
| --- | --- |
| `config/site.jsonc` | 站点名/欢迎语/副标题、**页脚版权**、ICP/公安备案号、页脚链接、功能开关 |
| `config/sections.jsonc` | 论坛分区（增删板块、改名、设管理员专属） |
| `config/moderation.jsonc` | AI 审核关键词、权重、风险分阈值 |

页脚版权格式：多行，一行一条，每行自动加「© 年份」前缀；行末用 `|网址` 则该行可点击。默认值：

```
杂鱼科技-杂鱼工作室|https://zayukeji.top
```

---

## 目录结构

```
class-forum/
├── app.js                    # 应用入口：中间件、安全头、会话、路由装配
├── start.js / stop.js        # 后台常驻启动 / 停止
├── init-db.js                # 数据库初始化（--force 重置）
├── build.js                  # npm run build（初始化数据库）
├── package.json
├── .env.example              # 运行时环境变量示例
├── DEPLOY.md                 # 详细部署指南
├── config/                   # ★ 可修改配置（站点信息/页脚/分区/审核规则）
│   ├── README.md
│   ├── site.jsonc
│   ├── sections.jsonc
│   └── moderation.jsonc
├── data/                     # 运行时生成（SQLite 库、头像/图片、日志、PID，已 gitignore）
├── public/
│   ├── css/style.css
│   └── js/                   # app.js、image-upload.js（图片上传/表情）、avatar-cropper.js（头像裁剪）
├── src/
│   ├── config.js             # .env 读取
│   ├── settings.js           # config/*.jsonc 加载/合并/写回 + AI 配置 + 规范内容
│   ├── db.js                 # SQLite 封装（node:sqlite 优先，回退 better-sqlite3）
│   ├── init-db.js            # 建表 SQL + 干净种子
│   ├── ai.js                 # AI 审核（规则引擎 + 大模型）
│   ├── account-deletion.js   # 销号生命周期（申请/审核/冷静期/软删除）
│   ├── middleware/           # flash、CSRF、登录态、权限、审计日志
│   ├── routes/               # index/auth/posts/comments/me/admin/uploads
│   └── utils/view-helpers.js # 模板辅助函数、页脚组装、头像/图片渲染
├── test/
│   └── smoke.js              # 端到端冒烟测试（npm test）
└── views/                    # EJS 模板（含 admin/ 子目录与 partials）
```

---

## 数据表

| 表 | 用途 |
| --- | --- |
| `users` | 账号、身份信息、角色/状态、邮箱/手机号、头像、禁言/封禁 |
| `posts` | 帖子正文、图片、分区、状态、置顶/公告、AI 判定、警告/驳回 |
| `comments` | 评论内容、图片、状态、AI 判定 |
| `reports` | 举报记录与处理状态 |
| `rename_requests` | 改名申请（待审/通过/驳回） |
| `account_deletion_requests` | 销号申请（待审/冷静期/已处理） |
| `sessions` | 会话存储 |
| `moderation_logs` | 管理动作审计日志 |
| `settings` | 后台「站点设置」保存的配置覆盖值 |

---

## 环境变量

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `PORT` | `3000` | 监听端口 |
| `HOST` | `127.0.0.1` | 监听地址，局域网访问设 `0.0.0.0` |
| `SITE_NAME` | `班级论坛` | 站点名称（也可后台/配置改） |
| `SESSION_SECRET` | 开发默认值 | **生产必须修改** |
| `ACCESS_LOG` | `1` | 是否输出访问日志 |
| `DATA_DIR` | `./data` | 数据目录 |
| `ADMIN_PASSWORD` | `admin123` | 首次初始化超管密码 |
| `AI_ENABLED` / `AI_BASE_URL` / `AI_API_KEY` / `AI_MODEL` / `AI_TIMEOUT_MS` | 见 `.env.example` | 大模型审核（也可在后台「站点设置」配置） |

> 大模型审核的 API Key 也可在后台「站点设置 → AI 大模型接入」里配置，只存数据库、不进源码包。

---

## 安全说明

- 表单均带 **CSRF Token** 校验。
- 所有 SQL 用**参数化查询**，模板输出经 EJS 转义，防注入与 XSS。
- 内置 **CSP / X-Frame-Options / nosniff / COOP / Referrer-Policy / HSTS** 等安全响应头。
- 密码 `bcryptjs`（cost 10）哈希存储。
- 登录失败统一提示，不暴露账号是否存在。
- 管理动作全部写入 `moderation_logs`，可追溯；内容删除为软删除。

---

## 测试与常用命令

```bash
npm start          # 前台启动
npm run dev        # 启动并监听文件变化
npm run serve      # 后台常驻启动
npm run stop       # 停止后台服务
npm run init-db    # 建表 + 首次种子（幂等）
npm run reset-db   # 删库重建（干净状态）
npm test           # 端到端冒烟测试（83 项断言，需服务已启动）
```

`npm test` 覆盖：公开页面/404、登录与权限边界、注册→审核、发帖 AI 审核、评论、图片上传链路、改名/销号审核、页脚设置与热重载等 83 项断言，测试结束自动清理临时数据并还原配置。

---

## License

MIT
