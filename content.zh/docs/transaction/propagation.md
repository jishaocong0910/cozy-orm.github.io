---
title: 事务传播
weight: 2
---

# 事务传播

事务传播机制是执行器的内部逻辑：检查`ctx`参数中的值，若存在与自己相同DB实例的事务则加入，否则开启新事务。

以下示例中，调用`createOrder`时，两个函数会在同一个事务中执行。

```go
// 创建商品和交易订单
func createOrder(ctx context.Context, productOrder *ProductOrder, tradeOrder *TradeOrder) error {
	return db.Tx(ctx).Do(func(ctx context.Context) error {
		_, err := db.Insert[ProductOrder](ctx).Must().Entities(productOrder).Do()
		if err != nil {
			return err
		}

		_, err = db.Insert[TradeOrder](ctx).Must().Entities(tradeOrder).Do()
		if err != nil {
			return err
		}

		err = deductInventory(ctx, productOrder)
		if err != nil {
			return err
		}
		return nil
	})
}

// 扣减库存
func deductInventory(ctx context.Context, productOrder *ProductOrder) error {
	return db.Tx(ctx).Do(func(ctx context.Context) error {
		// 查询库存
		inventory, err := db.FindOne[Inventory](ctx).Cond(func(c *orm.Cond) {
			c.Eq("sku_id", productOrder.SkuId)
		}).LastClause("FOR UPDATE").Do()
		if err != nil {
			return err
		}

		// 确认库存足够
		remainingStock := *inventory.Stock - *productOrder.Quantity
		if remainingStock < 0 {
			return errors.New("not enough stock")
		}

		// 更新库存
		db.Update[Inventory](ctx).Must().Set("stock", remainingStock).Cond(func(c *orm.Cond) {
			c.Eq("id", inventory.Id)
		}).Do()

		// 创建库存日志
		db.Insert[InventoryLog](ctx).Must().Entities(&InventoryLog{
			InventoryId:    inventory.Id,
			ProductOrderId: productOrder.Id,
			Stock:          productOrder.Quantity,
			StockBefore:    inventory.Stock,
			StockAfter:     new(remainingStock),
			Operation:      new("deduct"),
		}).Do()
		return nil
	})
}
```