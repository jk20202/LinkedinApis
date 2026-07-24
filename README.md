<div align="center">
    <a href="https://www.python.org/">
        <img src="https://img.shields.io/badge/python-3.10%2B-blue" alt="Python 3.10+">
    </a>
    <a href="https://nodejs.org/zh-cn/">
        <img src="https://img.shields.io/badge/nodejs-20%2B-green" alt="NodeJS 20+">
    </a>
    <a href="https://fastapi.tiangolo.com/">
        <img src="https://img.shields.io/badge/FastAPI-0.115%2B-009688" alt="FastAPI">
    </a>
</div>

# 💼 LinkedIn Platform

**✨ 专业的 LinkedIn 数据采集与私信发送解决方案，支持用户搜索、主页信息获取与消息发送**

当你需要让 AI Agent 感知 LinkedIn 职场生态——自动采集用户信息、批量触达目标人群、驱动销售或招聘策略——第一道墙往往不是模型能力，而是**平台数据获取与操作能力的缺失**。

本项目做的事很简单：把这道墙拆掉。

**⚠️ 严禁用于批量骚扰、爬取用户隐私或违规商业用途！本项目仅供学习与技术研究使用，后果自负。**

## 🌟 功能特性

- 🔍 **用户搜索**
  - 支持关键词全局搜索 LinkedIn 用户
  - 自动获取 GraphQL `queryId`，适配最新接口
- 👤 **用户主页信息获取**
  - 获取用户姓名、职位标题（headline）、个人简介（About）
  - 自动解析 JS 资源提取动态 `queryId`
- 💬 **私信发送**
  - 支持向企业主页 / 学校主页发送消息
  - 自动提取 `mailboxUrn` 与 `hostRecipientUrns`
- 🚀 **高性能服务**
  - 基于 FastAPI + Uvicorn 异步服务
  - 支持 Docker 一键部署

## 🛠️ 快速开始

### ⛳ 运行环境

- Python 3.10+
- Node.js 20+

### 🎯 本地安装

```bash
pip install -r requirements.txt
cd static && npm install
```

### 🚀 运行项目

```bash
python App.py
# 可选指定端口，默认 5011
python App.py -p 5011
```

服务启动后访问 http://localhost:5011/docs 查看交互式 API 文档。

### 🎨 Cookie 配置

在浏览器中打开 [www.linkedin.com](https://www.linkedin.com)，**登录账号**后按 `F12` 打开开发者工具，点击「网络」→ 切换到 Fetch/XHR → 找任意一个 API 请求 → 复制请求头中的 `Cookie` 字段值。

> ⚠️ 注意：必须登录后获取的 Cookie 才有效，其中 `JSESSIONID` 字段用于 CSRF Token 鉴权，缺失将导致请求失败。

将获取到的 Cookie 字符串作为 `cookies_str` 参数传入接口，格式如下：

```
bcookie=xxx; JSESSIONID="xxx"; li_at=xxx; ...
```

## 📡 接口说明

所有接口均为 `POST /{method}`，请求体为 JSON，必须包含 `cookies_str` 字段。

---

### POST `/get_user_info`

获取 LinkedIn **用户主页**的基本信息（姓名、职位、个人简介）。

**请求参数**

| 字段          | 类型  | 必填 | 说明                                              |
|-------------|-----|----|--------------------------------------------------|
| user_url    | str | 是  | 用户主页 URL，如 `https://www.linkedin.com/in/xxx/` |
| cookies_str | str | 是  | LinkedIn 登录 Cookie 字符串                         |

**请求示例**

```bash
curl -X POST http://localhost:5011/get_user_info \
  -H "Content-Type: application/json" \
  -d '{
    "user_url": "https://www.linkedin.com/in/davidailsworth/",
    "cookies_str": "bcookie=xxx; JSESSIONID=\"xxx\"; li_at=xxx"
  }'
```

**响应示例**

```json
{
  "user_name": "David Ailsworth",
  "user_id": "davidailsworth",
  "desc": "个人 About 简介内容...",
  "top_desc": "Software Engineer at XXX Company"
}
```

---

### POST `/search_some`

搜索 LinkedIn **用户列表**（全局搜索）。

**请求参数**

| 字段          | 类型  | 必填 | 说明                        |
|-------------|-----|----|---------------------------|
| start       | int | 是  | 分页起始偏移量，从 0 开始            |
| query       | str | 是  | 搜索关键词                     |
| query_id    | str | 是  | GraphQL queryId（见下方获取方法）  |
| cookies_str | str | 是  | LinkedIn 登录 Cookie 字符串    |

> `query_id` 需先调用 `/get_search_query_id` 接口获取。

**请求示例**

```bash
curl -X POST http://localhost:5011/search_some \
  -H "Content-Type: application/json" \
  -d '{
    "start": 0,
    "query": "David Ailsworth",
    "query_id": "voyagerSearchDashClusters.xxxxxxxx",
    "cookies_str": "bcookie=xxx; JSESSIONID=\"xxx\""
  }'
```

---

### POST `/get_search_query_id`

获取搜索接口所需的 GraphQL **queryId**（动态值，随版本更新变化）。

**请求参数**

| 字段          | 类型  | 必填 | 说明                     |
|-------------|-----|----|------------------------|
| cookies_str | str | 是  | LinkedIn 登录 Cookie 字符串 |

**请求示例**

```bash
curl -X POST http://localhost:5011/get_search_query_id \
  -H "Content-Type: application/json" \
  -d '{
    "cookies_str": "bcookie=xxx; JSESSIONID=\"xxx\""
  }'
```

---

### POST `/send_msg`

向 LinkedIn **企业主页 / 学校主页**发送私信。

**请求参数**

| 字段          | 类型  | 必填 | 说明                                              |
|-------------|-----|----|--------------------------------------------------|
| home_url    | str | 是  | 企业或学校主页 URL，如 `https://www.linkedin.com/company/xxx/` |
| msg         | str | 是  | 消息内容                                            |
| cookies_str | str | 是  | LinkedIn 登录 Cookie 字符串                         |

**请求示例**

```bash
curl -X POST http://localhost:5011/send_msg \
  -H "Content-Type: application/json" \
  -d '{
    "home_url": "https://www.linkedin.com/company/farmoutph/",
    "msg": "Hello, I am interested in your services.",
    "cookies_str": "bcookie=xxx; JSESSIONID=\"xxx\""
  }'
```

## 🐳 Docker 部署

```bash
docker build -t linkedin-platform .
docker run -d -p 5011:5011 linkedin-platform
```

## 🍥 日志

| 日期       | 说明                                          |
|----------|-----------------------------------------------|
| 26/04/10 | 项目初始化，完成用户搜索、主页信息获取、私信发送 API 封装 |

## 🤝 欢迎贡献 PR

本项目欢迎任何形式的贡献！如果你有新功能想法、Bug 修复或文档改进，欢迎提交 PR。

- Fork 本仓库并在新分支上开发
- 保持代码风格与现有代码一致
- PR 描述中请简要说明改动内容和目的
- 也欢迎通过 Issue 提出建议或报告问题

## 🧸 额外说明
1. 感谢 star⭐ 和 follow📰！不时更新
2. 作者的联系方式在主页里，有问题可以随时联系我
3. 可以关注下作者的其他项目，欢迎 PR 和 issue
4. 感谢赞助！如果此项目对您有帮助，请作者喝一杯奶茶~~ （开心一整天😊😊）
5. thank you~~~


## 🍔 交流群

如果你对爬虫和 AI Agent 感兴趣，请加作者主页 wx 通过邀请加入群聊

ps: 请加群，人满或者过期 issue | wx 提醒

| group-1 | group-2 | group-3 |
|:--:|:--:|:--:|
| <img width="280" alt="group1" src="https://cvcat.site/assets/group1.jpg" /> | <img width="280" alt="group2" src="https://cvcat.site/assets/group2.jpg" /> | <img width="280" alt="group3" src="https://cvcat.site/assets/group3.jpg" /> |
