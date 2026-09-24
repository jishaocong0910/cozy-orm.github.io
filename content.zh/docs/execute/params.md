---
title: 参数
weight: 1
---

# 参数

执行方法除了`ctx`是本身的参数，其他参数都是通过链式调用方式设置，它们都具有的以下参数。

| 参数                     | 描述                                                                                                                                                                                                                                      |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ctx context.Context`    | `ctx`会透传到[[日志器]](../../log/logger)、[[字段策略]](../../config/column_policy)，用于[[事务传播]](../../transaction/propagation)。若为`nil`会使用`context.Background()`创建默认值。它是非常重要的参数，因此设计为执行方法本身的参数。 |
| `Must()`                 | 默认的，执行方法会将错误返回，此参数会直接panic。                                                                                                                                                                                         |
| `Description(string)`    | 用于描述SQL业务，打印在SQL日志里。                                                                                                                                                                                                        |
| `SqlLogLevel(orm.Level)` | 指定SQL日志的级别，通过[[枚举]](../../config/db#%E6%9E%9A%E4%B8%BE)`orm.Level_`选择，优先级高于[[DB配置]](../../config/db)。指定为`orm.Level_.UNDEFINED`可以关闭SQL日志。                                                                 |

*Example*
```go
db.FindOne[User](ctx).Description("describe the SQL").Condition(orm.Cond().Eq("id", 1)).Do()
```