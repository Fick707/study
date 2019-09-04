# PostGres QL 命令备忘

## 登录退出

* 切换到postgres用户
```
su postgres
```

* 进入命令行工具，默认用户名和库都是postgres
```
psql
```

* 或者指定登录用户和登录的数据库
```
psql -U user -d dbname
```

* 退出
```
\q
```

* 修改某用户密码
```
\password userName
```

## 数据库信息查询

* 列举数据库
```
\l
```

* 切换数据库
```
\c dbname
```

* 列举表
```
\dt
```

* 查看表结构
```
\d tableName
```

* 查看索引
```
\di
```

## 服务信息命令

* 查看字符编码
```
\encoding
```

* 显示服务发行条款
```
\copyright
```

* 帮助
```
\h
```

## 常用查询命令

* 查看执行计划(可以看到执行时间)
```
explain sql语句
```

* 执行脚本（登录之后在命令行执行）
```
\i /pathA/xxx.sql 
```

* 未登录状态下执行
```
psql -d db1 -U userA -f /pathA/xxx.sql
```

## 高阶命令

* 导出指定表中的数据,成insert语句
* ```注``` pg_dump 工具在postgres安装目录下的bin目录下
```
pg_dump -h hostname -p port -U userName -t table1 -t table2 --inserts -f outputfilename.sql dbname
```

```注:```

* 表名，字段名大小写敏感；
* sql法大小写不敏感；