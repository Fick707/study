# PostGres QL 命令备忘

* 切换到postgres用户
> su postgres

* 进入命令行工具，默认用户名和库都是postgres
> psql
* 或者指定登录用户和登录的数据库
> psql -U user -d dbname

* 列举数据库
> \l

* 切换数据库
> \c dbname

* 列举表
> \dt

* 查看表结构
> \d tableName

* 查看索引
> \di

```注:```

* 表名，字段名大小写敏感；
* sql法大小写不敏感；