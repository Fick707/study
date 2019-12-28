# seaweedfs 学习

## 背景

facebook的论文以学习理解NoFS（Net Object File System）

## 核心概念
在逻辑上Seaweedfs的几个概念：

* Node 系统抽象的节点，抽象为DataCenter、Rack、DataNode
* DataCenter 数据中心，对应现实中的不同机房
* Rack 机架，对应现实中的机柜
* Datanode 存储节点，用于管理、存储逻辑卷
* Volume 逻辑卷，存储的逻辑结构，逻辑卷下存储Needle
* Needle 逻辑卷中的Object，对应存储的文件
* Collection 文件集，可以分布在多个逻辑卷上
