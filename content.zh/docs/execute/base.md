---
title: 基础执行器
weight: 2
---

# 基础执行器

## Query

`Query`执行器底层使用`sql.Stmt.QueryContext`方法执行，通过`orm.DB.Query[E](context.Context)`创建，其中泛型`E`必须是结构体。它通过自定义SQL进行查询，将结果映射为实体，并且兼容部分数据库获取自动生成Key的机制。

*执行器方法*

| 方法                              | 描述                                                                                                                              |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `BuildSql(func(*orm.SqlBuilder))` | 构建SQL处理函数，`*orm.SqlBuilder`变量是用于拼写自定义SQL，用法详见[[SQL构建器]](../sql_builder)                                  |
| `MapTarget(...*E)`                | 将查询结果映射到指定的实体，指定后`Do`方法将不会再返回查询结果。其设计目的，是为了兼容部分数据库，通过查询获取自动生成Key的机制。 |
| `Do([]*E, error)`                 | 执行SQL然后返回映射的实体。                                                                                                       |

*Example*

```go
users, _ := db.Query[User](nil).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("SELECT id, name, email FROM user WHERE id = $1", 1)
	}).Do()
```

### 获取自动生成Key



*PostgreSQL*

```
```

## Mutation

## SQL构建
