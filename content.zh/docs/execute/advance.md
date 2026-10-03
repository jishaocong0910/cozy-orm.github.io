---
title: 高级执行器
weight: 3
---

# 高级执行器

高级执行器提供了单表CRUD功能，所有创建执行器的方法都有一个泛型，泛型必须是结构体，且不能是[[预定义实体]](../../entity/tuple)。

## Find

`Find`执行器通过方法`orm.DB.Find[E]`创建，用于查询多行记录。

*参数/方法*

| 方法                        | 描述                                                                                                                                                                     |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Select(...string)`         | 查询的表字段，默认为所有字段。                                                                                                                                           |
| `OnDemand(*orm.OnDemand)`   | 按需指定查询的字段，详见[[按需字段]](../on_demand_fields)。                                                                                                              |
| `Condition(*orm.Condition)` | 查询条件，详见[[查询条件]](../condition)。                                                                                                                               |
| `OrderBy(*orm.orderBy)`     | 排序，使用`orm.OrderBy()`创建并链式指定排序的字段。                                                                                                                      |
| `Page(*orm.page)`           | 分页，使用`orm.Page(int, int)`创建，需配置[[分页模式]](../../config/db#%E5%88%86%E9%A1%B5%E6%A8%A1%E5%BC%8F)。                                                           |
| `IncludeDeleted()`          | 如果配置了[[软删除模式]](../../config/column_policy#%E8%BD%AF%E5%88%A0%E9%99%A4%E6%A8%A1%E5%BC%8F)，则默认会查询正常（未删除）的数据。此方法用于忽略对已删除数据的过滤。 |
| `LastClause(string)`        | SQL末尾的子句，例如`FOR UPDATE`。                                                                                                                                        |
| `Do() ([]*E, error)`        | 执行并返回多个实体。                                                                                                                                                     |

*Example*

```go
users, _ := db.Find[User](nil).Select("id, name").Condition(orm.Cond().Gt("id", 10)).
	OrderBy(orm.OrderBy().Asc("id")).Page(orm.Page(0, 10)).LastStr("FOR UPDATE").Do()
// 执行SQL:
// SELECT id, name FROM user WHERE id > 10 ORDER BY id ASC LIMIT 10 FOR UPDATE
```

## FindOne

`FindOne`执行器通过方法`orm.DB.FindOne[E]`创建，用于查询单行记录。具有`Find`执行器相同参数，区别有如下方法。

*参数/方法*

| 方法               | 描述                                                             |
|--------------------|------------------------------------------------------------------|
| `Compatible()`     | 默认在查询到多行记录时返回错误，此参数将取首个记录而不发生错误。 |
| `Do() (*E, error)` | 执行并返回单个实体。                                             |

## Insert

`Insert`执行器通过方法`orm.DB.Insert[E]`创建，用于插入记录。还可以将生成的Key映射回实体中带有`auto`[[标签]](../../entity/tag)的字段，该功能必须配置[[获取生成Key模式]](../../config/db/#%E8%8E%B7%E5%8F%96%E7%94%9F%E6%88%90key%E6%A8%A1%E5%BC%8F)。

*参数/方法*

| 方法                        | 描述                                                    |
|-----------------------------|---------------------------------------------------------|
| `Entities(...*E)`           | 插入的实体。只会插入非`nil`字段。                       |
| `Required(...string)`       | 必定会插入的字段，若值为`nil`，则插入`null`。           |
| `Do() (int64, error)`       | 执行并返回影响行数。                                    |

*Example*

```go
type User struct {
	Id     *int64 `orm:"auto"`
	Name   *string
	Email  *string
	Status *int8
}

func main() {
	sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

	db := orm.DBConfig{
		SqlDB:  sqlDB,
		DBType: orm.DBType_.Postgres, // 该参数会自动配置对应的获取生成Key模式
	}.Build()

	user := &User{Name: new("Alex"), Email: new("alex@example.com")}
	db.Insert[User](nil).Entities(user).Required("status").Do()
	// 执行SQL:
	// INSERT INTO user(name, email, status) VALUES('Alex', 'alex@example.com', NULL) RETURNING id
	fmt.Println(*user.Id) // 打印生成的ID
}
```

若插入多个实体，则非`nil`字段以首个为准。

*Example*

```go
user1 := &User{Name: new("Alex"), Email: new("alex@example.com")}
user2 := &User{Name: new("John"), Email: new("john@example.com"), Status: new(int8(1))}

db.Insert[User](nil).Entities(user1, user2).Do()
// 执行SQL:
// INSERT INTO user(name, email) VALUES ('Alex', 'alex@example.com'), ('John', 'john@example.com')

db.Insert[User](nil).Entities(user2, user1).Do()
// 执行SQL:
// INSERT INTO user(name, email, status) VALUES ('John', 'john@example.com', 1), ('Alex', 'alex@example.com', NULL)
```

## Update



## UpdateRow

## Delete

## DeletedSoftly
