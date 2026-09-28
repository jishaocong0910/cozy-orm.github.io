---
title: 基本概念
weight: 2
---

# 基本概念

## 实体

实体（Entity）是CozyORM中用于表示查询结果或持久化数据的结构体。实体的字段对应数据库中的列，使用`nil`映射数据库的`null`值。

*Example*

```go
type User struct {
    Id    *int64
    Name  *string
    Email *string
}
```

## 执行器

CozyORM通过类型`orm.DB`提供的方法创建执行器执行SQL，执行器分为**基础执行器**和**高级执行器**。基础执行器通过自定义SQL执行，具有更高的通用性。高级执行器是对常用功能的封装，自动生成SQL执行。

*基础执行器*

* `Query` 用于执行查询语句，返回查询结果。
* `Mutation` 用于执行更新、插入、删除语句，返回受影响的行数。

*高级执行器*

* `Find` 查询多行记录。
* `FindOne` 查询单行记录。
* `Insert` 插入记录。
* `Update` 更新记录。
* `UpdateRow` 按行更新记录。
* `Delete` 删除记录。
* `DeleteSoftly` 软删除记录。

