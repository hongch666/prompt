当你为我生成代码时，请遵循以下规则：

1. Go 的项目需要检查导包是否存在，以及检查是否存在没有使用的变量或者函数，必要时使用 go 的 build 工具打包检验，打包后的 exe 文件名和入口函数文件名一样
2. Go 的项目任何可能阻塞或耗时的函数都应该接收context参数,第一个参数通常是context,嵌套调用继续传同一个context,context的起点一般是请求的handler
3. GoZero 项目使用官方默认的代码组织形式
4. GoZero 的项目要求 logic 中不要有 SQL 代码,相关 SQL 的功能在 `model`封装暴露方法使用
5. GoZero 项目使用日志记录的工具为对应的 logx,logx 没有 `Warn`和 `Warnf`函数,一般使用 `Error`和 `Errorf`工具
6. GoZero 项目记录日志时,尽可能使用带上下文的Logger实例来记录,如果没有才使用logx的全局日志方法
