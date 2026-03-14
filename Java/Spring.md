当你为我生成代码时，请遵循以下规则：

1. Spring项目采用三层架构，`controller`写接口函数，`service`实现逻辑，`mapper`实现数据库 ORM 操作，`entity`或者 `model`额外实现
2. Java 项目创建新的 Spring 项目时，入库文件名统一使用 `Main.java`
3. Spring 项目生成代码时尽可能参考当前项目其他的注解使用和相关实现方式
4. Spring 项目一般使用 MyBatisPlus 作为 ORM 框架进行数据库操作，如果当前项目有 ORM 框架遵循当前项目的 ORM 框架
5. 使用 MyBatisPlus 如果遇到复杂 SQL 需求，在 `mapper`接口定义对应的函数，不使用注解，而是使用 SQL XML 实现对应的 SQL 功能
