---
title: SQL构建器
weight: 4
---

# SQL构建器

SQL构建器（orm.SQLBuilder）用于在[[基础执行器]](../base)中构建SQL语句和设置参数，提供了以下方法，支持链式调用。

## Write

拼接SQL语句和设置参数。

*Example*

```go
type User struct {
	Id     *int64
	Name   *string
	Email  *string
	Status *int8
}

func param(i *int) string {
	*i++
	return "$" + strconv.Itoa(*i)
}

func main() {
	sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

	db := orm.DBConfig{
		SqlDB:       sqlDB,
		DBType:      orm.DBType_.Postgres,
	}.Build()

	user := &User{
		Id:    new(int64(1)),
		Name:  new("Alex"),
		Email: new("alex@example.com"),
	}

	db.Mutation(nil).BuildSql(func(b *orm.SQLBuilder) {
		var i int
		b.Write("Update user SET")
		if user.Name != nil {
			b.Write(" name = "+param(&i), user.Name)
		}
		if user.Email != nil {
			if i > 0 {
				b.Write(",")
			}
			b.Write(" email = "+param(&i), user.Email)
		}
		if user.Status != nil {
			if i > 0 {
				b.Write(",")
			}
			b.Write(" status = "+param(&i), user.Status)
		}
		b.Write(" WHERE id = "+param(&i), user.Id)
	}).Do()
	// 执行SQL：
	// Update user SET name = $1, email = $2 WHERE id = $3
	//
	// $1: Alex
	// $2: alex@example.com
	// $3: 1
}
```
## WritePh

拼接参数占位符，受[[DB配置]](../../config/db)的配置项`ParamPrefix`影响。

*Example*

```go
type User struct {
	Id     *int64
	Name   *string
	Email  *string
	Status *int8
}

func main() {
	sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

	db := orm.DBConfig{
		SqlDB:  sqlDB,
		DBType: orm.DBType_.Postgres, // 该参数会自动配置对应的参数占位符前缀
	}.Build()

	user := &User{
		Id:    new(int64(1)),
		Name:  new("Alex"),
		Email: new("alex@example.com"),
	}

	db.Mutation(nil).BuildSql(func(b *orm.SQLBuilder) {
		sep := ""
		b.Write("Update user SET")
		if user.Name != nil {
			b.Write(" name = ").WritePh().AddArgs(user.Name)
			sep = ","
		}
		if user.Email != nil {
			b.Write(sep).Write(" email = ").WritePh().AddArgs(user.Email)
			sep = ","
		}
		if user.Status != nil {
			b.Write(sep).Write(" status = ").WritePh().AddArgs(user.Status)
			sep = ","
		}
		b.Write(" WHERE id = ").WritePh().AddArgs(user.Id)
	}).Do()
	// 执行SQL：
	// Update user SET name = $1, email = $2 WHERE id = $3
	//
	// $1: Alex
	// $2: alex@example.com
	// $3: 1
}
```

## WriteColumn

拼接带引用标识符的字段名，受[[DB配置]](../../config/db)的配置项`QuotedIdentifier`影响。

*Example*

```go
type User struct {
	Id     *int64
	Name   *string
	Email  *string
	Status *int8
}

func main() {
	sqlDB, _ := sql.Open("postgres", "postgres://postgres:12345678@localhost:5432/postgres?sslmode=disable")

	db := orm.DBConfig{
		SqlDB:       sqlDB,
		DBType:      orm.DBType_.Postgres, // 该参数会自动配置对应的引用标识符
	}.Build()

	user := &User{
		Id:    new(int64(1)),
		Name:  new("Alex"),
		Email: new("alex@example.com"),
	}

	db.Mutation(nil).BuildSql(func(b *orm.SQLBuilder) {
		sep := ""
		b.Write("Update user SET")
		if user.Name != nil {
			b.Write(" ").WriteColumn("name").Write(" = ").WritePh().AddArgs(user.Name)
			sep = ","
		}
		if user.Email != nil {
			b.Write(sep).Write(" ").WriteColumn("email").Write(" = ").WritePh().AddArgs(user.Email)
			sep = ","
		}
		if user.Status != nil {
			b.Write(sep).Write(" ").WriteColumn("status").Write(" = ").WritePh().AddArgs(user.Status)
            sep = ","
		}
		b.Write(" WHERE ").WriteColumn("id").Write(" = ").WritePh().AddArgs(user.Id)
	}).Do()
	// 执行SQL：
	// Update user SET "name" = $1, "email" = $2 WHERE "id" = $3
	//
	// $1: Alex
	// $2: alex@example.com
	// $3: 1
}
```

## AddArgs

添加参数。

*Example*

```go
db.Query[User](nil).BuildSql(func(b *orm.SQLBuilder) {
	b.Write("SELECT * FROM user WHERE id = $1 AND status = $2")
	b.Args(1, 2)
}).Do()
```

## ForEach

遍历切片拼接SQL语句，同时可拼接开始、分隔和结束符号。

方法声明：`ForEach[T any](sep separate, items []T, handler func(i int, item T)) *orm.SQLBuilder`。其中`sep`通过以下函数指定，`handler`的参数`i`为当前元素的索引，`t`为当前元素。

| 函数                                           | 描述                                                      |
|------------------------------------------------|-----------------------------------------------------------|
| `orm.Sep(separator string)`                    | 拼接分隔符。                                              |
| `orm.SepWrap(open, separator close string)`    | 拼接开始、分隔和结束符。                                  |
| `orm.SepWrapOpt(open, separator close string)` | 拼接开始、分隔和结束符，切片为空时不拼接`open`和`close`。 |

*Example*

```go
columns := []string{"id", "name", "email"}
ids := []int64{1, 2, 3}
var status []int8

db.Query[User](nil).BuildSql(func(b *orm.SQLBuilder) {
	b.Write("SELECT ").ForEach(b.Sep(", "), columns, func(_ int, item string) {
		b.Write(item)
	})
	b.Write(" FROM user WHERE id IN").ForEach(b.SepWrap("(", ", ", ")"), ids, func(i int, item int64) {
		b.Write("$"+strconv.Itoa(i+1), item)
	})
	b.ForEach(b.SepWrapOpt("AND status IN(", ", ", " )"), status, func(i int, item int8) {
		b.Write("$"+strconv.Itoa(i+1), item)
	})
}).Do()
// 执行SQL：
// SELECT id, name, email FROM user WHERE id IN($1, $2, $3)
//
// $1: 1
// $2: 2
// $3: 3
```

## Accept

接受自定义的SQL拼接。

方法声明：`Accept(SQLWriter) *orm.SQLBuilder`。其中`SQLWriter`是一个接口，要求实现`WriteSQL(*SQLBuilder)`用于拼接SQL语句。

*Example*

```go
type ColumnWriter struct {
	Columns []string
}

func (c ColumnWriter) WriteSQL(b *orm.SQLBuilder) {
	b.ForEach(b.Sep(", "), c.Columns, func(_ int, item string) {
		b.Write(item)
	})
}

func main() {
	// ...

	columns := ColumnWriter{Columns: []string{"id", "name", "email"}}

	db.Query[User](nil).BuildSql(func(b *orm.SQLBuilder) {
		b.Write("SELECT ").Accept(columns).Write(" FROM user WHERE id = 1")
	}).Do()
	// 执行SQL：
	// SELECT id, name, email FROM user WHERE id = 1
}
```

## Cancel

取消SQL的执行，不会返回错误。调用该方法后应立即`return`。

*Example*

```go
var ids []int64

users, err := db.Query[User](nil).BuildSql(func(b *orm.SQLBuilder) {
	if len(ids) == 0 {
		b.Cancel()
		return
	}
	b.Write("SELECT id, name, email FROM user WHERE id IN")
	b.ForEach(b.SepWrap("(", ",", ")"), ids, func(_ int, item int64) {
		b.WritePh().AddArgs(item)
	})
}).Do()
// users=[]*User{}，err=nil
```

## Error

取消SQL的执行，并返回错误。调用该方法后应立即`return`。

*Example*

```go
var ids []int64

users, err := db.Query[User](nil).BuildSql(func(b *orm.SQLBuilder) {
	if len(ids) == 0 {
		b.Error(errors.New("ids is empty"))
		return
	}
	b.Write("SELECT id, name, email FROM user WHERE id IN")
	b.ForEach(b.SepWrap("(", ",", ")"), ids, func(_ int, item int64) {
		b.WritePh().AddArgs(item)
	})
}).Do()
// users=[]*User{}，err="ids is empty"
```