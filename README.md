# AIChat-10 一体化说明

父仓库聚合前端 `AIChat-Android` 与后端 `AIChat-Server`（Git 子模块），提供账号密码认证、SSE 流式回复、多模态图片上传与历史记录等完整能力。

## 项目结构
- `app/`：Android 前端（MVVM + Compose + Room，OkHttp 直连，SSE 文本流）
- `server/`：Spring Boot 后端（账号密码认证、JWT、JPA/MySQL、SSE 流、健康检查）

## 环境准备
- 前端：Android 11+（API 30），Android Studio，Gradle，Kotlin
- 后端：Java 21，Maven（可用 `mvnw.cmd`），MySQL 8

## 后端配置与启动
- 配置文件 `server/aichat-server/aichat-server/src/main/resources/application.yaml`（可用环境变量覆盖）：
  - `server.port=8080`
  - 数据源：`DB_URL`、`DB_USER`、`DB_PASS`
  - JWT：`APP_JWT_SECRET`（必填，≥32 字符）、`APP_TOKEN_TTL_SEC`（默认 86400）
  - CORS：`CORS_ALLOWED_ORIGINS`
- 构建与启动（Windows）：
  ```bash
  ./mvnw.cmd -DskipTests package
  java -jar server/aichat-server/aichat-server/target/aichat-server-0.0.1-SNAPSHOT.jar
  ```
- 健康检查：`GET /health` → `{"status":"OK"}`

## 地址选择（联调）
- 本机浏览器：`http://127.0.0.1:8080`
- Android 模拟器（AVD）：`http://10.0.2.2:8080`
- Genymotion：`http://10.0.3.2:8080`
- 真机同一 Wi‑Fi：`http://<电脑IPv4>:8080`（`ipconfig` 查看）
- USB 直连真机：执行 `adb reverse tcp:8080 tcp:8080` 后用 `http://127.0.0.1:8080`

## 前端使用说明（综合）
- 在登录页填写账号与密码（注册需确认密码），设置“后端地址”，点击“测试连接”
- 登录/注册成功后进入聊天界面，支持：
  - SSE 流式增量回复（`Accept: text/event-stream`）
  - 模型切换与图片上传（MIME/大小校验，缩略图显示）
  - 历史记录持久化与分页（Room）
- 常见问题：
  - 连接失败：地址格式不合法、防火墙未放行 8080、真机未执行 `adb reverse`
  - 注册失败：两次密码不一致或长度 < 8；用户名冲突
  - 429/forbidden：并发或权限受限，稍后重试或检查令牌

## 后端认证与接口（综合）
- 注册：`POST /auth/register` Body `{"username","password"}` → `{"token","ttlSec","expiresAt"}`
- 登录：`POST /auth/login` Body 同上
- 鉴权：除 `/auth/*` 与 `/health` 外所有接口需 `Authorization: Bearer <token>`
- 错误统一结构：`{"code","message"}`，如 `UNAUTHORIZED`、`VALIDATION_ERROR`、`CONFLICT`
- SSE 流：`/stream`（需 `Bearer` 令牌；并发拦截配置在后端）
- 模型列表：`GET /models`

## 技术架构概览
- 前端（Android）：
  - 架构：MVVM + Compose + Room
  - 状态：ViewModel + StateFlow（SSE 增量拼接）
  - 网络：OkHttp + 全局认证拦截器
  - 关键文件：
    - `app/src/main/java/com/example/aichat10/viewmodel/AuthViewModel.kt`（认证与地址保存/测试）
    - `app/src/main/java/com/example/aichat10/viewmodel/ChatViewModel.kt`（SSE、模型、图片上传）
- 后端（Spring Boot）：
  - 技术栈：Spring Boot、Web/WebFlux、JPA/MySQL、JJWT、BCrypt、Actuator
  - 模块：`controller`、`config`（`WebConfig`/`GlobalExceptionHandler`）、`domain/repo`、`service`
  - 必填环境变量：`APP_JWT_SECRET`；可选：`APP_TOKEN_TTL_SEC`、`DB_URL/USER/PASS`、`CORS_ALLOWED_ORIGINS`

## 快速开始
1. 启动后端（设置 `APP_JWT_SECRET` 等）：
   ```bash
   ./mvnw.cmd -DskipTests package
   java -jar server/aichat-server/aichat-server/target/aichat-server-0.0.1-SNAPSHOT.jar
   ```
2. 运行前端：在 App 登录页配置后端地址，测试连接后注册/登录并开始对话

## 版本与变更（摘要）
- 登录方式：从短信验证码切换为账号密码注册/登录，返回 `token/ttlSec/expiresAt`
- 新增健康端点：`/health`；错误结构统一为 `code/message`
- 前端支持模型切换、图片上传、并发与错误提示优化

## 子模块说明
- 子模块在 `.gitmodules` 中配置为 `main` 分支；父仓库记录子模块指针。
- 更新到子模块最新：在父仓库执行 `git submodule update --remote --recursive`，再提交一次记录指针变更。
