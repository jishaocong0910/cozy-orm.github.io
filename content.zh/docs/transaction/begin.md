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
		/* 必须将处理函数的ctx参数传递给执行器，才能使其事务中进行 */
		_, err := db.Insert[ProductOrder](ctx).Entities(productOrder).Do()
		if err != nil {
			return err
		}
		_, err = db.Insert[TradeOrder](ctx).Entities(tradeOrder).Do()
		if err != nil {
			return err
		}

		/* 以下代码为错误示范 */

		// 执行器没有传入ctx参数
		//
		// _, err := db.Insert[ProductOrder](nil).Entities(productOrder).Do()
		// if err != nil {
		//     return err
		// }

		// 执行器传错ctx参数
		//
		// _, err = db.Insert[TradeOrder](normalCtx).Entities(tradeOrder).Do()
		// if err != nil {
		//     return err
		// }
		return nil
	})
}
```