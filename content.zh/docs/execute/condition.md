---
title: 查询条件
weight: 5
---

# 查询条件

类型`orm.Cond`提供了各种方法用于拼接条件，支持链式调用。

| 方法                                         | 条件操作符/表达式/描述                                      |
|----------------------------------------------|-------------------------------------------------------------|
| `Eq(column string, arg any)`                 | `=`                                                         |
| `Ne(column string, arg any)`                 | `<>`                                                        |
| `Gt(column string, arg any)`                 | `>`                                                         |
| `Lt(column string, arg any)`                 | `<`                                                         |
| `Ge(column string, arg any)`                 | `>=`                                                        |
| `Le(column string, arg any)`                 | `<=`                                                        |
| `Like(column string, str string)`            | `LIKE '%<str>%'`                                            |
| `LikeLeft(column string, str string)`        | `LIKE '<str>%'`                                             |
| `LikeRight(column string, str string)`       | `LIKE '%<str>'`                                             |
| `LikePattern(column string, pattern string)` | `LIKE '<pattern>'`                                          |
| `In(column string, args []any)`              | `IN( ... )`                                                 |
| `Between(column string, min, max any)`       | `BETWEEN <min> AND <max>`                                   |
| `IsNull(column string)`                      | `IS NULL`                                                   |
| `IsNotNull(column string)`                   | `IS NOT NULL`                                               |
| `Not()`                                      | 下个条件增加`NOT`修饰                                       |
| `Or()`                                       | 下个条件使用`OR`连接                                        |
| `Sub(func(*orm.Cond))`                       | 子条件处理函数，使用`*orm.Cond`拼接子条件                   |
| `Expr(func(*orm.CondExpr))`                  | 条件表达式处理函数，使用`*orm.CondExpr`条件表达式和设置参数 |

`Raw`方法用于添加原生SQL的条件表达式，如下面例子：

```
```

`Determine`方法用于在链式调用中动态拼接条件，如下面例子：

```go
// 查找用户的订单
func findOrders(ctx context.Context, userId int64, merchantId int64, orderNo *string, status *int8,
	startAt *time.Time, endAt *time.Time, page int, pageSize int) ([]*Order, error) {
	/*
	c := orm.Cond().
		Eq("user_id", userId).
		Eq("merchant_id", merchantId)

	if orderNo != nil {
		c.Eq("order_no", orderNo)
	}

	if status != nil {
		c.Eq("status", status)
	}

	if startAt != nil {
		if endAt != nil {
			c.Between("create_at", startAt, endAt)
		} else {
			c.Ge("create_at", startAt)
		}
	} else if endAt != nil {
		c.Le("create_at", endAt)
	}

	return db.Find[Order](ctx).Cond(c).OrderBy(orm.OrderBy().Asc("id")).
		Page(orm.Page((page-1)*pageSize, pageSize)).Do()
	*/

	/* 使用Determine方法优化上面的代码，使链式调用更连贯 */

	return db.Find[Order](ctx).Cond(orm.Cond().
		Eq("user_id", userId).
		Eq("merchant_id", merchantId).
		Determine(func(c *orm.Cond) {
			if orderNo != nil {
				c.Eq("order_no", orderNo)
			}
		}).
		Determine(func(c *orm.Cond) {
			if status != nil {
				c.Eq("status", status)
			}
		}).
		Determine(func(c *orm.Cond) {
			if startAt != nil {
				if endAt != nil {
					c.Between("create_at", startAt, endAt)
				} else {
					c.Ge("create_at", startAt)
				}
			} else if endAt != nil {
				c.Le("create_at", endAt)
			}
		})).
		OrderBy(orm.OrderBy().Asc("id")).Page(orm.Page((page-1)*pageSize, pageSize)).Do()
}
```

`Sub`方法会自动为子条件添加括号对，如下面例子：

```go
// 获取用户单个订单
func getOrder(ctx context.Context, userId int64, merchantId int64, orderNo *string) ([]*Order, error) {
	return db.Find[Order](ctx).Cond(orm.Cond().
		Eq("user_id", userId).
		Eq("merchant_id", merchantId).
		Sub(orm.Cond().
			Eq("order_no", orderNo).
			Or().
			Eq("trade_no", orderNo))).
		Do()
	// 执行SQL:
	// SELECT ... FROM order WHERE user_id = ? AND merchant_id = ? AND (order_no = ? OR trade_no = ?) AND status = ?
}
```




