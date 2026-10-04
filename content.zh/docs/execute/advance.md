---
title: 高级执行器
weight: 3
---

# 高级执行器

高级执行器提供了单表CRUD功能，所有创建执行器的方法都有一个泛型，泛型必须是结构体，且不能是[[预定义实体]](../../entity/tuple)。

## Find

`Find`执行器通过方法`orm.DB.Find[E]`创建，用于查询多行记录。

| 参数                        | 描述                                                                                                                            |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `Select(...string)`         | 查询的表字段，默认为所有字段。                                                                                                  |
| `OnDemand(*orm.Demand)`     | 按需指定查询的字段，详见[[按需字段]](../on_demand_columns)。                                                                    |
| `Condition(*orm.Condition)` | 查询条件，详见[[查询条件]](../condition)。                                                                                      |
| `OrderBy(*orm.orderBy)`     | 排序，使用`orm.OrderBy()`创建并链式指定排序的字段。                                                                             |
| `Page(*orm.page)`           | 分页，使用`orm.Page(int, int)`创建，需配置[[分页模式]](../../config/db#分页模式)。                                              |
| `IncludeDeleted()`          | 如果配置了[[软删除模式]](../../config/column_policy#软删除模式)，会自动添加正常（未删除）数据的过滤条件，此方法用于跳过该处理。 |
| `LastClause(string)`        | SQL末尾的子句，例如`FOR UPDATE`。                                                                                               |
| `Do() ([]*E, error)`        | 执行并返回多个实体。                                                                                                            |

*Example*

```go
sqlDB, _ := sql.Open("mysql", "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")

db := orm.DBConfig{
	SqlDB:  sqlDB,
	DBType: orm.DBType_.MySQL, //该参数会自动配置对应的分页模式
}.Build()

users, _ := db.Find[User](nil).Select("id, name").Condition(orm.Cond().Gt("id", 10)).
	OrderBy(orm.OrderBy().Asc("id")).Page(orm.Page(0, 10)).LastClause("FOR UPDATE").Do()
// 执行SQL:
// SELECT id, name FROM user WHERE id > 10 ORDER BY id ASC LIMIT 10 FOR UPDATE
```

## FindOne

`FindOne`执行器通过方法`orm.DB.FindOne[E]`创建，用于查询单行记录。具有`Find`执行器相同参数，区别有如下方法。

| 参数               | 描述                                                                   |
|--------------------|------------------------------------------------------------------------|
| `Lenient()`        | 默认情况下查询到多行记录时会返回错误，此参数将取首个记录而不发生错误。 |
| `Do() (*E, error)` | 执行并返回单个实体。                                                   |

## Insert

`Insert`执行器通过方法`orm.DB.Insert[E]`创建，用于插入记录。还可以将生成的Key映射回实体中带`auto`[[标签]](../../entity/tag)的字段，该功能必须配置[[获取生成Key模式]](../../config/db/#获取生成key模式)，其中对于`Oracle`模式限制单次插入的实体不能1000个。

| 参数                  | 描述                                                                           |
|-----------------------|--------------------------------------------------------------------------------|
| `Entities(...*E)`     | 插入的实体，只会插入非`nil`字段，若插入多个实体，则以首个实体的非nil字段为准。 |
| `Required(...string)` | 必定会插入的字段，若值为`nil`，则插入`null`。                                  |
| `Do() (int64, error)` | 执行并返回影响行数。                                                           |

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
		DBType: orm.DBType_.Postgres, //该参数会自动配置对应的获取生成Key模式
	}.Build()

	user := &User{Name: new("Alex"), Email: new("alex@example.com")}
	db.Insert[User](nil).Entities(user).Required("status").Do()
	// 执行SQL:
	// INSERT INTO user(name, email, status) VALUES('Alex', 'alex@example.com', NULL) RETURNING id
	fmt.Println(*user.Id) // 打印生成的ID

	// 插入多个实体以首个实体的非nil字段为准
	user1 := &User{Name: new("Alex"), Email: new("alex@example.com")}
	user2 := &User{Name: new("John"), Email: new("john@example.com"), Status: new(int8(1))}

	db.Insert[User](nil).Entities(user1, user2).Do()
	// 执行SQL:
	// INSERT INTO user(name, email) VALUES ('Alex', 'alex@example.com'), ('John', 'john@example.com')

	db.Insert[User](nil).Entities(user2, user1).Do()
	// 执行SQL:
	// INSERT INTO user(name, email, status) VALUES ('John', 'john@example.com', 1), ('Alex', 'alex@example.com', NULL)
}
```

## Update

`Update`执行器通过方法`orm.DB.Update[E]`创建，用于更新记录。

| 参数                        | 描述                                                                                                                            |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `Entity(*E)`                | 更新的实体，只会更新非`nil`字段，带`pk`[[标签]](../../entity/tag)的字段例外，非`nil`时会自动作为条件。                          |
| `Required(...string)`       | 必定会更新的字段，若值为`nil`，则插入`null`。                                                                                   |
| `OnDemand(*orm.Demand)`     | 按需指定更新的字段，详见[[按需字段]](../on_demand_columns)。                                                                    |
| `Set(string, any)`          | 设置字段的更新值。                                                                                                              |
| `SetRaw(string, string)`    | 设置字段更新为指定的原生SQL表达式。                                                                                             |
| `Condition(*orm.Condition)` | 查询条件，详见[[查询条件]](../condition)。                                                                                      |
| `IncludeDeleted()`          | 如果配置了[[软删除模式]](../../config/column_policy#软删除模式)，会自动添加正常（未删除）数据的过滤条件，此方法用于跳过该处理。 |
| `SkipSafety()`              | 默认的`Update`执行器会防止全表更新，此参数可跳过该安全检查。                                                                    |
| `Do() (int64, error)`       | 执行并返回影响行数。                                                                                                            |

*Example*

```go
type User struct {
	Id       *int64 `orm:"pk"`
	Name     *string
	Email    *string
	Status   *int8
	UpdateAt *time.Time
}

func main() {
	// ...

	user := &User{Id: new(int64(1)), Name: new("Alex"), Email: new("alex@example.com")}

	affected, _ := db.Update[User](nil).Entity(user).Required("name", "email", "status").SetRaw("update_at", "now()").
		Condition(orm.Cond().Eq("status", 1)).Do()
	// 执行SQL:
	// UPDATE user SET name = 'Alex', email = 'alex@example.com', status = NULL, update_at = now() WHERE id = 1 AND status = 1
}
```

> [!TIP]
>
> 参数及[[字段策略]](../../config/column_policy#字段策略)都会影响更新的字段，详见[[更新字段优先级]](#更新字段优先级)。

## UpdateRow

`UpdateRow`执行器通过方法`orm.DB.UpdateRow[E]`创建，用于按行更新记录，可批量更新。它要求实体`E`必须有且仅有一个带`pk`[[标签]](../../entity/tag)的字段，并且传入的实体中该字段不能为`nil`。

| 参数                        | 描述                                                                                                                                           |
|-----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| `Entities(... *E)`          | 更新的实体，只会更新非`nil`字段，带`pk`[[标签]](../../entity/tag)的字段例外，会自动作为更新条件，若更新多个实体，则以首个实体的非nil字段为准。 |
| `Required(...string)`       | 必定会更新的字段，若值为`nil`，则插入`null`。                                                                                                  |
| `OnDemand(*orm.Demand)`     | 按需指定更新的字段，详见[[按需字段]](../on_demand_columns)。                                                                                   |
| `Set(string, any)`          | 设置字段的更新值。                                                                                                                             |
| `SetRaw(string, string)`    | 设置字段更新为指定的原生SQL表达式。                                                                                                            |
| `Condition(*orm.Condition)` | 查询条件，详见[[查询条件]](../condition)。                                                                                                     |
| `IncludeDeleted()`          | 如果配置了[[软删除模式]](../../config/column_policy#软删除模式)，会自动添加正常（未删除）数据的过滤条件，此方法用于跳过该处理。                |
| `Do() (int64, error)`       | 执行并返回影响行数。                                                                                                                           |

*Example*

```go
type User struct {
	Id       *int64 `orm:"pk"`
	Name     *string
	Email    *string
	Role     *string
	Status   *int8
	UpdateAt *time.Time
}

func main() {
	// ...
	
	users := []*User{
		{Id: new(int64(1)), Name: new("Alex"), Email: new("alex@example.com"), Role: new("admin")},
		{Id: new(int64(2)), Name: new("John"), Email: new("john@example.com")},
		{Id: new(int64(3)), Name: new("Charlie"), Email: new("charlie@example.com")},
	}

	affected, _ := db.UpdateRow[User](nil).Entities(users...).Required("name", "email", "role", "status").
		SetRaw("update_at", "now()").Condition(orm.Cond().Eq("status", 1)).Do()
	// 执行SQL:
	// UPDATE
	//    user SET
	//    name = CASE id
	//        WHEN 1 THEN 'Alex'
	//        WHEN 2 THEN 'John'
	//        WHEN 3 THEN 'Charlie'
	//    END,
	//    email = CASE id
	//        WHEN 1 THEN 'alex@example.com'
	//        WHEN 2 THEN 'john@example.com'
	//        WHEN 3 THEN 'charlie@example.com'
	//    END,
	//    role = CASE id
	//        WHEN 1 THEN 'admin'
	//        WHEN 2 THEN NULL
	//        WHEN 3 THEN NULL
	//    END,
	//    status = NULL,
	//    update_at = now()
	// WHERE id IN(1, 2, 3)
	//   AND status = 1
}
```

> [!TIP]
>
> 参数及[[字段策略]](../../config/column_policy#字段策略)都会影响更新的字段，详见[[更新字段优先级]](#更新字段优先级)。

## Delete

## DeletedSoftly

## 更新字段优先级
