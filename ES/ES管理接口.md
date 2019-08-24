# ES管理接口

## 查询

|功能|url|备注|
|---|---|---|
|服务状态查询|/_cat/health?v|获取ES服务的概览信息|
|服务节点列表|/_cat/nodes?v|获取节点概览信息|
|索引列表|/_cat/indices?v|获取所有索引概览信息|
|指定索引状态查询|/indexname?pretty=true|查询指定索引的状态|
|指定索引指定映射的映射|/indexname/_mapping/typename|查询指定类型的数据结构(映射)|