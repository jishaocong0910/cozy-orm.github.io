---
title: 核心概念
weight: 2
---

# 核心概念

## 实体

实体（Entity）是CozyORM中用于表示查询结果或持久化数据的结构体。实体的字段对应数据库中的列，使用`nil`映射数据库的`null`值，只允许字段为指针、切片或map，详见[[字段类型]](../../entity/field_type)。

*Example*

```go
type User struct {
    Id    *int64
    Name  *string
    Email *string
}
```

## 执行方法

CozyORM通过类型`orm.DB`提供的方法执行SQL操作数据库，这些方法称为**执行方法**，并且分为**基础执行方法**和**高级执行方法**。**基础执行方法**通过自定义 SQL 语句执行数据库操作，具有更高的通用性。**高级执行方法**是对常用功能的封装，自动生成 SQL 并执行。

*基础执行方法*

* `orm.DB.Query` 用于执行查询语句，返回查询结果。
* `orm.DB.Mutation` 用于执行更新、插入、删除语句，返回受影响的行数。

*高级执行方法*

* `orm.DB.Find` 查询多行记录。
* `orm.DB.FindOne` 查询单行记录。
* `orm.DB.Insert` 插入多行记录。
* `orm.DB.Update` 更新记录。
* `orm.DB.UpdateRow` 按行更新记录。
* `orm.DB.Delete` 删除记录。
* `orm.DB.DeleteSoftly` 软删除记录。

