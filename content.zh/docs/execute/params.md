---
title: 执行参数
weight: 1
---

# 执行参数

执行方法（基础/高级执行方法）除了`ctx`是方法本身的参数，其他参数都是通过链式调用方式设置。

*Example*
```go
db.Query[User](ctx).Describe("query users")
```

公共参数是所有执行方法都具有的参数。

*公共参数/方法*

| 参数/方法   | 描述 |
|-------------|------|
| ctx         |      |
| Must        |      |
| Describe    |      |
| SqlLogLevel |      |