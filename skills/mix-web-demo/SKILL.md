---
name: mix-web-demo
description: mix-web-demo 多语言微服务仓库专属编码规范，覆盖 Spring WebFlux、NestJS、FastAPI、GoZero 四个服务的分层组织、依赖注入、日志、常量、并行化、SQL 参数化与远程调用约定。在本仓库生成或修改任何服务代码时必须使用；与通用 skills 冲突时以本 skill 的项目实证约定为准。
---

# mix-web-demo 项目专属编码规范

## 适用范围与配套 skills

本仓库包含四个业务服务与一个网关，生成代码前必须先确认目标服务：

| 目录       | 技术栈                                                   | 端口 |
| ---------- | -------------------------------------------------------- | ---- |
| `spring/`  | Spring Boot 3 WebFlux + R2DBC + WebClient + Resilience4j | 8081 |
| `gozero/`  | go-zero 1.9 + goctl + sqlx                               | 8082 |
| `nestjs/`  | NestJS + Fastify + Mongoose/TypeORM + opossum            | 8083 |
| `fastapi/` | FastAPI + SQLAlchemy async + contextvars + APScheduler   | 8084 |
| `gateway/` | Apache APISIX（配置驱动，无业务代码）                    | 8080 |

配套通用 skills（本 skill 未覆盖的细节查阅它们，冲突时以本 skill 为准）：

**Spring / Java**

- `springboot-patterns`、`spring-boot-engineer`：Spring WebFlux 响应式模式
- `java-spring-boot`、`java-springboot`：Spring Boot 模块划分与最佳实践

**NestJS / Node.js**

- `nestjs-patterns`、`nestjs-best-practices`、`nestjs-expert`：NestJS 模块、DI 与架构模式
- `nodejs-backend-typescript`：Node.js + TypeScript 后端通用约定

**FastAPI / Python**

- `fastapi-patterns`、`fastapi-expert`、`fastapi-python`：FastAPI 依赖注入与异步模式
- `python-backend`：Python 后端安全与代码质量
- `sqlalchemy-postgres`：SQLAlchemy 2.0 + Pydantic + PostgreSQL（FastAPI 侧的 ORM 与 pgvector 访问）

**Go / GoZero**

- `golang-design-patterns`、`golang-pro`：Go 组合根与并发模式
- `golang-code-style`：Go 代码风格与可读性约定
- `golang-database`：Go 数据库访问（sqlx 参数化查询、事务、连接池）
- `golang-error-handling`：Go 错误创建、包装与结构化日志
- `golang-testing`：Go 表驱动测试与 testify
- `golang-security`：Go 安全实践（注入防护、密钥管理、日志安全）

**数据库**

- `database-schema-design`、`database-migration`：Schema 设计与迁移（MySQL / PostgreSQL / MongoDB）

**AI / LLM**

- `langchain`、`llm-application-dev-langchain-agent`：LangChain Agent、工具调用与多模型编排
- `rag`：RAG 检索增强（pgvector 向量库与语义检索）

**工程化**

- `devops`：Docker / Docker Compose 部署与 CI/CD

## 通用规则（全部服务）

1. 使用中文回复；代码注释与字符串信息使用中文；注释与说明不含任何 emoji 与标志
2. 不主动生成任何文档，除非明确要求
3. SQL 一律参数化，禁止字符串拼接用户值。各服务占位风格：Spring R2DBC 用 `:param`；FastAPI ClickHouse 用 `%(name)s`；GoZero sqlx 用 `?`；NestJS 走 ORM 方法 API
4. 常量抽取：四个服务均有常量类，消息类字符串必须进常量类引用
   - Spring：`common/constants/Messages.java`、`HttpCode.java`
   - NestJS：`common/constants/`（`Messages`、`HttpCode`、`Defaults`、`ErrorIds`）
   - FastAPI：`core/constants/`（`Messages`、`HttpCode`、`RedisKeys`、`Scripts`、`WarehouseScripts`）
   - GoZero：`app/common/constants/`（`messages.go`、`defaults.go`）
   - 语言支持字符串模板时（Python f-string、TS 模板串、Go fmt），动态部分用模板拼接常量，不把整句抽离
   - 注解 / 装饰器的字符串参数**不抽常量**，直接写字面量：这类字符串只在声明处使用一次，抽到常量类后需跳转查看，反而降低可读性
     - Spring：`@Operation(summary, description)`、`@Tag`
     - NestJS：`@ApiOperation({ summary, description })`、`@ApiTags`
     - FastAPI：`@router.get/post(..., summary=, description=)`
     - GoZero 无注解机制，不适用本条
5. 相互独立的 IO 调用（RPC/HTTP/DB/Redis/ES/MQ）必须并行化，禁止串行等待；各服务并行原语见对应章节
6. 日志遵循各服务现有封装（见各服务章节），禁止绕过封装直接 print/console/logx 散用
7. 远程调用统一走各服务封装的客户端，自动透传用户上下文头：`X-User-Id`、`X-Username`、`X-Session-Id`、`Authorization`、`X-Internal-Token`；无登录用户时内部令牌 `userId=-1` 表示系统调用。禁止在新代码里裸用 httpx/axios/http.Client 直连其他服务
8. 服务发现基于 Nacos；新服务接入需注册实例并在 metadata 声明能力
9. 生成代码时参考目标服务同类文件的命名与组织方式；已有成熟风格优先

## Spring 服务（spring/，WebFlux 响应式栈）

### 目录组织（三层 + 横切）

```
api/controller   接口层（返回 Mono/Flux）
api/service      业务接口
api/service/impl 业务实现
api/repository   R2DBC Repository
entity/po|vo|dto|projection  实体与视图对象
infra/client     远程调用客户端（ServiceWebClient）
infra/filter     WebFilter（UserContextWebFilter）
infra/handler    全局异常处理（GlobalExceptionHandler）
core/config|aspect|annotation|properties  配置、切面、注解、属性
common/constants|utils  常量与工具（RedisUtil、JwtUtil、UserContext）
```

### 硬性约定

- ORM 是 Spring Data R2DBC，不是 MyBatisPlus：Repository 继承 `ReactiveCrudRepository<T, ID>`；自定义 SQL 用 `@Query` + `:param`，更新加 `@Modifying`；事务用 `TransactionalOperator.transactional()` 包裹，禁止 `@Transactional`
- Controller 返回 `Mono<Result<T>>` / `Flux<Result<T>>`，Service 返回 `Mono<T>` / `Flux<T>`；禁止返回裸类型、禁止调用 `.block()`
- 请求上下文走 Reactor Context：`UserContextWebFilter` 用 `.contextWrite()` 写入，业务用 `Mono.deferContextual(ctx -> ...)` 读取 `UserContext.getUserId(ctx)`；禁止 `ThreadLocal`、`RequestContextHolder`、`OncePerRequestFilter`、`HandlerInterceptor`
- 并行独立 IO 用 `Mono.zip()` / `Mono.when()`，禁止 `CompletableFuture.allOf()`
- 阻塞操作（加密、验证码图片等）用 `Mono.fromCallable(() -> ...).subscribeOn(Schedulers.boundedElastic())` 隔离
- 全局异常：`@RestControllerAdvice` + `@ExceptionHandler` 返回 `Mono<ResponseEntity<Result<?>>>`，通过 ResponseEntity 设置 HTTP 状态码；`ResponseBodyAdvice` 在 WebFlux 不可用
- 微服务调用用 `ServiceWebClient`（WebClient + LoadBalancer + Resilience4j 熔断/重试），新增远程接口在该类中加方法，通过 `deferContextual` 注入上下文头；禁止 OpenFeign
- Redis 用 `ReactiveStringRedisTemplate`，封装在 `common/utils/RedisUtil`；缓存读写走 Mono 链式 `switchIfEmpty(Mono.defer(...))` 模式，禁止 `@Cacheable`
- AOP 切面中 `pjp.proceed()` 返回 `Mono<?>` 时必须在 Mono 链内操作（`.flatMap()` / `.doOnSuccess()`），上下文用 `.deferContextual()` 获取
- 注入风格：`@Service` + `@RequiredArgsConstructor` + `private final` 字段（构造器注入），禁止字段 `@Autowired`
- Swagger 用 springdoc：`@Operation(summary, description)`、`@Tag`，注解参数直接写字面量，不抽常量
- 日志：`common/utils/SimpleLogger` 实例注入使用

## NestJS 服务（nestjs/，Fastify + Mongoose/TypeORM）

### 目录组织

```
config/           YAML 配置与 index.ts 加载器
common/constants  常量（Messages、HttpCode、Defaults、ErrorIds）
common/exceptions 全局异常与错误结构
common/utils      工具类
framework/        全局横切：decorators、guards、interceptors、filters、middleware
module/common/    公共服务模块（nacos、logger、oss、redis、mail 等）
module/system/    系统业务模块（apiLog、articleLog、sqlTools 等）
```

### 模块内组织与约定

- 每个业务模块：`xxx.module.ts` + `xxx.controller.ts` + `xxx.service.ts`，辅以 `dto/` 与 `schema/`（Mongoose）或 `entities/`（TypeORM）
- ORM 按模块现状选择：日志类用 Mongoose（`@Schema` + `SchemaFactory`，`@InjectModel`）；结构化表用 TypeORM（`@InjectRepository`）。新模块先参考同类模块，禁止凭空引入新 ORM
- 请求上下文用 nestjs-cls：`ClsMiddleware` 写入（userId/username/sessionId/token/internalToken），业务注入 `ClsService` 读取；gRPC 或脱离 HTTP 异步链的场景必须用 `ClsService.run` 手动开启上下文
- 远程调用统一走 `module/common/nacos/nacos.service.ts` 的 `call(opts: CallOptions)`（Nacos 发现 + opossum 熔断 + 自动上下文头/内部令牌），按目标服务在调用方封装 Client 类
- 独立异步调用用 `Promise.all` 并行；条件不满足时用 `Promise.resolve(空值)` 占位保证数组结构一致
- 并行调用中存在「部分失败可容忍」时用 `Promise.allSettled`，逐项判断 `status` 决定降级或抛出，避免单个失败拖垮整体；不要用 `Promise.all` + `catch` 吞异常来模拟降级，会丢失失败项与输入项的对应关系
  - 批量任务逐项降级（参考 `module/system/sqlTools/sqlTools.service.ts` 并行查多表行数）：`results.map((r, i) => r.status === "fulfilled" ? r.value : 降级值)`，用索引 `i` 关联回输入项
  - 多个独立调用中区分必选与可选（参考 `module/common/github/github.service.ts` 并行拉 profile 与 email）：解构后分别判断，必选项 `rejected` 直接 `throw reason`，可选项 `fulfilled` 才取值、否则走兜底
- 尽可能使用 TypeScript 类型标注（接口、泛型、联合类型），即使编译通过也补全类型
- 遵循项目 `eslint.config.mjs`，提交前通过 lint
- Swagger 用 `@nestjs/swagger`：`@ApiOperation({ summary, description })`、`@ApiTags`，注解参数直接写字面量，不抽常量
- 日志：注入 `LoggerService`（`module/common/logger`）

## FastAPI 服务（fastapi/）

### 目录组织

```
app/core/         基础设施：auth、base(Logger)、client(远程调用)、config、constants、db、errors
app/common/       decorators、middleware（contextvars 上下文中间件）
app/internal/api        路由（按业务分 router）
app/internal/services   业务服务（类组织 + 工厂函数）
app/internal/crud       数据访问 Mapper（含 ClickHouse 查询）
app/internal/models     SQLAlchemy 模型
app/internal/schemas    Pydantic 模型
app/internal/clients    按服务封装的远程客户端（SpringClient、NestjsClient、GozeroClient）
app/internal/cache      两级缓存（L1 内存 + L2 Redis + 版本号失效）
app/internal/tasks      APScheduler 调度任务（scheduler.py + logic/）
app/internal/agents     LangChain Agent 与工具
```

### 硬性约定

- 依赖注入：`app/dependencies.py` 集中定义 `Annotated` 别名（`DbSession`、`XxxServiceDep`）；服务用类组织功能方法，配 `@lru_cache()` 工厂函数 `get_xxx_service()` 供 `Depends` 使用；新服务必须在 `dependencies.py` 注册别名
- 全异步：接口必须 `async def`；优先用 asyncio 生态；同步阻塞库（clickhouse_driver、连接池同步接口）用 `asyncio.to_thread(...)` 包裹后 `await`
- 并行：独立 IO 用 `asyncio.gather`；循环内并行把每次迭代封装为内部 async 函数后 gather
- `__init__.py` 导出：包内有 `__init__.py` 导出的功能，导入一律走包路径（`from app.internal.crud import UserAnalysisMapper`），禁止深入到文件路径；新增模块必须同步更新 `__init__.py` 导出
- 类型标注全覆盖；`import` 全部在文件顶部，禁止逻辑中导入；生成后检查并删除未使用的 import
- SQL：ClickHouse 查询用 `%(name)s` 参数化；SQL 模板集中在 `core/constants/scripts.py` 与 `warehouse.py`，业务代码不内联 SQL 字符串
- 缓存：使用 `internal/cache` 的两级缓存体系（L1 内存 5 分钟 + L2 Redis 1 天 + ClickHouse 版本号失效），新统计接口优先接入而非自造缓存
- 定时任务：APScheduler 注册在 `internal/tasks/scheduler.py`，任务逻辑放 `tasks/logic/`；多实例互斥用 Redis `try_lock`/`unlock`（key 进 `RedisKeys` 常量）
- 远程调用：统一 `core/client/call_remote_service`（httpx 共享连接池 + SimpleCircuitBreaker 熔断 + tenacity 重试 + 上下文头自动注入），按服务在 `internal/clients/` 封装 Client 类
- 日志：`from app.core.base import Logger`，`Logger.info/warning/error/debug`
- 上下文：`common/middleware/contextMiddleware.py` 的 contextvars（`get_current_user_id` 等），深层函数免参读取用户信息

## GoZero 服务（gozero/，API-First）

### API-First 流程（强制）

1. 先在 `gozero/api/` 修改 `.api` 文件：`main.api` 按业务分组 import（chat、search、sqlTools、task 等），路由写在分组文件中，`@server` 块声明 `group`、`prefix`、`middleware`
2. 生成代码：`script/goctl/genApi.ps1`（内部执行 `goctl api go -api main.api -dir ../app --style=goZero --home ../template`）
3. ORM：`script/goctl/genOrm.ps1`（`goctl model mysql ddl --style goZero --home ../template`），SQL DDL 放 `script/sql/`
4. 模板在 `gozero/template/`（api、model、mongo、newapi 等），生成代码基于模板；禁止手写 handler/types 绕过生成流程，logic 文件头部有 `// Code scaffolded by goctl. Safe to edit.` 标记可编辑

### 目录组织与硬性约定

```
app/internal/boot        启动（Run/CreateServer）
app/internal/config      配置结构体
app/internal/handler     goctl 生成的路由与 handler（routes.go 挂中间件）
app/internal/logic       业务逻辑（按业务分目录：chat、search、sqlTools、task）
app/internal/middleware  中间件（usercontext、internalservice、recovery、apilog）
app/internal/svc         ServiceContext 组合根（分域上下文）
app/internal/client      按服务封装的远程客户端（fastapiClient、nestjsClient、springClient）
app/internal/types       请求响应类型（goctl 生成）
app/common/constants     常量（messages.go、defaults.go）
app/common/keys          context key（未导出类型）
app/common/client        ServiceDiscovery 统一远程调用
app/model/<table>        数据模型（goctl 生成 _gen.go + custom 扩展文件）
```

- logic 中禁止 SQL：数据访问全部封装在 `app/model/`，logic 通过 `l.svcCtx.XxxModel.Method(l.ctx, ...)` 调用；model 方法第一个参数是 `ctx`
- svc 分域：`ServiceContext` 匿名嵌入 `RuntimeContext`、`InfrastructureContext`、`ModelContext`、`HubContext`、`ClientContext`、`LoggerContext`、`MiddlewareContext`；新增依赖加入对应分域，在 `serviceComponentsContext.go` 组装，禁止往 ServiceContext 平铺字段
- 日志：logic 结构体嵌入 `*utils.ZeroLogger`（构造时 `ZeroLogger: svcCtx.Logger.WithContext(ctx)`），调用 `l.Info/l.Errorf/l.Error`；logx 全局方法仅限启动阶段（logx 无 Warn/Warnf）；项目 ZeroLogger 提供 `Warningf`，警告级日志用它，异常一律 `l.Errorf`
- 并行：`mr.Finish`，每个任务为 `func() error`，结果写入闭包局部变量，任务内部吞错返回 nil（错误在任务外统一处理）
- context：一切可能阻塞的函数第一个参数接收 `context.Context`，嵌套调用透传同一 ctx；logic 用 `l.ctx`
- 中间件：`.api` 文件 `@server middleware:` 声明 + `routes.go` 的 `rest.WithMiddlewares`；内部服务接口必须挂 `InternalServiceMiddleware`（校验 `X-Internal-Token`）
- 远程调用：`common/client/ServiceDiscovery.CallService(ctx, serviceName, path, RequestOptions)`（Nacos 发现 + 重试 + 上下文头/内部令牌自动注入），按目标服务在 `internal/client/` 封装 Client 结构体
- 常量：消息进 `app/common/constants/messages.go`，动态部分 `fmt.Sprintf(constants.XXX+": %v", err)` 拼接
- 构建校验：`go build ./...` 必须通过；检查未使用导包与变量；产物命名与入口一致

## 验证命令

| 服务    | 验证                                                                    |
| ------- | ----------------------------------------------------------------------- |
| spring  | `mvn compile -f spring/pom.xml`（打包 `mvn clean package -DskipTests`） |
| nestjs  | `npm run node:build`（nestjs 目录）                                     |
| fastapi | `.venv/Scripts/python.exe -c "import app"` 级导入校验 + pytest          |
| gozero  | `go build ./...`（gozero/app 目录）                                     |
