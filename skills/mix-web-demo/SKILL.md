---
name: mix-web-demo
description: mix-web-demo 多语言微服务仓库专属编码规范，覆盖 Spring WebFlux、NestJS、FastAPI、GoZero 四个服务与 APISIX 网关的分层组织、依赖注入、请求校验、实时链路、日志、测试、常量、并行化、SQL 参数化、远程调用与 Swagger 生成约定，并含仓库工程约束（CRLF 行尾、goctl 生成流程）。在本仓库生成或修改任何服务代码时必须使用；与通用 skills 冲突时以本 skill 的项目实证约定为准。
---

# mix-web-demo 项目专属编码规范

## 适用范围与配套 skills

本仓库包含四个业务服务与一个网关，生成代码前必须先确认目标服务：

| 目录       | 技术栈                                                   | 端口 |
| ---------- | -------------------------------------------------------- | ---- |
| `spring/`  | Spring Boot 3 WebFlux + R2DBC + WebClient + Resilience4j | 8081 |
| `gozero/`  | go-zero 1.10 + goctl 1.9.2 + sqlx                        | 8082 |
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

## 按任务加载参考规范

- 创建、修改、删除或评审测试，以及修复需要回归测试的缺陷时，必须完整读取 [references/unit-testing.md](references/unit-testing.md)
- 测试规范只约束测试边界与质量，不要求为了覆盖率测试生成代码、DTO、getter 或第三方库内部行为

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
10. 注释说明：注释的结束不能包含中文句号，直接留空，如果注释过长，使用多行注释形式，而不是多条单行注释，短注释使用1行的单行注释即可
11. **日志采集必须排除敏感字段**：请求体进入日志后会被投递到 `api-log-queue`，最终落在 MongoDB `apilogs` 与 ClickHouse `ods_api_log`，因此凡是携带密码、验证码、令牌、授权码的接口都要显式排除。各服务能力：Spring `@ApiLog(excludeFields = {...})`、NestJS `@ApiLog({ excludeFields: [...] })`、FastAPI `@logWithConfig(exclude_fields = [...])`（`@log` 不支持）；**GoZero 的 `ApplyApiLog` 没有任何排除能力**，涉及凭据的接口不要挂它，或先给中间件补过滤参数

## Spring 服务（spring/，WebFlux 响应式栈）

### 目录组织（三层 + 横切）

```
api/controller   接口层（返回 Mono/Flux）
api/service      业务接口
api/service/impl 业务实现
api/repository   R2DBC Repository
entity/po|vo|dto|projection  实体与视图对象
entity/assembler 实体到 VO 的装配器（依赖 api/repository，供 service/impl 复用，跨包调用需 public）
infra/client     远程调用客户端（ServiceWebClient）
infra/filter     WebFilter（UserContextWebFilter）
infra/handler    全局异常处理（GlobalExceptionHandler）
core/config|aspect|annotation|properties  配置、切面、注解、属性
common/constants|utils  常量与工具（RedisUtil、JwtUtil、UserContext）
```

### 硬性约定

- **依赖方向**：`api` 是最上层，`infra` / `core` / `common` / `entity` 都不依赖它；目前唯一例外是 `entity/assembler`（装配器需要 `api/repository` 才能加载关联数据）。新增跨层组件前先确认会不会引入新的反向依赖
- ORM 是 Spring Data R2DBC，不是 MyBatisPlus：Repository 继承 `ReactiveCrudRepository<T, ID>`；自定义 SQL 用 `@Query` + `:param`，更新加 `@Modifying`；事务用 `TransactionalOperator.transactional()` 包裹，禁止 `@Transactional`
- Controller 返回 `Mono<Result<T>>` / `Flux<Result<T>>`，Service 返回 `Mono<T>` / `Flux<T>`；禁止返回裸类型、禁止调用 `.block()`
- 请求上下文走 Reactor Context：`UserContextWebFilter` 用 `.contextWrite()` 写入，业务用 `Mono.deferContextual(ctx -> ...)` 读取 `UserContext.getUserId(ctx)`；禁止 `ThreadLocal`、`RequestContextHolder`、`OncePerRequestFilter`、`HandlerInterceptor`
- 并行独立 IO 用 `Mono.zip()` / `Mono.when()`，禁止 `CompletableFuture.allOf()`
- 阻塞操作（加密、验证码图片等）用 `Mono.fromCallable(() -> ...).subscribeOn(Schedulers.boundedElastic())` 隔离
- 全局异常：`@RestControllerAdvice` + `@ExceptionHandler` 返回 `Mono<ResponseEntity<Result<?>>>`，通过 ResponseEntity 设置 HTTP 状态码；`ResponseBodyAdvice` 在 WebFlux 不可用
- **WebFlux 的请求体校验失败抛的是 `WebExchangeBindException`（继承 `ServerWebInputException`），不是 MVC 的 `MethodArgumentNotValidException` / `org.springframework.validation.BindException`**。`@ExceptionHandler` 未声明它就会被 `@ExceptionHandler(Exception.class)` 兜底吞成 500，必须显式声明；畸形 JSON 与参数类型不匹配抛 `ServerWebInputException`，一并声明。字段消息取 `getBindingResult().getFieldError().getDefaultMessage()`
- 请求 DTO 用 jakarta 注解声明约束（放 `entity/dto/`），控制器参数用 `@Valid` / `@Validated` 触发。控制器**没有**类级 `@Validated`，查询参数与路径变量目前没有方法级约束校验，需要时在 service 层手工校验
- 微服务调用用 `ServiceWebClient`（WebClient + LoadBalancer + Resilience4j 熔断/重试），新增远程接口在该类中加方法，通过 `deferContextual` 注入上下文头；禁止 OpenFeign
- Redis 用 `ReactiveStringRedisTemplate`，封装在 `common/utils/RedisUtil`；缓存读写走 Mono 链式 `switchIfEmpty(Mono.defer(...))` 模式，禁止 `@Cacheable`
- AOP 切面中 `pjp.proceed()` 返回 `Mono<?>` 时必须在 Mono 链内操作（`.flatMap()` / `.doOnSuccess()`），上下文用 `.deferContextual()` 获取
- 横切能力由 AOP 实现：`ApiLogAspect`（接口日志）、`PermissionValidationAspect`（`@RequirePermission` 声明式权限）、`InternalTokenAspect`（`@RequireInternalToken` 内部令牌）、`ArticleSyncAspect`（文章变更同步 MQ / ES / 向量）、`Neo4jSyncAspect`（图谱同步）。新增写接口时确认是否需要权限注解与同步触发，同步失败只记日志、没有补偿机制
- **响应式切面的固定写法**：`pjp.proceed()` 返回的是**未订阅的冷流**（业务代码此时尚未执行），且 Reactor Context 只在订阅时可见。所以切面必须把校验 / 日志 / 同步挂到 Mono 链上（`Mono.deferContextual(ctx -> ...)`）：校验类用 `validate(ctx).then(businessMono)` 保证「校验先于业务」，副作用类用 `monoResult.doOnSuccess(...)` 发后即忘；副作用里若需要用户身份，必须用 `UserContext.writeContext(Context.empty(), ...)` 重建 Context 再 `contextWrite`，否则异步链上读不到。**禁止用同步代码在 `proceed()` 之前读上下文**，那时 ctx 不可见，只会拿到 null
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
module/common/    公共服务模块（nacos、logger、oss、redis、mail、github、task、client）
module/system/    系统业务模块（apiLog、articleLog、sqlTools 等）
```

### 模块内组织与约定

- 每个业务模块：`xxx.module.ts` + `xxx.controller.ts` + `xxx.service.ts`，辅以 `dto/` 与 `schema/`（Mongoose）或 `entities/`（TypeORM）
- ORM 按模块现状选择：日志类用 Mongoose（`@Schema` + `SchemaFactory`，`@InjectModel`）；结构化表用 TypeORM（`@InjectRepository`）。新模块先参考同类模块，禁止凭空引入新 ORM
- 入参校验走全局 `ValidationPipe`（`app/index.ts` 注册，`transform` + `enableImplicitConversion`），DTO 用 class-validator 注解；自定义 `exceptionFactory` 把错误包成 `BusinessException(400, PARAM_PARSE_FAILED)`，最终由 `AllExceptionsFilter` 输出统一结构。**当前未开 `whitelist` / `forbidNonWhitelisted`**，未声明字段既不剥离也不报错，不要依赖"多余字段会被拒"
- 请求上下文用 nestjs-cls：`ClsMiddleware` 写入（userId/username/sessionId/token/internalToken），业务注入 `ClsService` 读取；gRPC 或脱离 HTTP 异步链的场景必须用 `ClsService.run` 手动开启上下文
  - **未登录语义注意**：`parseUserId` 要求 `Number.isInteger(userId) && userId > 0`，所以 ≤0 的值（含约定的系统调用 `-1`）在 CLS 里是 `undefined`；出站头由 `NacosService.call` 生成（`X-User-Id: String(userId || 0)`，内部令牌按 `userId > 0 ? userId : -1`）。需要区分"未登录 / 系统调用"时不要直接依赖 CLS 的值
- 公共层入口是 `module/common/common.module.ts`：`imports` 只保留需要根级加载的子模块（Logger、Client、Github、Mail、Task），`exports` **只导出 ClientModule**（根模块的 `RequireAdminGuard` 需要它）。业务模块用到的其他公共能力（oss、word、nacos、redis）由消费方**自行 import 对应模块**，禁止把 CommonModule 做成"全量中转池"（那会让模块边界形同虚设）
- `LoggerModule` 是 `@Global`，日志能力全模块可直接注入，无需 import
- 用户可控字符串在拼接存储 key 前必须清洗（长度 + 字符白名单）：`upload` 模块的 `customFilename` 走 `@Query` 原样拼 OSS key，属需要加固的写法
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

- 依赖注入分三层，各有固定登记处：
  - 定义层 `internal/{clients,crud,cache}/` 各文件底部：类 + 构造 + `@lru_cache` 工厂三位一体
  - 注册层 `app/dependencies/` 包（`database.py`、`mappers.py`、`caches.py`、`clients.py`、`services.py`、`tools.py`、`llm.py`）：集中定义 `Annotated` 别名（`DbSession`、`XxxServiceDep`、`XxxToolDep`），由包 `__init__.py` 统一导出
  - 装配层 `dependencies/services.py` / `tools.py` / `llm.py`：`provide_xxx(...)` 函数，**参数即依赖别名**，函数体调用底层工厂；服务与工具的构造依赖必须由这里显式传入，构造函数内不得自建依赖
- 注册范围只含**路由函数签名里直接出现**的依赖（DB 会话 + 服务 + agent 工具）；Client / Mapper / Cache 工厂不注册，只被 `provide_*` 与 `get_*_service` 消费
- `@lru_cache` 工厂是承重墙：`provide_*` 每次请求都会执行，底层工厂漏 `@lru_cache` 会让 `get_xxx_service` 的缓存按新键无界累积
- 非请求链路（APScheduler 任务）用 `resolve_xxx_service()` 复用同一组单例工厂，禁止 `cls(...)` 直接 new（会造出第二个实例、内部状态不共享）
- 全异步：接口必须 `async def`；优先用 asyncio 生态；同步阻塞库（clickhouse_driver、连接池同步接口）用 `asyncio.to_thread(...)` 包裹后 `await`
- 并行：独立 IO 用 `asyncio.gather`；循环内并行把每次迭代封装为内部 async 函数后 gather
- `__init__.py` 导出：包内有 `__init__.py` 导出的功能，导入一律走包路径（`from app.internal.crud import UserAnalysisMapper`），禁止深入到文件路径；新增模块必须同步更新 `__init__.py` 导出
- 类型标注全覆盖；`import` 全部在文件顶部，禁止逻辑中导入；生成后检查并删除未使用的 import
- SQL：ClickHouse 查询用 `%(name)s` 参数化；SQL 模板集中在 `core/constants/scripts.py` 与 `warehouse.py`，业务代码不内联 SQL 字符串
- 缓存：使用 `internal/cache` 的两级缓存体系（L1 内存 5 分钟 + L2 Redis 1 天 + ClickHouse 版本号失效），新统计接口优先接入而非自造缓存
- 定时任务：APScheduler 注册在 `internal/tasks/scheduler.py`，任务逻辑放 `tasks/logic/`；多实例互斥用 Redis `try_lock`/`unlock`（key 进 `RedisKeys` 常量）
- 远程调用：统一 `core/client/call_remote_service`（httpx 共享连接池 + SimpleCircuitBreaker 熔断 + tenacity 重试 + 上下文头自动注入），按服务在 `internal/clients/` 封装 Client 类
- ClickHouse ORM 会话的唯一入口是 `core/db/clickhouse.py` 的 `clickhouse_session()`（`get_clickhouse_db` 复用它）。**粒度保持"每查询一个会话"**：`crud/user.py` 有 `asyncio.gather` 并发跑两条 CH 查询，而 `AsyncSession` 不支持并发复用；非请求链路用引擎级 `execute_clickhouse_query` / `execute_clickhouse_sql`
- Agent 工具经 `internal/agents/toolFactories.py` 的 `AgentToolFactories` 注入 `BaseAiService`（**传工厂而非实例**，以保留线程池并行加载与单组失败隔离）
- Agent 工具权限：`internal/agents/toolScope.py` 的 contextvars 作用域（**fail-closed**）——`intentRouter` 在路由前统一写入 `ToolScope(user_id, is_admin)`，SQL 与 MongoDB 工具执行前校验行级范围（查询必须含 `user_id` 且绑定值等于当前用户，admin 放行）。禁止把请求态写进 `@lru_cache` 单例的实例字段
- 日志：`from app.core.base import Logger`，`Logger.info/warning/error/debug`
- 上下文：`common/middleware/contextMiddleware.py` 的 contextvars（`get_current_user_id` 等），深层函数免参读取用户信息

## GoZero 服务（gozero/，API-First）

### API-First 流程（强制）

1. 先在 `gozero/api/` 修改 `.api` 文件：`main.api` 按业务分组 import（chat、search、sqlTools、task 等），路由写在分组文件中，`@server` 块声明 `group`、`prefix`、`middleware`
2. 生成代码：`gozero/script/goctl/genApi.sh`（bash）或 `genApi.ps1`（PowerShell），两者等价，顶层入口 `scripts/goctl-api-init.sh`。核心动作是 `goctl api format -dir .` + `goctl api go -api main.api -dir ../app --style=goZero --home ../template`；脚本会自动备份并还原 `main.go` 与 `etc/`、删除生成的 `app.go`
3. ORM：`gozero/script/goctl/genOrm.sh` / `genOrm.ps1`（`goctl model mysql ddl --style goZero --home ../template`），SQL DDL 放 `gozero/script/sql/`
4. 模板在 `gozero/template/`（api、model、mongo、newapi 等），生成代码基于模板；禁止手写 handler/types 绕过生成流程，logic 文件头部有 `// Code scaffolded by goctl. Safe to edit.` 标记可编辑
5. **生成后必须处理的四件事**：
   - `goctl` 会把 `.api`、`types.go`、`routes.go` 的行尾统一转成 LF，仓库约定 CRLF，需逐文件转回
   - import 路径若发生变更，import 块的字母序会失效（`gofmt -l` 会报未格式化），按「剥离 CRLF → gofmt 重排 → 还原 CRLF」处理
   - 已存在的 handler / logic 会被跳过（`exists, ignored generation`），但 `template/api/handler.tpl` 里的 `ApplyApiLog(..., "TODO: 添加接口描述")` 是占位字面量（仓库所有 handler 都手工换成了 `constants.API_LOG_*`），所以「重新生成」只对**缺失**文件安全
   - 生成后跑 `cd gozero/app && go build ./...` 确认模板不变量未被破坏

### 目录组织与硬性约定

```
app/internal/boot        启动（Run/CreateServer）
app/internal/config      配置结构体
app/internal/handler     goctl 生成的路由与 handler（routes.go 挂中间件）
app/internal/logic       业务逻辑（按业务分目录：chat、search、sqlTools、task）
app/internal/middleware  中间件（usercontext、internalservice、recovery、apilog）
app/internal/svc         ServiceContext 组合根（分域上下文 + dependencyManifest 依赖清单校验）
app/internal/client      按服务封装的远程客户端（fastapiClient、nestjsClient、springClient）
app/internal/types       请求响应类型（goctl 生成的 types.go + 手写 <domain>Validate.go）
app/internal/hub         实时通信（chatHub、sseHub、chatRealtimeDispatcher、realtimeTypes）
app/internal/task        定时任务（cron 调度器 + logic/esSyncerTask）
app/common/constants     常量（messages.go、defaults.go、validations.go、httpCode.go、sqlTools.go、scripts.go、redisKeys.go）
app/common/keys          context key（未导出类型）
app/common/client        ServiceDiscovery 统一远程调用
app/common/exceptions    业务异常类型（BadRequest / InternalServerError 等）
app/common/pubsub        Redis Pub/Sub 跨实例广播（redisPubSub.go）
app/common/utils         ZeroLogger 与工具（response、redisLock、internalToken、safeGo）
app/model/<table>        数据模型（goctl 生成 _gen.go + custom 扩展文件）
```

- **分层方向单向**：`internal/*` 可依赖 `common/*`，`common/*` **不得**依赖 `internal/*`。实时帧类型定义在 `.api` → `internal/types`，hub 因此放在 `internal/hub`；若放 `common/hub` 就会产生反向依赖
- logic 中禁止 SQL：数据访问全部封装在 `app/model/`，logic 通过 `l.svcCtx.XxxModel.Method(l.ctx, ...)` 调用；model 方法第一个参数是 `ctx`
- svc 分域：`ServiceContext` 匿名嵌入 `RuntimeContext`、`InfrastructureContext`、`ModelContext`、`HubContext`、`ClientContext`、`LoggerContext`、`MiddlewareContext`；新增依赖加入对应分域，在 `serviceComponentsContext.go` 组装，禁止往 ServiceContext 平铺字段
- **定时任务调度器挂在 `RuntimeContext.TaskScheduler`**（2026-09-16 从包级变量收口）：`internal/task` 只提供 `NewTaskScheduler(svcCtx) *cron.Cron` 负责构造并启动，返回值由 `boot/server.go` 赋给 `ctx.TaskScheduler`；停止统一由 `ServiceContext.Close()` 承担（先 `TaskScheduler.Stop()` 再 `Cancel()`），`boot/init.go` 的关闭入口调 `ctx.Close()` 而非直接操作调度器。**不要再引入包级调度器变量**（测试无法替换、生命周期不受 Close 管理）。cron 表达式用标准 5 字段（分 时 日 月 周），"每小时"是 `0 * * * *`，写成 `* * */1 * *` 会变成每分钟执行
- `ServiceContext.Close()` 是唯一的关闭入口（此前长期无人调用，2026-09-16 起由 `boot` 调用）；新增需要释放的资源时把释放逻辑加进 `Close()`，不要另建包级 stop 函数
- 日志：logic 结构体嵌入 `*utils.ZeroLogger`（构造时 `ZeroLogger: svcCtx.Logger.WithContext(ctx)`），调用 `l.Info/l.Errorf/l.Error`；logx 全局方法仅限启动阶段（logx 无 Warn/Warnf）；项目 ZeroLogger 提供 `Warningf`，警告级日志用它，异常一律 `l.Errorf`
- 并行：`mr.Finish`，每个任务为 `func() error`，结果写入闭包局部变量，任务内部吞错返回 nil（错误在任务外统一处理）
- context：一切可能阻塞的函数第一个参数接收 `context.Context`，嵌套调用透传同一 ctx；logic 用 `l.ctx`
- 中间件：`.api` 文件 `@server middleware:` 声明 + `routes.go` 的 `rest.WithMiddlewares`；内部服务接口必须挂 `InternalServiceMiddleware`（校验 `X-Internal-Token`）
- 远程调用：`common/client/ServiceDiscovery.CallService(ctx, serviceName, path, RequestOptions)`（Nacos 发现 + 重试 + 上下文头/内部令牌自动注入），按目标服务在 `internal/client/` 封装 Client 结构体
- 常量：消息进 `app/common/constants/messages.go`，动态部分 `fmt.Sprintf(constants.XXX+": %v", err)` 拼接
- 构建校验：`cd gozero/app && go build ./... && go vet ./... && go test ./... -count=1` 必须通过；检查未使用导包与变量；产物命名与入口一致；格式用「剥离 CRLF → `gofmt -l`」复核，禁止直接 `gofmt -w`

### 请求参数校验

- **只用一种机制：手写 `Validate() error`**，放 `internal/types/<domain>Validate.go`（现有 `chatValidate.go`、`searchValidate.go`、`sqlToolsValidate.go`、`taskValidate.go`），由 `httpx.Parse` 通过 go-zero 的 `validation.Validator` 接口自动调用，handler 里模板还会再显式调一次
- **每个请求类型都必须有 `Validate()`**：`template/api/handler.tpl` 对任何带请求类型的路由都会无条件生成 `req.Validate()`，缺了就编译不过（空请求体类型也补一个返回 nil 的方法）。SSE / WebSocket 走 `template/api/sse_handler.tpl`，不生成该调用
- **不要使用结构体 `validate` 标签**（`httpx.SetValidator` + go-playground/validator 方案已试过并回退）：接口判断优先，给已有 `Validate()` 的类型加标签会静默失效，同一类型两种机制互斥
- `Validate()` 只做请求边界检查（非空、长度、数值范围、格式）；业务与安全规则留在 logic（如 `sqlToolsQueryLogic.validateQuery` 的只读前缀白名单、表名白名单、强制 LIMIT + 上限），避免同一规则两处维护
- SSE / WebSocket 的 `Validate()` 只校验「若提供 `user_id` 则必须 > 0」（**允许缺省**：EventSource 与 WebSocket 握手无法自定义请求头，身份可能来自网关透传的 `X-User-Id`）；身份解析在 logic 的 `ResolveUserID`，handler 只负责接管连接

### 实时链路（SSE / WebSocket）

- **帧结构的单一来源是 `.api`**：`ChatSSEMessage`、`ChatWsMessage` 声明在 `api/chat/stream.api`，接口的 `returns` 直接指向它们，**不要**再建空的 `XxxConnectResp` 占位类型。goctl 生成类型用 `Id / UserId / SenderId / ReceiverId / MessageId` 命名，与手写结构旧的 `ID / UserID / ...` 不同，跨用时需改字段名并把 `uint` 改 `uint64`
- hub 与实时分发在 `internal/hub`（`chatHub`、`sseHub`、`chatRealtimeDispatcher`、`realtimeTypes`）：本机连接管理 + 跨 Pod 事件分发。内部信封 `ChatRealtimeEvent` 只在对内使用，**不进 `.api`**
- **不要采用 `sse_handler.tpl` 的「每请求 chan + handler 内 flush 循环」单机模式**：跨实例广播依赖 `internal/hub` + `common/pubsub/redisPubSub.go` 的 Redis Pub/Sub，照模板改会丢掉多副本下的消息投递。连接接管（`HandleConnection` / `upgrader.Upgrade`）必须留在 handler，且接管前不得写 `w`
- **`common/pubsub.RedisPubSub` 是「一个实例一个频道」**（2026-09-16 由 `common/realtime` 迁移并参数化）：频道在 `NewRedisPubSub(client, logger, channel)` 构造时绑定，`Publish` / `Start` 都不收频道参数。内部只持有单个 `*redis.PubSub` 引用，所以**同一个实例不能订阅第二个频道**（会覆盖前一个订阅的引用、造成连接泄漏与丢消息）。需要新频道就新建实例，频道常量统一放 `common/constants/redisKeys.go`
- 心跳与初始帧属协议内容：SSE 连接建立时下发 `{"type":"connected"}`，WS 收到 `{"type":"ping"}` 回 `{"type":"pong"}`

### 搜索链路

- 流程是「召回 → 融合重排 → 切页」：`resolveRecallWindow(page, size)` 把 `page * size` 按 `SEARCH_RECALL_STEP_SIZE`（100）向上取整成 `recallSize`，ES 以 `Page=1, Size=recallSize` 取一档候选；融合后由 `pageSlice` 在内存切出目标页，`total` 始终取 ES 的 `TotalHits`
- `page * size` 超过 `SEARCH_RECALL_MAX_SIZE`（200）时退化为窗口内重排并打 `SEARCH_RECALL_DEGRADE_LOG`
- 融合归一化分母取**整个召回候选集**的最大 ES 分（同一档位内各页可比）；未被向量或图谱召回的候选用**候选集均值**填充，权重保持全局固定。禁止把缺失信号的权重按比例重分配给其余信号
- 两层权重分开看：ES 内 `script_score` 权重（`ESScoreWeight` / `ViewsWeight` / `RecencyWeight` 等）管第一层，`FusionConfig`（`VectorScoreWeight` / `GraphScoreWeight` / `HybridMinESWeight`）管第二层
- ES 搜索脚本、权重、参数名映射三件套任一拉取失败 → 整条增强链路关闭，降级为普通 ES 条件分页查询（无脚本时补 `create_at` 降序，避免 `_score` 恒为 0 时顺序不确定）；ES 召回后回填 Spring 统计（views / likes / collects / follows）四路任一失败 → 该指标保留 ES 原值，不抛错
- 两侧候选上限必须对齐：`SEARCH_VECTOR_CANDIDATE_LIMIT` / `SEARCH_GRAPH_CANDIDATE_LIMIT` ≥ 召回上限，FastAPI 侧 `VECTOR_SEARCH_CANDIDATE_LIMIT` / `GRAPH_SEARCH_CANDIDATE_LIMIT` 同步

### 错误处理与依赖

- **禁止用错误文案做控制流**（重试 / 降级判定）：用类型化错误 + `errors.As`。`common/client/clientErrors.go` 的 `httpStatusError{StatusCode, Body}` 承载下游状态码，`shouldRetry` 集中判定（context 取消/超时、5xx、超时、拨号失败、EOF → 可重试；4xx、业务错误、读连接被重置 → 不重试）
- **平台约束**：Windows 的 `syscall` 只声明 `WSAE*`，没有 `syscall.ECONNREFUSED` 等哨兵，网络错误判定用 `net.OpError.Op == "dial"` 与 `net.Error.Timeout()`
- 依赖清单：`internal/svc/dependencyManifest.go` 的 `validateDependencies` 在基础设施装配后执行——`nacos` / `mysql` / `elasticsearch` 缺失直接 panic 拒绝启动，`rabbitmq` / `redis` / `mysql_raw` 缺失聚合一条降级警告。新增必需依赖需在清单登记

### Swagger 文档

- 产物在 `app/docs/`：`main.json` 被 `docs/embed.go` **嵌入二进制**（改后需重新编译并重启才生效），`main.yaml` 不嵌入
- 生成流程 `gozero/script/swagger/genSwagger.sh`：`goctl api swagger --api main.api --dir ../app/docs`（+ `--yaml`）→ `swagger2openapi -o main.json -p main.json` → `python fix.py main.json main.yaml`（后处理需要 PyYAML）
- `fix.py` 是生成后修补产物的固定环节：中文标签、info、servers 修正，以及长连接接口的响应语义（`/sse/chat` 改 `text/event-stream` 并复用 goctl 生成的 inline schema、`/ws/chat` 改 `101 Switching Protocols`、两者补 `400`）。**该文档的 schema 全部是 inline（`components` 下没有 `schemas`），不要改成 `$ref`**

## 网关（gateway/，Apache APISIX）

- 以 YAML 数据面运行（`role_data_plane` + `config_provider: yaml`），无业务代码；路由与插件全部在 `apisix/apisix.yaml`
- **认证外置**：业务路由挂 `forward-auth` 回调 Spring 的 `/users/internal/auth/validate`，由网关注入 `X-User-Id` / `X-Username` / `X-Session-Id` 供下游透传；**下游服务只信任请求头**，不自行解析 JWT
- 内部接口防护靠 `block-internal` 路由（URI 黑名单），新增内部接口必须同步加入黑名单
- **`block-internal` 的 `uri` 是精确匹配**（只有以 `*` 结尾才是前缀匹配），而它要挡的接口往往落在宽松前缀路由（`/articles/*`、`/users/*`、`/email/*`、`/ai_history/*`）覆盖范围内。新增内部接口时除了加黑名单，还要确认黑名单条目能覆盖实际访问路径，否则会出现「精确路径被挡、子路径可直达」的缺口
- 长连接用独立 upstream（读超时 3600s），与普通接口区分
- 新增对外路由：按既有分组挂 `forward-auth` 与 `limit-req`（登录类用 `*login_limit`）；public 路由必须有明确理由且不能漏挂限流
- 改配置后用 YAML 解析器校验语法，并确认目标路由的插件确实挂载
- **端口暴露（2026-09-16 按仓库实证核对）**：根 `docker-compose.yml` 里只有 gateway 映射宿主机（8080 → 容器 9080），spring / gozero / nestjs / fastapi **均不映射宿主机端口**，容器间经 `hcsy` 网络 + Nacos 直连。下游无条件信任 `X-User-Id`，所以这个收敛状态正是身份不可伪造的前提，改动编排时不要给业务服务加回 `ports`；本地开发模式（`./mix seq`）直连服务端口，没有这层保护

## 仓库工程约束

- Go / Python / TS 文件行尾统一 CRLF，仓库无 `.gitattributes`
- `gofmt -l` 会把全仓约 88 个文件报为未格式化，**全部只是行尾符差异**：判断真实格式问题必须先剥离 `\r` 再比对；禁止直接 `gofmt -w` 全量重排（会产生整文件级噪声 diff），需要重排时用「剥离 CRLF → gofmt → 还原 CRLF」
- `goctl`（`api format` / `api go`）会强制把行尾转成 LF，生成后需把 `.api`、`types.go`、`routes.go` 转回 CRLF；新建文件后同样要检查行尾
- git 用于精确核对与回退：`git status --porcelain`、`git diff --ignore-cr-at-eol`（判断是否仅行尾差异）、`git checkout -- <文件>`
- 本机环境参考：Go / gofmt 在 `C:\Program Files\Go\bin\`，goctl 在 `C:\Users\30708\go\bin\goctl.EXE`，git 在 `C:\Program Files\Git\cmd\git.EXE`（`usr\bin\` 下有 grep / tr / sed / basename 等，用完整路径调用），maven 在 `C:\apache-maven-3.9.11\bin\mvn.CMD`，javap 在 `C:\Program Files\Java\jdk-17\bin\javap.exe`
- bash 环境的 `dirname` / `head` 等不稳定（PATH 时有时无），批量格式与行尾校验优先用 Python `subprocess` 调绝对路径

- `mix` 脚本顶层子命令只有 `setup`、`swag`、`goctl-api`、`goctl-orm`、`dev`、`dist`、`docker`、`docker-services`、`loki`、`compose`、`help`；开发模式必须写全 `./mix dev multi|seq|stop`（**没有** `./mix seq` / `./mix multi` / `./mix stop`，README 历史版本里这三处写错）
- `scripts/run.sh` 的运行工具默认值：`--java-build` 默认 `maven`、`--node-runtime` 默认 `bun`、`--python-runtime` 默认 `uv`
- 两套容器编排的容器名不同：`./mix docker` 用 `mix-<service>-container`，`./mix compose` 用 `mix-<service>`（compose 的 `container_name`）
- `README.md` 同样是 CRLF，批量改文档要按「归一化 LF → 断言唯一性后替换 → 还原 CRLF」处理，不要逐处手工编辑

## 验证命令

| 服务    | 验证                                                                    |
| ------- | ----------------------------------------------------------------------- |
| spring  | `mvn compile -f spring/pom.xml`（打包 `mvn clean package -DskipTests`） |
| nestjs  | `npm run node:build`（nestjs 目录）                                     |
| fastapi | `.venv/Scripts/python.exe -c "import app"` 级导入校验 + pytest          |
| gozero  | `cd gozero/app && go build ./... && go vet ./... && go test ./... -count=1`；格式用「剥离 CRLF 后 gofmt -l」复核 |
| gateway | `apisix.yaml` 改动后校验 YAML 语法与目标路由的插件挂载（配置驱动，无业务代码） |
