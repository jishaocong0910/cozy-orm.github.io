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

```go
// 保存操作日志
func SaveOperationLog(ctx context.Context, log *OperationLog) error {
	// 若ctx存在事务，则在事务提交之前插入操作日志，hookCtx等于ctx。
	hook := db.TxHook(ctx).BeforeCommitSync(func(hookCtx context.Context) error {
		_, err := db.Insert[OperationLog](hookCtx).Must().Entities(log).Do()
		return err
	})

	// 若ctx不存在事务，则直接保存操作日志。
	if hook == nil {
		_, err := db.Insert[OperationLog](ctx).Entities(log).Do()
		if err != nil {
			return err
		}
	}
	return nil
}
```

## AfterCompleteAsync

在事务完成后异步执行。

方法参数：`(handler func(ctx context.Context, commit bool), copyCtxKeys ...any)`。`handler`函数的`ctx`参数以`context.Background()`为基础创建，`commit`参数在事务提交成功为true，回滚时为false。若`handler`发生`panic`，会被恢复并打印错误日志。`copyCtxKeys`参数用于指定需要从事务钩子管理器的`ctx`复制到`handler`的`ctx`的值对应的key。

```go
// 压缩用户头像
func ShrinkAvatar(ctx context.Context, userId *int64) error {
	// 若ctx存在事务，则在事务提交成功后，异步对用户头像进行压缩。
	hook := db.TxHook(ctx).AfterCompleteAsync(func(hookCtx context.Context, commit bool) {
		if commit {
			ShrinkAvatarHandler(ctx, userId)
		}
	}, "trace_id") // 复制ctx中key为“trace_id”的值到hookCtx。

	// 若ctx不存在事务，则直接压缩用户头像。
	if hook == nil {
		go ShrinkAvatarHandler(ctx, userId)
	}
	return nil
}
```