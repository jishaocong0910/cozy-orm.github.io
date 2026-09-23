---
title: 字段类型
weight: 1
---

# 字段类型

实体字段不支持嵌套结构体、非导出的字段，这些字段会被忽略。使用`nil`映射数据库的`null`值，规定字段种类必须为 **Pointer**、 **Slice**和 **Map**。

## Go内置类型

支持以下Go内置类型，包括它们的类型定义和别名。

`*int`、`*int8`、`*int16`、`*int32`、`*int64`、`*uint`、`*uint8`、`*uint16`、`*uint32`、`*uint64`、`*float32`、&#8203;
`*float64`、&#8203;`*bool`、&#8203;`*string`、&#8203;`*[N]byte`、`*time.Time`、`*uuid.UUID`(Go1.27新增)、`[]byte`

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

CozyORM拥有独立的自定义类型机制，通过实现`orm.Convert[V]`接口来实现。

泛型`V`是一个中间类型，用于保存或映射数据库字段，类型必须是[[Go内置类型]](#go内置类型)，并且为了方便，对于指针种类，允许直接写底层类型。

*`orm.Convert[V]`的方法*

| 方法                                                               | 描述                                                                                                                                                                               |
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| {{< html >}}<code>orm.Convert.ToValue()&nbsp;V</code>{{< /html >}} | 实现**自定义类型值->中间类型值->SQL参数值**的转换，若自定义类型值为`nil`，则此方法不会被调用。                                                                                     |
| `orm.Convert.ToField(V)`                                           | 实现**数据库值->中间类型值->自定义类型值**的转换，若数据库值为`null`则方法不会被调用。**该方法接受者必须是指针**，被调用时，它指向底层类型的零值，需将中间类型值转化为它指向的值。 |

> [!WARNING]
>
> 如果`orm.Convert.ToField(V)`方法的接受者底层类型是`map`，由于它是零值，不能使用`map[key]=value`设置值，必须先用`make`初始化。

*Example*

```go
import (
  "encoding/json"
  "strings"
)

// 实体
type Product struct {
	Id          *int64
	Name        *string
	Description *Description
	Tags        Tags
	SkuMap      SkuMap
}

// 自定义类型
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

// 自定义类型
type Tags []string

func (p Tags) ToValue() string {
	return strings.Join(p, ",")
}

func (p *Tags) ToField(val string) {
	*p = strings.Split(val, ",")
}

// 自定义类型
type SkuMap map[string]string

func (s *SkuMap) ToValue() *string {
	if b, err := json.Marshal(s); err == nil {
		return new(string(b))
	}
	return nil
}

func (s *SkuMap) ToField(value *string) {
	// 数据库值不为null时该方法才会被调用，因此value不会等于nil
	json.Unmarshal([]byte(*value), s)
}
```