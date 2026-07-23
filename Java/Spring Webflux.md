1. 当你为我生成代码时，请遵循以下规则：

   1. Java 项目创建新的 Spring 项目时，入库文件名统一使用 `Main.java`
   2. Java 项目不同业务场景在 `AsyncConfig` 中定义独立 `ThreadPoolTaskExecutor` Bean；`CompletableFuture.runAsync + ConcurrentHashMap` 收集并行结果，失败降级为串行兜底
   3. Spring 项目生成代码时尽可能参考当前项目其他的注解使用和相关实现方式
   4. Spring WebFlux 项目 Controller 返回 `Mono<Result<T>>` / `Flux<Result<T>>`，Service 返回 `Mono<T>` / `Flux<T>`，不得返回裸类型或调用 `.block()`
   5. Spring WebFlux 项目数据库使用 Spring Data R2DBC，Repository 继承 `ReactiveCrudRepository<T, ID>`，自定义 SQL 用 `@Query` + `@Param`，增删改用 `@Modifying`；事务用 `TransactionalOperator.transactional()` 包裹，不得使用 `@Transactional`；不再使用 MyBatisPlus
   6. Spring WebFlux 项目请求上下文通过 Reactor Context 传递，用 `Mono.deferContextual(ctx -> {...})` 获取用户信息；过滤器用 `WebFilter` + `.contextWrite()` 写入 Context；不得使用 `ThreadLocal`、`RequestContextHolder`、`OncePerRequestFilter`、`HandlerInterceptor`
   7. Spring WebFlux 项目并行独立 IO 用 `Mono.zip()` / `Mono.when()`，不得使用 `CompletableFuture.allOf()`
   8. Spring WebFlux 项目全局异常处理保留 `@RestControllerAdvice`，`@ExceptionHandler` 返回 `Mono<ResponseEntity<Result<?>>>`；`ResponseBodyAdvice` 不可用，通过 `ResponseEntity` 设置 HTTP 状态码
   9. Spring WebFlux 项目阻塞 CPU 操作（如加密）用 `Mono.fromCallable(() -> ...).subscribeOn(Schedulers.boundedElastic())` 隔离
   10. Spring WebFlux 项目微服务间调用用 `WebClient` + LoadBalancer，封装为 Client 类，通过 `Mono.deferContextual()` 注入 `X-User-Id` 等请求头；不得使用 OpenFeign
   11. Spring WebFlux 项目 Redis 用 `ReactiveStringRedisTemplate`；缓存模式用 `redisUtil.get(key).switchIfEmpty(Mono.defer(() -> loadFromDB().flatMap(data -> writeCache(key, data).thenReturn(data))))`，不得使用 `@Cacheable`
   12. Spring WebFlux 项目 AOP 切面中 `pjp.proceed()` 返回 `Mono<?>` 时必须在 Mono 链内操作（`.flatMap()` / `.doOnSuccess()` / `.doOnError()`），用 `.deferContextual()` 获取上下文，不得使用 `RequestContextHolder`