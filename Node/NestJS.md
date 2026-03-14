当你为我生成代码时，请遵循以下规则：

1. Node.js 的项目要使用 eslint 来规范代码，符合当前项目的 eslint 的配置文件即可
2. Node.js 项目一般使用 TypeORM 作为 ORM 框架，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
3. TypeScript 项目尽可能使用 TypeScript 的类型，即使编译通过也尽可能加上
4. NestJS 项目使用官方默认的代码组织形式
5. NestJS 的项目，`common `一般放通用的非 `module`方法，`modules`一般放 module 的功能模块(无 controller)，接口相关的放在 `api`下
