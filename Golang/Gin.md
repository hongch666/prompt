当你为我生成代码时，请遵循以下规则：

1. Go 的项目需要检查导包是否存在，以及检查是否存在没有使用的变量或者函数，必要时使用 go 的 build 工具打包检验，打包后的 exe 文件名和入口函数文件名一样
2. Go 的项目任何可能阻塞或耗时的函数都应该接收context参数,第一个参数通常是context,嵌套调用继续传同一个context,context的起点一般是请求的handler
3. Gin 的项目一般使用 Gorm 作为 ORM 框架进行数据库操作，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
4. Gin 项目使用依赖注入的模式，采用的是手动依赖注入（Manual DI）模式，通过结构体分组来管理依赖
5. Gin 等项目采用三层架构，`controller`写接口函数，`service`实现逻辑，`mapper`实现数据库 ORM 操作，`entity`或者 `model`额外实现
