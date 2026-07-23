当你为我生成代码时，请遵循以下规则：

1. Node.js 的项目要使用 eslint 来规范代码，符合当前项目的 eslint 的配置文件即可
2. Node.js 项目一般使用 TypeORM 作为 ORM 框架，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
3. Node.js 项目独立异步调用用 `Promise.all` 并行化，条件不满足时用 `Promise.resolve(空值)` 占位保证数组一致性
4. TypeScript 项目尽可能使用 TypeScript 的类型，即使编译通过也尽可能加上
5. NestJS 项目使用官方默认的代码组织形式
6. NestJS 项目按以下分层组织代码：`config/` 放 YAML 配置和`index.ts`加载器；`common/` 放全局共享的常量、异常、工具类（非 module）；`framework/` 放全局横切关注点（decorator、guard、interceptor、filter、middleware等）；`module/common/` 放公共服务模块；`module/system/` 放系统业务模块
