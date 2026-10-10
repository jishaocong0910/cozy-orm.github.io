---
title: 开始事务
weight: 1
---

# 开始事务

`orm.Tx`是一个事务实例，用于管理事务的生命周期。通过`orm.DB.Tx(ctx context.Context)`创建，`ctx`参数为nil时将使用`context.Background()`作为默认值。`orm.Tx`具有以下方法并支持链式调用。

| 方法                                  | 描述                                             |
|---------------------------------------|--------------------------------------------------|
| `Must()`                              | 默认会将错误返回，此参数会直接panic。            |
| `TxOptions(*sql.TxOptions)`           | 设置事务选项。                                   |
| `Do(func(ctx context.Context) error)` | 执行业务处理函数，它会自动开启、提交和回滚事务。 |

`orm.Tx`在开启事务时，会向`ctx`添加一个`*sql.Tx`值，这就是`Do`方法中业务处理函数的`ctx`参数。每个执行器都具有一个`ctx`参数，并优先使用其中`*sql.Tx`执行SQL。将业务处理函数的`ctx`参数将传递给执行器，即可使执行器在事务中执行。

*Example*

```go
// 创建商品和交易订单
func createOrder(normalCtx context.Context, productOrder *ProductOrder, tradeOrder *TradeOrder) error {
	return db.Tx(normalCtx).Do(func(ctx context.Context) error {
		// 处理函数的ctx参数必须传递给执行器，才能使其在事务中执行。
		_, err := db.Insert[ProductOrder](ctx).Entities(productOrder).Do()
		if err != nil {
			return err
		}

		// 这里执行器设置了Must参数，错误会被panic，orm.Tx实例会recover并返回error
		db.Insert[TradeOrder](ctx).Must().Entities(tradeOrder).Do()

		/* 以下代码是导致执行器不在事务中执行的错误示范 */

		// 没有传入ctx参数
		//
		// _, err := db.Insert[ProductOrder](nil).Entities(productOrder).Do()
		// if err != nil {
		//     return err
		// }

		// 传错ctx参数
		//
		// _, err = db.Insert[TradeOrder](normalCtx).Entities(tradeOrder).Do()
		// if err != nil {
		//     return err
		// }
		return nil
	})
}
```