---
title: 预定义实体
weight: 3
---

# 预定义实体

预定义实体是CozyORM提供的一些类型，可直接当作实体，用于方便查询少量字段，或非表字段（例如聚合函数）等业务，有如下类型。预定义实体只能用于基础执行方法[[Query]](../../execute/base/#query)

* `orm.Tuple[T]`
* `orm.Tuple2[T1, T2]`
* `orm.Tuple3[T1, T2, T3]`
* `orm.Tuple4[T1, T2, T3, T4]`
* `orm.Tuple5[T1, T2, T3, T4, T5]`
* `orm.Tuple6[T1, T2, T3, T4, T5, T6]`

泛型的个数代表可查询的字段个数，且须满足[[字段类型]](../field_type)要求，为了方便，对于指针，允许直接写底层类型。

```go
emails, _ := db.Query[orm.Tuple2[int64, *string]](nil).BuildSql(func(b *orm.SqlBuilder) {
	b.Write("SELECT id, email FROM user WHERE id IN(1, 2, 3)")
}).Do()

for _, name := range emails {
	fmt.Println(name.Field1, name.Field2)
}

orders, _ := db.Query[orm.Tuple3[int64, int64, int64]](nil).BuildSql(func(b *orm.SqlBuilder) {
	b.Write("SELECT user_id, COUNT(*), SUM(order_amount) ")
	b.Write("FROM orders ")
	b.Write("WHERE status = 'paid' ")
	b.Write("  AND user_id IN(1, 2, 3) ")
	b.Write("GROUP BY user_id")
}).Do()

for _, order := range orders {
	fmt.Println(order.Field1, order.Field2, order.Field3)
}
```