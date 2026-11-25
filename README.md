# 拾光 · Web 效率工作台

一个参考滴答清单产品类别、采用原创界面的**纯网页端全栈 MVP**。前端使用 React + TypeScript + Vite，后端使用 Node.js + NestJS + TypeORM，数据存储在 MySQL。无需付费服务或 API Key。

## 本地运行

环境要求：**Node.js 22.13+、MySQL 8.0.16+**。已在 MySQL 8.4 上验证。

仓库提供 `.nvmrc`。使用 nvm 时先运行 `nvm install` 和 `nvm use`，再安装依赖或启动。

先在自己的 MySQL 实例创建一个**专用空库**，不要指向已有业务库。通过数据库管理工具执行：

```sql
CREATE DATABASE shiguang CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

建议使用仅有该库权限的专用账号。迁移账号需要建表、索引、外键等 DDL 权限；运行账号仅需该库的 SELECT、INSERT、UPDATE、DELETE 权限。账号和权限由数据库管理员配置，不必使用 root。

```bash
npm install
cp .env.example .env
```

在本机编辑 `.env`，填写 `MYSQL_HOST`、`MYSQL_PORT`、`MYSQL_USER`、`MYSQL_PASSWORD`、`MYSQL_DATABASE`，无需在聊天中提供密码。然后执行：

```bash
npm run db:migrate
npm run dev
```

打开 `http://127.0.0.1:5173`。该命令同时启动 Vite 前端（5173）与 Node.js API（3001），前端通过代理访问 `/api`。

后端采用 NestJS 模块、Controller、Service 和 TypeORM 数据层；认证由 Guard 保护，统一异常过滤器保持原有错误格式。前端和 `/api` 路径不变。`npm run dev:server` 仅启动后端，`npm run dev:client` 仅启动前端。后端开发模式通过 TypeScript 编译器保留 NestJS 装饰器元数据并监听重启。

**数据库结构不会自动同步**：`synchronize` 和 `migrationsRun` 均关闭，启动只检查迁移记录。只有显式执行 `npm run db:migrate` 才会建表；重复执行不会重新创建已迁移的表。MySQL DDL 不支持完整事务回滚，首次迁移失败时请先检查专用库状态，再由管理员处理，不要直接在有数据的库中反复尝试。此初始迁移不提供自动删表回滚。

首次打开会自动建立一个独立的体验空间，附带示例清单、任务和过去几天的习惯打卡。注册账号会直接保留这个空间，**不会清空当前数据**。注册后的用户可在不同浏览器登录同一账号。

体验空间依赖 HttpOnly Cookie；清除 Cookie、会话到期或换浏览器后，未注册的体验数据无法找回。请先注册或导出数据。新注册且没有体验空间时也会初始化示例数据，可自行删除。

## 已实现

| 模块 | 功能 |
| --- | --- |
| 任务 | 新建、编辑、完成、重新开启、日期、具体时间、四级优先级、备注、标签、子任务 |
| 清单 | 自定义名称与颜色、修改、删除；删除清单后任务回到收集箱 |
| 智能视图 | 今天（含未完成的逾期任务）、最近 7 天、收集箱、全部任务、按清单 / 标签查看 |
| 任务视图 | 清单与优先级看板、拖动卡片修改优先级、排序、优先级筛选、隐藏已完成任务 |
| 日历 | 月历切换、查看某一天的安排、指定日期添加任务 |
| 四象限 | 高 / 中优先级视为重要，今天及逾期视为紧急；拖动会同步修改优先级与日期 |
| 重复任务 | 每日、每周、每月；完成后创建下一次任务，月末自动收敛到下月最后一天 |
| 番茄专注 | 1–120 分钟、自定义时长、关联任务、刷新恢复、专注历史、服务端校验完成时长 |
| 习惯 | 自定义图标 / 颜色、今日打卡、过去 6 天补卡、连续天数、累计天数 |
| 统计 | 完成数、近 7 天柱状图、完成率、专注分钟、各清单进度；均由实际数据计算 |
| 账号 | 邮箱密码注册 / 登录 / 退出、体验空间升级、账号数据隔离 |
| 其他 | 搜索标题 / 备注 / 标签、回收站恢复与永久删除、JSON 导出、浅 / 深色主题、响应式布局 |

任务详情中的修改需要点击“保存更改”。重复任务按原截止日期推进一次，不会自动跳过所有错过的日期。专注计时不支持暂停，提前结束不会计入已完成统计；专注结束时若网页已关闭，下次进入专注页会根据服务器开始时间结算。

快捷键：`N` 新建任务、`/` 搜索、`?` 帮助、`Esc` 关闭弹窗。输入文字时不会触发全局快捷键。

## 构建与部署

```bash
npm run build
npm run db:migrate
npm start
```

打开 `http://127.0.0.1:3001`。生产模式由同一个 Node.js 进程同时提供 `dist/` 静态页面和 `/api`，无需单独部署前端。

后端、迁移和旧数据导入命令通过 dotenv 自动读取根目录 `.env`；进程环境变量优先。`.env` 已被忽略，不要提交数据库密码。

| 环境变量 | 默认值 | 说明 |
| --- | --- | --- |
| `PORT` | `3001` | API / 生产站点端口 |
| `HOST` | `127.0.0.1` | 对外部署时按需改成 `0.0.0.0` |
| `MYSQL_HOST` / `MYSQL_PORT` | 必填 / `3306` | MySQL 主机和端口 |
| `MYSQL_USER` / `MYSQL_PASSWORD` | 必填 | 数据库账号和密码；空密码需显式配置 |
| `MYSQL_DATABASE` | 必填 | 事先创建的专用数据库 |
| `MYSQL_CONNECTION_LIMIT` | `10` | 连接池上限，1–100 |
| `MYSQL_SSL` | `false` | 远程数据库按服务商要求设为 `true`，启用证书验证 |
| `MYSQL_SSL_CA_PATH` | 空 | 可选的 CA 证书文件路径，仅在 SSL 开启时使用 |
| `COOKIE_SECURE` | `false` | HTTPS 部署时设为 `true` |
| `TRUST_PROXY` | `false` | 仅在可信的单层反向代理后设为 `true` |

开发代理默认连接 `3001`，会读取 `.env` 或进程中的 `PORT`，与后端保持一致，无需手工修改 `vite.config.ts`。更改 `.env` 后重启开发进程。MySQL 应限制网络访问并定期备份。JSON 导出不替代数据库备份。

开发代理必须保留浏览器的 `Host`（`changeOrigin: false`），后端据此校验请求来源。不要改成会自动改写 `Host` 的字符串简写代理，否则首次创建体验空间、注册和保存任务会返回“请求来源不受信任”。反向代理部署时也应透传原始 `Host`，不要通过关闭同源校验来绕过报错。

### 从旧版 SQLite 迁移数据

如果之前已使用 SQLite 版，原 `data/` 文件会保留，不会自动改动或导入。先停止旧版服务并备份 SQLite（包括仍在使用时的 WAL 文件），为新版准备一个专用空 MySQL 库，配置 `.env`，再执行：

```bash
npm run db:migrate
npm run db:import-sqlite -- ./data/shiguang.sqlite
npm run dev
```

导入只读 SQLite，保留账号 ID、密码摘要、会话、清单、任务、习惯及专注记录。目标业务表必须全部为空；导入以事务执行，失败则回滚，不删除或改写原 SQLite。导入时不要同时启动其他写入该目标库的应用。`node:sqlite` 仅用于此兼容工具，Node.js 22 可能提示实验性警告。

### 安全与版本边界

- 密码使用随机盐 + scrypt；会话令牌为随机值，数据库只保存 SHA-256 摘要；Cookie 为 HttpOnly / SameSite=Lax。
- 所有业务接口按用户过滤；参数验证、JSON 大小限制、基础登录 / 注册限流与同源写入校验已经实现。
- 前后端应使用**同一个站点来源**；不要直接开放跨域 Cookie。上线前配置 HTTPS 和 `COOKIE_SECURE=true`。
- 当前是单进程、小规模部署版本，不是滴答清单全部功能复刻，也未做大流量 / 高可用验证。
- 网页端使用精简工作空间：只传最近 30 条已结束专注及正在进行的专注，今日 / 累计统计由 MySQL 对完整历史聚合，导出仍保留全部记录。任务和习惯日志仍全量加载，数据增长后需进一步分页与负载验证。
- 写入成功但刷新失败时会提示「已保存」，关闭创建弹窗，并提供「重新同步」只读重试；不会自动重复发送写入。写入响应本身丢失时，服务端尚无通用幂等键，不能保证手动重提不会产生重复数据。
- **未实现**：团队协作 / 分享、附件、自然语言日期识别、邮箱验证、密码找回、系统推送 / 邮件通知、第三方登录 / 日历同步、通用 JSON 数据导入、移动 App、离线同步。
- 页面铃铛仅列出当前逾期和今日未完成任务，不承诺后台或站外提醒。
- 演示账号不会自动清理；公开开放前应增加验证码、体验空间配额 / 定时清理、审计、备份、账号找回和监控。登录限流为进程内存计数，多实例部署应改为共享存储。
- 导出的 JSON 用于离线备份 / 自行处理，当前界面不提供导入恢复。

## 验证

```bash
npm run typecheck
npm run build
```

API 测试使用真实 MySQL，覆盖账号、数据隔离、任务增删改、重复规则、回收站、习惯、专注时长校验、导出和同源保护。必须明确指定**独立测试实例**，不要使用生产实例。测试只读取 `TEST_MYSQL_*`，不会回退到 `.env` 中的生产数据库；每组测试创建随机命名的 `shiguang_test_*` 库并在结束后删除，仅测试账号需要创建和删除数据库权限。

```bash
export TEST_MYSQL_HOST=127.0.0.1
export TEST_MYSQL_PORT=3306
export TEST_MYSQL_USER=your_test_user
export TEST_MYSQL_PASSWORD='your_test_password'
npm test
```

未配置测试连接时，集成测试会明确报错，不会跳过并假装验证成功。

`npm test` 同时使用项目实际的 Vite 配置启动临时开发代理，验证浏览器同源请求能够创建会话、写入任务，同时拒绝恶意来源及伪造的转发 Host。

浏览器端到端测试会启动独立服务和 Vite 开发代理，使用临时数据库；先检查生产静态站点，再通过真实开发代理验证桌面 / 手机页面与核心交互：

```bash
npx playwright install chromium
npm run test:e2e
```

可通过 `PLAYWRIGHT_EXECUTABLE_PATH` 指定已有的 Chromium / Chrome 可执行文件。截图保存到 `test-results/`。

## 项目结构

```text
src/
  App.tsx                       应用状态、导航、账号与编辑弹窗
  components/TaskViews.tsx       任务清单、看板、日历、四象限
  components/TaskDetail.tsx      任务详情编辑
  components/ProductivityViews.tsx  专注、习惯、统计
  components/UI.tsx              通用 UI
  api.ts / types.ts / utils.ts   请求、类型和日期工具
  styles.css                    响应式样式与主题
server/
  src/main.ts                   NestJS 入口
  src/bootstrap.ts              服务组装、安全中间件与静态服务
  src/app.module.ts             应用模块
  src/auth/                     注册、登录、会话与认证 Guard
  src/tasks/                    任务与重复规则
  src/lists/                    清单管理
  src/habits/                   习惯与打卡
  src/focus/                    番茄专注
  src/workspace/                工作空间与 JSON 导出
  src/database/                 TypeORM 实体、配置、迁移与旧数据导入
  src/common/                   校验、安全中间件、异常过滤器
  dist/                         编译后的后端，不入版本库
scripts/dev.mjs                 编译并监听后端，同时启动前端
tests/                          MySQL API、开发代理与浏览器测试
data/                           仅保留旧 SQLite 数据，不入版本库
```

## API 概览

所有请求体为 JSON，业务接口需要会话 Cookie。错误响应统一为 `{ "error": "说明" }`。

```text
GET    /api/health
GET    /api/auth/me
POST   /api/auth/demo | /api/auth/register | /api/auth/login | /api/auth/logout
GET    /api/workspace
POST   /api/tasks
PATCH  /api/tasks/:id
DELETE /api/tasks/:id              仅允许永久删除已进入回收站的任务
POST   /api/lists
PATCH  /api/lists/:id
DELETE /api/lists/:id
POST   /api/habits
POST   /api/habits/:id/check
DELETE /api/habits/:id
POST   /api/focus
PATCH  /api/focus/:id
GET    /api/export
```

### 精简工作空间

`GET /api/workspace` 默认仍返回完整数据，以兼容原有调用。网页使用以下可选参数减少专注历史传输：

```text
GET /api/workspace?compact=true&dayStart=<本地当天零点毫秒时间戳>&dayEnd=<本地次日零点毫秒时间戳>
```

- `focus` 包含最近 30 条已结束记录和所有正在进行的记录，不删除数据库中的历史。
- `focusSummary` 包含 `totalMinutes`、`completedSessions`、`todayMinutes` 和 `todaySessions`，只统计已完成的专注，按当前用户隔离。
- 今日统计按 `finishedAt`（为空时取 `startedAt`）落入 `[dayStart, dayEnd)` 的记录计算。边界由浏览器本地日期生成，支持夏令时 23 / 25 小时的日期；服务端校验时间戳和 20～28 小时的范围，不依赖 MySQL 时区表。
- `compact=false` 或省略该参数时不截断历史、不附加聚合字段；`GET /api/export` 始终全量导出。
- 同一响应在一致性事务中读取；聚合仍需访问该用户的历史记录，减少传输不等于完成大规模查询优化，也没有新增可翻页的专注历史接口。
