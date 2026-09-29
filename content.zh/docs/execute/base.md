---
title: 基础执行器
weight: 2
---

# 基础执行器

基础执行器是CozyORM的核心功能，通过自定义SQL执行，并且兼容部分数据库获取自动生成Key的机制。

## Query

`Query`执行器用于执行SELECT语句，底层使用方法`sql.Stmt.QueryContext`执行，使用方法`orm.DB.Query[E]`创建，其中泛型`E`必须是结构体。

*执行器方法*

| 方法                              | 描述                                                                                                                        |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `BuildSql(func(*orm.SqlBuilder))` | 构建SQL处理函数，使用`*orm.SqlBuilder`变量拼接SQL，用法详见[[SQL构建器]](../sql_builder)                                    |
| `MapTargets(...*E)`               | 将查询结果映射到指定的实体，指定后`Do`方法将不会再返回查询结果。其设计目的是：兼容部分数据库通过查询结果获取生成Key的机制。 |
| `Do() ([]*E, error)`              | 执行SQL并返回映射的实体。                                                                                                   |

*Example*

```go
users, _ := db.Query[User](nil).BuildSql(func(b *orm.SqlBuilder) {
		b.Write("SELECT id, name, email FROM user WHERE id = $1", 1)
	}).Do()
```

### 获取生成Key

部分数据库获取生成Key是通过结果集返回，而不是`sql.Result.LastInsertId`方法，例如PostgreSQL、SQL Server。

*Go原生方式示例*

```go
users := []*User{
	{Name: new("Alex"), Email: new("alex@example.com")},
	{Name: new("John"), Email: new("john@example.com")},
}

sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

rows, err := sqlDB.Query("INSERT INTO user(name, email) VALUES($1, $2), ($3, $4) RETURNING id",
	users[0].Name, users[0].Email, users[1].Name, users[1].Email)
if err != nil {
	panic(err)
}

var ids []int64
for i := 0; rows.Next(); i++ {
	var id int64
	rows.Scan(&id)
	ids = append(ids, id)
}
fmt.Println(ids)
```

`Query`执行器兼容这种机制，通过`MapTargets`方法将生成的Key映射到实体中。

*Query执行器示例*

```go
users := []*User{
	{Name: new("Alex"), Email: new("alex@example.com")},
	{Name: new("John"), Email: new("john@example.com")},
}

sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

db := orm.DBConfig{
	SqlDB:  sqlDB,
	DBType: orm.DBType_.Postgres,
}.Build()

db.Query[User](nil).MapTargets(users...).BuildSql(func(b *orm.SqlBuilder) {
	b.Write("INSERT INTO user(name, email) VALUES($1, $2), ($3, $4) RETURNING id",
		users[0].Name, users[0].Email, users[1].Name, users[1].Email)
}).Do()

fmt.Println(users[0].Id, users[1].Id)
```

## Mutation

`Mutation`执行器用于执行INSERT、UPDATE和DELETE语句，底层使用方法`sql.Stmt.ExecContext`执行。

*执行器方法*

| 方法                              | 描述                                                                                     |
|-----------------------------------|------------------------------------------------------------------------------------------|
| `BuildSql(func(*orm.SqlBuilder))` | 构建SQL处理函数，使用`*orm.SqlBuilder`变量拼接SQL，用法详见[[SQL构建器]](../sql_builder) |
| `MapTargets(...*E)`               | 将`sql.Result.LastInsertId`映射到指定实体。                                              |
| `Do() (int64, error)`             | 执行SQL并返回映射的实体。                                                                |

*Example*

```
```