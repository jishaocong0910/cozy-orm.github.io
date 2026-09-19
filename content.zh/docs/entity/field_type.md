---
title: 字段类型
weight: 1
---

# 字段类型

实体字段不支持嵌套结构体、非导出的字段，这些字段会被忽略。使用`nil`映射数据库的`null`值，规定字段种类必须为 **Pointer**、 **Slice**和 **Map**。

## Go内置类型

支持的Go内置类型如下，包括它们的类型定义和别名。

`*int`、`*int8`、`*int16`、`*int32`、`*int64`、`*uint`、`*uint8`、`*uint16`、`*uint32`、`*uint64`、&#8203;`*float32`、&#8203;
`*float64`、&#8203;`*bool`、&#8203;`*string`、&#8203;`[]byte`、byte数组、&#8203;`*time.Time`、&#8203;`*uuid.UUID`(Go1.27新增)

## 自定义类型

支持Go内置的`driver.Valuer`和`sql.Scanner`接口，实现这些接口的类型将被CozyORM支持，因此支持一些第三方类型，例如
`github.com/shopspring/decimal.Decimal`。

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

CozyORM还拥有独立的自定义类型机制，通过实现`orm.Convert[V]`接口来实现。

泛型`V`是一个中间类型，用于保存或映射数据库字段，类型如下，包括它们的类型定义和别名。

`int`、`int8`、`int16`、`int32`、`int64`、`uint`、&#8203;`uint8`、&#8203;`uint16`、&#8203;`uint32`、&#8203;`uint64`、&#8203;
`float32`、&#8203;`float64`、&#8203;`bool`、&#8203;`string`、&#8203;`[]byte`、byte数组、`time.Time`、`uuid.UUID`。

*需实现的方法*

| 方法                | 描述                                                                                                                                                                                 |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| orm.Convert.ToValue | 实现**自定义类型（指针）变量值->中间值->SQL参数值**的转换，若自定义类型指针变量为`nil`，则此方法不会被调用。                                                                         |
| orm.Convert.ToField | 实现**数据库值->中间值->自定义类型（指针）变量值**的转换，若数据库值为`null`则方法不会被调用。**方法接受者必须是指针**，被调用时，它指向底层类型的零值，需将中间值转化为它指向的值。 |

> [!WARNING]
>
> 如果`orm.Convert.ToField`方法的接受者底层类型是`map`，要对其设置值时，必须先用`make`初始化。

*Example*

```go
import (
  "encoding/json"
  "strings"
)

type Product struct {
    Id          *int64
    Name        *string
    Description *Description
    Tags        Tags
    SkuMap      SkuMap
}

type Description struct {
    Title  string
    Body   string
    Images []string
}

func (d Description) ToValue() string {
    b, _ := json.Marshal(d)
    return string(b)
}

func (d *Description) ToField(val string) {
    json.Unmarshal([]byte(val), d)
}

type Tags []string

func (p Tags) ToValue() string {
    return strings.Join(p, ",")
}
                                       °
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
```