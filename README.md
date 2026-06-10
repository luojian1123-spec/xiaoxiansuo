# LeadFlow Backend

单用户源码部署版后端基础工程，面向中小型企业线索管理系统，技术选型为 `PHP + MySQL`。

## 当前已落地内容

- 基础目录结构
- 应用引导与环境配置
- 简易路由与统一 JSON 响应
- 健康检查接口
- 认证登录与会话校验
- 部门 / 组 / 员工管理接口
- 线索列表 / 详情 / 新增 / 归组 / 分发 / 跟进
- 异常处理列表 / 异常处理详情 / 异常处理动作
- 驾驶舱与报表中心聚合接口
- 消息通道概览接口
- 导入中心接口
- MySQL 初始化迁移脚本
- 基础种子数据

## 目录说明

- `public/` Web 入口
- `bootstrap/` 应用启动
- `config/` 配置文件
- `routes/` 路由定义
- `src/Core/` 框架基础能力
- `src/Controllers/` 控制器
- `src/Domain/` 业务领域模块
- `database/migrations/` 建表脚本
- `database/seeds/` 基础数据
- `storage/logs/` 运行日志

## 本地启动

1. 复制环境文件

```bash
cp backend/.env.example backend/.env
```

2. 创建数据库

```sql
CREATE DATABASE leadflow DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

3. 执行迁移与种子

```bash
php backend/bin/migrate.php
```

4. 启动开发服务器

```bash
php -S 127.0.0.1:8080 -t backend/public
```

## Smoke 回归脚本

用于核心接口重构后的快速回归验证：

```bash
php backend/tests/smoke/api_smoke.php --base-url=http://127.0.0.1:8085
```

如果需要顺带验证“新增跟进”写链路，可额外指定一个测试线索：

```bash
php backend/tests/smoke/api_smoke.php --base-url=http://127.0.0.1:8085 --write-followup-lead-id=1
```

默认会校验：

- 管理员登录
- 主管登录
- 跟进记录列表
- 跟进记录页元数据
- 团队线索页元数据
- 驾驶舱周视图
- 数据中心月报表
- 跟进附件下载
- 可选：新增跟进写链路

## 当前接口

- `POST /api/auth/login`
- `GET /api/auth/me`
- `POST /api/auth/logout`
- `POST /api/auth/change-password`
- `GET /api/health`
- `GET /api/system/overview`
- `GET /api/dashboard/overview`
- `GET /api/reports/overview`
- `GET /api/reports/export`
- `GET /api/message/channels`
- `GET /api/import/template`
- `POST /api/import/jobs`
- `GET /api/import/jobs`
- `GET /api/import/job`
- `GET /api/import/job-report`
- `GET /api/leads/exceptions`
- `GET /api/leads/exception-detail`
- `GET /api/departments`
- `POST /api/departments`
- `GET /api/groups`
- `POST /api/groups`
- `GET /api/employees`
- `POST /api/employees`
- `GET /api/leads`
- `GET /api/lead`
- `POST /api/leads`
- `POST /api/leads/mark-exception`
- `POST /api/leads/resolve-exception`
- `POST /api/leads/assign-rule-group`
- `GET /api/my-leads`
- `GET /api/supervisor-queue`
- `GET /api/returns`
- `POST /api/leads/return`
- `POST /api/leads/return/resolve`
- `POST /api/follow-ups`
- `GET /api/rule-groups`
- `GET /api/rule-group`
- `POST /api/rule-groups`
- `POST /api/rule-groups/assign`

## 默认管理员

- 账号：`admin@admin.com`
- 密码：`11111111`

说明：

- 默认管理员通过数据库种子写入。
- 执行迁移前请先确保本机 MySQL 服务已启动，并且 `.env` 中的数据库连接参数可用。

## 下一步开发顺序

1. 前端页面联调
2. 异常处理合并动作增强
3. 消息中心与异常提醒扩展
4. 微信绑定与公众号模板消息接入

## 导入中心说明

- 第一版仅支持 `CSV` 文件导入
- 标准模板列头固定为：`姓名,手机号,来源编码,线索类型,地区,备注`
- 模板列头必须完全一致，且不能改列名、删列或调整顺序
- 硬性必填列：`姓名`、`手机号`、`来源编码`
- 强烈建议填写：`线索类型`、`地区`
- `来源编码` 需填写编码值，如 `manual`、`import`
- `线索类型` 需填写系统标准名称，如 `试听预约`
- 当前第一版导入模板不支持：`方便联系时间`、`意向产品 / 服务`、`客户阶段`、`客户需求描述`
- 文件大小上限 `10MB`
- 支持 `UTF-8`、`UTF-8 with BOM`、`GBK/GB18030` 自动转码
- 手机号需为中国大陆 11 位手机号，建议在表格软件中按文本格式填写，避免被转成科学计数法
- 重复手机号不会拦截导入，但会进入“异常待处理-重复”口径
- 导入文件保存目录：`backend/storage/imports/`
- 导入报告输出目录：`backend/storage/import-reports/`

## 线索录入归属策略

- 员工账号新增/编辑时可设置 `lead_intake_mode`
- `public_pool`：新线索默认进入公海
- `self_follow`：新线索直接归本人，进入个人承接链路
- 该策略会同时写入用户资料和新建线索快照，便于后续归档与异常恢复时保持一致
