# 项目简介

Flasky 是一个基于 Flask 的博客/社交微平台示例项目，涵盖用户认证、角色与权限、文章与评论、分页与慢查询日志、REST API、邮件通知、部署等完整功能，适合用作学习与扩展。

## 概览
- 应用工厂、蓝图分层、配置多环境、数据库迁移与测试一应俱全
- 提供 Web 界面与 REST API（基础认证与令牌认证）
- 支持用户注册/登录、邮箱确认、密码重置、资料编辑、关注/取关、Markdown 渲染、评论审核

## 技术栈
- 核心：`Flask`、`Flask-SQLAlchemy`、`Flask-Login`、`Flask-Migrate`
- 前端：`Flask-Bootstrap`、`Flask-Moment`、`Flask-PageDown`、`Jinja2`
- 其他：`Flask-Mail`、`Flask-HTTPAuth`、`Markdown`、`bleach`

## 目录结构（前后端）
- 前端目录
  - `app/templates/` 页面与片段
    - 布局与片段：`base.html`、`_posts.html`、`_comments.html`、`_macros.html`
    - 主站页面：`index.html`、`post.html`、`edit_post.html`、`user.html`、`followers.html`、`moderate.html`、错误页 `403.html`、`404.html`、`500.html`
    - 认证页面与邮件模板：`auth/*.html`、`auth/email/*`、`mail/*`
  - `app/static/` 静态资源（`styles.css`、`favicon.ico`）
- 后端目录
  - 站点视图与表单：`app/main/`、`app/auth/`
  - REST API：`app/api/`
  - 业务与基础：`app/models.py`、`app/decorators.py`、`app/email.py`、`app/exceptions.py`、`app/fake.py`、`app/__init__.py`
  - 配置与入口：`config.py`、`flasky.py`
  - 数据库迁移：`migrations/`（`alembic.ini`、`env.py`、`versions/`）
  - 依赖与测试：`requirements/`、`requirements.txt`、`tests/`
  - 部署与运维：`Dockerfile`、`docker-compose.yml`、`Procfile`、`boot.sh`
  
## 应用架构
- 应用工厂：`create_app` 负责加载配置、初始化扩展并注册蓝图  
  参考：`app/__init__.py:20`、蓝图注册见 `app/__init__.py:36-44`
- 配置选择：通过环境变量 `FLASK_CONFIG` 指定（默认 `development`）
- `.env` 加载：入口会尝试读取 `.env` 并注入环境变量  
  参考：`flasky.py:4-6`

## 数据模型
- 用户与角色：`User`、`Role`、`Permission`（位标志）
- 关系：`Follow`（关注关系）、`Post`（文章）、`Comment`（评论）
- Markdown 渲染与 XSS 清理：`markdown` + `bleach`  
  参考：`app/models.py:13-19, 289-327, 330-367`

## 主要功能
- 认证与账号：注册、登录、邮箱确认、密码重置、邮箱修改  
  参考：`app/auth/views.py:30-90, 93-140, 142-169`
- 文章与评论：发布、编辑、分页浏览与评论  
  参考：`app/main/views.py:35-157, 118-139`
- 关注与时间线：关注/取关、粉丝列表、关注时间线  
  参考：`app/main/views.py:160-191, 194-225`
- 评论审核：启用/禁用评论（需版主权限）  
  参考：`app/main/views.py:244-278`
- 权限装饰器：`permission_required`、`admin_required`  
  参考：`app/decorators.py:7-19`

## 配置与环境变量
- 关键变量：`SECRET_KEY`、`MAIL_*`、`FLASKY_ADMIN`、数据库 URL
- 配置类：`DevelopmentConfig`、`TestingConfig`、`ProductionConfig` 等  
  参考：`config.py:5-23, 29-47, 71-117`
- 生产域名：`SERVER_NAME`（生产需设置）  
  参考：`config.py:45`
- 慢查询阈值：`FLASKY_SLOW_DB_QUERY_TIME`，日志在主蓝图 after-app 钩子  
  参考：`app/main/views.py:13-21`

## 本地运行（Windows）
- 创建虚拟环境与安装依赖（开发依赖）
  - `python -m venv .venv`
  - `.venv\Scripts\activate`
  - `pip install -r requirements\dev.txt`
- 配置环境变量
  - `set FLASK_APP=flasky.py`
  - `set FLASK_CONFIG=development`
- 初始化数据库并运行
  - `flask db upgrade`
  - `flask run`

## 数据库迁移
- 初始化与升级
  - `flask db init`（首次）
  - `flask db migrate -m "message"`
  - `flask db upgrade`
- 部署任务（角色与自关注初始化）  
  参考 CLI：`flasky.py:72-82`，角色插入与自关注见 `app/models.py:35-55, 110-117`

## REST API 快速上手
- 认证
  - 基础认证：`Authorization: Basic base64(email:password)`
  - 获取令牌：`POST /api/v1/tokens/`（需基础认证）  
    参考：`app/api/authentication.py:39-44`
  - 令牌认证：`Authorization: Basic base64(token:)`
- 帖子
  - 列表：`GET /api/v1/posts/`；单个：`GET /api/v1/posts/<id>`  
    参考：`app/api/posts.py:9-34`
  - 新建：`POST /api/v1/posts/`（需写权限）  
    参考：`app/api/posts.py:36-45`
  - 编辑：`PUT /api/v1/posts/<id>`（作者或管理员）  
    参考：`app/api/posts.py:47-57`
- 用户
  - 获取用户：`GET /api/v1/users/<id>`  
  - 用户帖子：`GET /api/v1/users/<id>/posts/`  
  - 关注时间线：`GET /api/v1/users/<id>/timeline/`  
    参考：`app/api/users.py:6-53`
- 评论
  - 列表与单条：`GET /api/v1/comments/`、`GET /api/v1/comments/<id>`  
  - 帖子评论：`GET/POST /api/v1/posts/<id>/comments/`  
    参考：`app/api/comments.py:8-67`

## 命令行与维护任务
- 交互 shell：预置模型上下文  
  参考：`flasky.py:24-27`
- 测试：`flask test [--coverage] [test_names...]`  
  参考：`flasky.py:30-56`
- 性能分析：`flask profile --length 25 --profile-dir tmp/profile`  
  参考：`flasky.py:64-69`
- 部署初始化：`flask deploy`  
  参考：`flasky.py:72-82`

## 测试
- 基础、客户端、API、模型等测试用例  
  参考：`tests/test_basics.py`、`tests/test_client.py`、`tests/test_api.py`
- 运行方式：`flask test --coverage` 或按名称运行部分测试

## 部署
- Heroku：`HerokuConfig`（自动 SSL 重定向、日志到 stderr）  
  参考：`config.py:71-91`
- Docker：`DockerConfig`（日志到 stderr）  
  参考：`config.py:93-104`
- Unix：`UnixConfig`（日志到 syslog）  
  参考：`config.py:106-117`

## 关键代码索引
- 应用工厂与蓝图：`app/__init__.py:20, 36-44`
- 配置类与环境：`config.py:5-23, 29-47, 71-117`
- 模型与权限：`app/models.py:13-19, 21-71, 83-281, 289-367`
- 权限装饰器：`app/decorators.py:7-19`
- Web 视图（主站）：`app/main/views.py:35-278`
- 认证视图：`app/auth/views.py:12-21, 30-90, 93-169`
- API 蓝图：`app/api/*.py`（用户、帖子、评论、认证）
- CLI 命令：`flasky.py:24-82`

## 参考与鸣谢
本项目示例源自《Flask Web Development（第二版）》示例代码，适合用于学习与实践扩展。