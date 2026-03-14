当你为我生成代码时，请遵循以下规则：

1. Node.js 的项目要使用 eslint 来规范代码，符合当前项目的 eslint 的配置文件即可
2. Node.js 项目一般使用 TypeORM 作为 ORM 框架，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
3. Express 项目采用三层架构，`controller`写接口函数，`service`实现逻辑，`mapper`实现数据库 ORM 操作，`entity`或者 `model`额外实现
4. Express 项目一般使用 JS 而不是 TS
