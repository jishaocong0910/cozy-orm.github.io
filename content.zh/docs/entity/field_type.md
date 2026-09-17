---
title: 字段类型
weight: 1
---

# 字段类型

实体字段不支持嵌套结构体、非导出的字段，使用`nil`映射数据库的`null`值，规定字段种类必须为**Pointer**、**Slice**和**Map**。

## Go内置类型

Go内置类型支持如下，包括它们的类型定义和别名。

| 种类    | 类型                                                                                                                                                                                                                        |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Pointer | `int`、`int8`、`int16`、`int32`、`int64`、`uint`、`uint8`、`uint16`、`uint32`、`uint64`、&#8203;`float32`、&#8203;`float64`、&#8203;`bool`、&#8203;`string`、byte数组、&#8203;`time.Time`、&#8203;`uuid.UUID`（Go1.27新增） |
| Slice   | `byte`                                                                                                                                                                                                                      |
| Map     | -                                                                                                                                                                                                                           |

## 自定义类型

CozyORM支持同时实现Go内置的`driver.Valuer`和`sql.Scanner`接口的类型，例如第三方类型`github.com/shopspring/decimal.Decimal`。

*Example*

```go
import (
	"github.com/shopspring/decimal"
)

type Wallet struct {
	Id      *int64
	UserId  *int64
	Balance *decimal.Decimal
}
```

CozyORM拥有独立的自定义类型机制，通过实现`orm.Convert[V]`接口来实现。

泛型`V`是一个标量类型，用于保存或映射数据库字段，标量类型有`int`、`int8`、`int16`、`int32`、`int64`、`uint`、&#8203;`uint8`、&#8203;`uint16`、&#8203;`uint32`、&#8203;`uint64`、&#8203;`float32`、&#8203;`float64`、&#8203;`bool`、&#8203;`string`。

*需实现的方法*

* `orm.Convert.ToValue` 实现**字段->标量值**值的转换，作用是当自定义类型的变量作为SQL参数时转换为标量值，若变量为`nil`则此方法不会被调用。
* `orm.Convert.ToField` 实现**标量值->字段**值的转换，作用是查询的字段映射为自定义类型时，会先映射为标量值，再通过此方法转换为自定义类型。**此方法的接受者必须是指针**，被调用时接受者是一个指向底层类型零值的指针，需将标量值转化为到指针指向的值。

> [!WARNING]
>
> 如果`orm.Convert.ToField`方法的接受者底层类型是`map`，对其设置值是必须先通过`make`对其初始化。

*Example*

```go
import (
	"encoding/json"
	"strings"
)

type Product struct {
	Id     *int64
	Name   *string
	Tags   Tags
	SkuMap SkuMap
}

type Tags []string

func (p Tags) ToValue() string {
	return strings.Join(p, ",")
}

func (p *Tags) ToField(val string) {
	*p = strings.Split(val, ",")
}

type SkuMap map[string]string

func (s *SkuMap) ToValue() string {
	b, _ := json.Marshal(s)
	return string(b)
}

func (s *SkuMap) ToField(val string) {
	json.Unmarshal([]byte(val), s)
}

type Description struct{
	
}
```