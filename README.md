# CampusChat 校园即时通信与文件传输系统

基于 LAN 的校园即时通信与文件传输系统，支持文字聊天、文件共享、好友管理、群组讨论等功能，无需互联网连接。

## 技术栈

| 层级 | 技术 |
|------|------|
| 后端框架 | Java 17, Spring Boot 3.3.5, Maven |
| 数据库 | MySQL 8.0 |
| ORM | MyBatis |
| 实时通信 | Spring WebSocket (原生 WebSocket) |
| 认证 | JWT (jjwt 0.12.6), BCrypt 密码加密 |
| 文件加密 | AES-256-GCM |
| 前端框架 | Vue 3, Vite 4 |
| 状态管理 | Pinia |
| HTTP 客户端 | Axios |
| 路由 | Vue Router 4 |

## 功能特性

- **即时通信** — 支持私聊、群聊，WebSocket 实时推送
- **文件传输** — 支持文件上传/下载
- **好友系统** — 好友请求、好友列表、备注管理
- **群组管理** — 创建群组、邀请成员
- **在线状态** — 实时在线用户列表
- **消息状态** — 发送 → 已送达 → 已读 三状态追踪
- **离线消息** — 离线消息持久化，重连后自动拉取
- **系统广播** — 管理员广播通知
- **断线重连** — 指数退避重连（1s→2s→4s→8s→16s）

## 项目结构

```
keshe/
├── backend/                    # Spring Boot 后端
│   ├── src/main/java/com/campuschat/
│   │   ├── config/             # WebSocket、CORS、JWT 配置
│   │   ├── controller/         # REST API 控制器
│   │   ├── service/            # 业务逻辑层
│   │   ├── repository/         # MyBatis Mapper 接口
│   │   ├── entity/             # 数据实体
│   │   ├── dto/                # 请求/响应 DTO
│   │   ├── websocket/          # WebSocket 处理器
│   │   ├── util/               # 工具类 (JWT 等)
│   │   └── exception/          # 异常处理
│   ├── src/main/resources/
│   │   ├── application.yml     # 应用配置
│   │   └── mappers/            # MyBatis XML 映射文件
│   └── pom.xml
├── frontend/                   # Vue 3 前端
│   ├── src/
│   │   ├── api/                # Axios API 封装
│   │   ├── websocket/          # WebSocket 连接管理
│   │   ├── stores/             # Pinia 状态存储
│   │   ├── views/              # 页面组件
│   │   ├── components/         # 通用组件
│   │   └── router/             # 路由配置
│   ├── vite.config.js
│   └── package.json
├── 需求分析文档.md              # 需求规格说明
├── 接口文档.md                  # REST API & WebSocket 协议文档
└── 前后端编码流程.md             # 开发路线图
```

## 快速开始

### 环境要求

- JDK 17+
- Maven 3.8+
- MySQL 8.0
- Node.js 18+
- npm 9+

### 数据库配置

创建 MySQL 数据库：

```sql
CREATE DATABASE campuschat DEFAULT CHARACTER SET utf8mb4;
```

修改 `backend/src/main/resources/application.yml` 中的数据库连接信息：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/campuschat?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai&characterEncoding=utf-8
    username: your_username
    password: your_password
```

### 启动后端

```bash
cd backend
./mvnw spring-boot:run
```

后端默认运行在 `http://localhost:8088`。

### 启动前端

```bash
cd frontend
npm install
npm run dev
```

前端默认运行在 `http://localhost:5173`，API 请求自动代理到后端 8088 端口。

## API 概览

所有 API 前缀为 `/api/`，认证通过 `Authorization: Bearer <token>` 请求头传递。统一响应格式：`{ code, message, data }`。

| 模块 | 端点 |
|------|------|
| 认证 | `POST /api/auth/register` `POST /api/auth/login` |
| 用户 | `GET /api/users/me` `GET /api/users/online` `GET /api/users/search` |
| 好友 | `POST /api/friends/request` `GET /api/friends` `PUT /api/friends/{id}/remark` |
| 群组 | `POST /api/groups/create` `POST /api/groups/invite` `GET /api/groups/my` |
| 消息 | `GET /api/messages/private/{userId}` `GET /api/messages/group/{roomId}` `GET /api/messages/search` |
| 文件 | `POST /api/files/upload` `GET /api/files/{id}/download` |
| 分片上传 | `GET /api/files/chunk/status` `POST /api/files/chunk/upload` `POST /api/files/chunk/merge` |
| 管理 | `POST /api/admin/broadcast` (需要 ADMIN 角色) |

## WebSocket 协议

- **端点**: `ws://host:8088/ws?token=<JWT>`
- **消息格式**: `{ type, data }` JSON
- **客户端→服务端**: `PRIVATE_MESSAGE`, `GROUP_MESSAGE`, `PING`, `MESSAGE_READ`, `REQUEST_MISSING_MESSAGES`
- **服务端→客户端**: `PRIVATE_MESSAGE`, `GROUP_MESSAGE`, `ONLINE_USERS`, `FILE_NOTIFICATION`, `PONG`, `OFFLINE_MESSAGES`, `FRIEND_REQUEST`, `FRIEND_ADDED`, `GROUP_INVITE`, `SYSTEM_BROADCAST`

详细协议见 `接口文档.md`。

## 数据库表

`user`, `chat_message`, `file_record`, `friend`, `group_room`, `group_member`, `upload_chunk` — 共 7 张表，完整 DDL 见 `接口文档.md` §13.1。

