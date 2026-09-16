---
title: 字段类型
weight: 1
---

# 字段类型

实体字段不支持嵌套结构体、非导出的字段，使用`nil`表示数据库的`null`值，必须为以下种类。

* Pointer
* Slice
* Map

## 内置类型

对于Go内置类型支持如下，包括它们的类型定义和别名。

| 种类    | 底层类型                                                                                                                                                                          |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Pointer | `int`、`int8`、`int16`、`int32`、`int64`、`uint`、`uint8`、`uint16`、`uint32`、`uint64`、`float32`、`float64`、`bool`、`string`、byte数组、`time.Time`、`uuid.UUID`（Go1.27新增） |
| Slice   | `byte`                                                                                                                                                                            |
| Map     | -                                                                                                                                                                                 |

## 自定义类型

CozyORM支持同时实现`driver.Valuer`和`sql.Scanner`的类型。因此支持一些第三方类型，例如`github.com/shopspring/decimal.Decimal`。

*Example*

```go
import (
	"github.com/shopspring/decimal"
)

type Wallet struct {
	Id      *int64
	Balance *decimal.Decimal
}
```

除了兼容Go内置的自定义类型方案，CozyORM还提供了一套更加友好的方案，通过实现`orm.Convert[V]`接口来使用。接口的泛型是一个标量类型，用于保存或映射数据库字段，并且有下列方法，标量类型有int、int8、int16、int32、int64、uint、uint8、uint16、uint32、uint64、float32、float64、bool、string。

* `orm.Convert.ToValue` 实现**字段->标量值**值的转换，作用是当字段作为SQL参数时，会被转换为标量值传入。
* `orm.Convert.ToField` 实现**标量值->字段**值的转换，作用是查询该字段时，会先映射为标量值，再通过该方法转换为字段。

