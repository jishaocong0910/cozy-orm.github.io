---
title: 事务钩子
weight: 3
---

# 事务钩子

`orm.TxHook`是一个事务钩子管理器，用于指定在事务实例生命周期的特定节点，调用自定义函数。通过`orm.DB.TxHook(ctx context.Context)`创建，只有`ctx`存在事务才会创建成功，否则返回`nil`。`orm.TxHook`提供了以下钩子方法，支持链式调用。

## BeforeCommitSync

在事务提交前同步执行，遵循事务传播机制。

方法参数：`(handler func(ctx context.Context) error)`。`handler`函数的返回值`error`会被事务实例返回。

*Example*

```
```

## AfterCompleteAsync

在事务完成后异步执行。

方法参数：`(handler func(ctx context.Context, commit bool), inheritCtxKeys ...any)`。`handler`函数若发生`panic`，会被恢复并打印错误日志。`ctx`值为`context.Background()`，并且会继承事务实例的`ctx`参数中，key为`inheritCtxKeys`参数的值。若事务提交`commit`参数为true，回滚则为false。

```go
func updateUser(normalCtx context.Context, u *User, at *AvatarShrinkTask) error {
	db.Tx(normalCtx).Do(func(ctx context.Context) error {
		db.Update[User](ctx).Must().Entity(u).Do()
		db.Insert[AvatarShrinkTask](ctx).Must().Entities(at).Do()

		db.TxHook(ctx).AfterCompleteAsync(func(hookCtx context.Context, commit bool) {
			// 已提交，则处理任务
			if commit {
				// 查询任务信息
				task, _ := db.FindOne[AvatarShrinkTask](hookCtx).Must().Cond(func(c *orm.Cond) {
					c.Eq("id", at.Id)
				}).Do()

				// ...

				// 压缩用户头像 ...

				// ...
			}

			// 删除任务
			db.Delete[AvatarShrinkTask](hookCtx).Must().Cond(func(c *orm.Cond) {
				c.Eq("id", at.Id)
			}).Do()
		}, "trace_id") // hookCtx将继承ctx中key为“trace_id”的值
		return nil
	})
	return nil
}
```