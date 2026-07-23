当你为我生成代码时，请遵循以下规则：

1. 使用中文进行回复，除非明确要求使用其他语言
2. 代码中的注释和字符串信息要使用中文
3. 在项目里面不要生成任何文档，除非明确说明需要生成文档
4. 生成注释和相关的文章说明不要有任何的 emoji 表情和标志
5. 代码中如果需要使用SQL,尽可能使用`?`占位语法, 而不是字符串拼接
6. 生成代码时参考项目的代码组织方式和文件的命名方式
7. 生成代码时参考当前项目或者服务的日志记录方式
8. 如果项目有常量类，字符串尽可能抽取到常量类调用，如果代码的语言有支持字符串模板，字符串模板就不用抽离，且抽离的字符串主要是消息类的
9. 如果当前项目有使用 Swagger，按照当前项目的接口的 Swagger 实现方式进行对应实现
10. 写业务代码时，相互独立的 IO 调用（RPC/HTTP/DB/Redis/ES 等）必须并行化，不得串行等待
11. 当前项目所有服务使用默认的项目结构
13. 如果项目中已有成熟的代码组织形式和代码风格，遵循已有的风格和组织方式
14. Python 的项目生成代码时，如果要使用某个代码中的函数/类/变量，如果该代码/功能所在的文件夹有 `__init__.py`文件导出，在导入使用的时候不要使用文件的位置，而是文件夹的位置，因为已经被导出，除非不存在 `__init__.py`文件或者明确要求
15. Python 项目尽可能使用类型标注进行生成代码
16. Python 项目生成代码时，不要生成过多没有使用的 `import`，生成完之后检查一下
17. Python 项目生成代码时，所有的 `import`统一在代码的最前面导入，不要在逻辑中进行导入
18. Python 项目独立异步 IO 用 `asyncio.gather` 并行执行，循环内并行将每次迭代封装为内部 async 函数后 gather
19. FastAPI 的项目使用 FastAPI 的 `Depend`依赖注入实现，并且服务要通过类来组织功能方法，包括对应获取类实例的函数(用于依赖注入)
20. FastAPI 项目的接口需要使用 `async`异步接口，其他功能函数也尽可能使用 `asyncio`库的功能和异步函数，并且使用 ` await`调用，如果有同步流程使用 `run_in_threadpool`结合 `await`调用
21. FastAPI 项目使用 SQLAlchemy 作为 ORM 框架进行数据库操作，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
22. Go 的项目需要检查导包是否存在，以及检查是否存在没有使用的变量或者函数，必要时使用 go 的 build 工具打包检验，打包后的 exe 文件名和入口函数文件名一样
23. Go 的项目任何可能阻塞或耗时的函数都应该接收context参数,第一个参数通常是context,嵌套调用继续传同一个context,context的起点一般是请求的handler
26. GoZero 的项目要求 logic 中不要有 SQL 代码,相关 SQL 的功能在 `model`封装暴露方法使用
27. GoZero 项目使用日志记录的工具为对应的 logx,logx 没有 `Warn`和 `Warnf`函数,一般使用 `Error`和 `Errorf`工具
28. GoZero 项目记录日志时,尽可能使用带上下文的Logger实例来记录,如果没有才使用logx的全局日志方法
29. GoZero项目在并行业务上使用 `mr.Finish` 并行执行独立任务，每个任务为 `func() error`，结果写入闭包局部变量，内部吞错返回 nil
30. TypeScript 项目尽可能使用 TypeScript 的类型，即使编译通过也尽可能加上
31. Node.js 的项目要使用 eslint 来规范代码，符合当前项目的 eslint 的配置文件即可
32. Node.js 项目使用 TypeORM 作为 ORM 框架
33. Node.js 项目独立异步调用用 `Promise.all` 并行化，条件不满足时用 `Promise.resolve(空值)` 占位保证数组一致性
34. NestJS 项目按以下分层组织代码：`config/` 放 YAML 配置和`index.ts`加载器；`common/` 放全局共享的常量、异常、工具类（非 module）；`framework/` 放全局横切关注点（decorator、guard、interceptor、filter、middleware等）；`module/common/` 放公共服务模块；`module/system/` 放系统业务模块
35. Java 项目创建新的 Spring 项目时，入库文件名统一使用 `Main.java`
36. Java 项目不同业务场景在 `AsyncConfig` 中定义独立 `ThreadPoolTaskExecutor` Bean；`CompletableFuture.runAsync + ConcurrentHashMap` 收集并行结果，失败降级为串行兜底
37. Spring 项目生成代码时尽可能参考当前项目其他的注解使用和相关实现方式
40. Spring WebFlux 项目 Controller 返回 `Mono<Result<T>>` / `Flux<Result<T>>`，Service 返回 `Mono<T>` / `Flux<T>`，不得返回裸类型或调用 `.block()`
41. Spring WebFlux 项目数据库使用 Spring Data R2DBC，Repository 继承 `ReactiveCrudRepository<T, ID>`，自定义 SQL 用 `@Query` + `@Param`，增删改用 `@Modifying`；事务用 `TransactionalOperator.transactional()` 包裹，不得使用 `@Transactional`；不再使用 MyBatisPlus
42. Spring WebFlux 项目请求上下文通过 Reactor Context 传递，用 `Mono.deferContextual(ctx -> {...})` 获取用户信息；过滤器用 `WebFilter` + `.contextWrite()` 写入 Context；不得使用 `ThreadLocal`、`RequestContextHolder`、`OncePerRequestFilter`、`HandlerInterceptor`
43. Spring WebFlux 项目并行独立 IO 用 `Mono.zip()` / `Mono.when()`，不得使用 `CompletableFuture.allOf()`
44. Spring WebFlux 项目全局异常处理保留 `@RestControllerAdvice`，`@ExceptionHandler` 返回 `Mono<ResponseEntity<Result<?>>>`；`ResponseBodyAdvice` 不可用，通过 `ResponseEntity` 设置 HTTP 状态码
45. Spring WebFlux 项目阻塞 CPU 操作（如加密）用 `Mono.fromCallable(() -> ...).subscribeOn(Schedulers.boundedElastic())` 隔离
46. Spring WebFlux 项目微服务间调用用 `WebClient` + LoadBalancer，封装为 Client 类，通过 `Mono.deferContextual()` 注入 `X-User-Id` 等请求头；不得使用 OpenFeign
47. Spring WebFlux 项目 Redis 用 `ReactiveStringRedisTemplate`；缓存模式用 `redisUtil.get(key).switchIfEmpty(Mono.defer(() -> loadFromDB().flatMap(data -> writeCache(key, data).thenReturn(data))))`，不得使用 `@Cacheable`
48. Spring WebFlux 项目 AOP 切面中 `pjp.proceed()` 返回 `Mono<?>` 时必须在 Mono 链内操作（`.flatMap()` / `.doOnSuccess()` / `.doOnError()`），用 `.deferContextual()` 获取上下文，不得使用 `RequestContextHolder`