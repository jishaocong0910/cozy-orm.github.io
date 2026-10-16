---
title: 代码生成器
weight: 5
---

# 代码生成器

CozyORM提供了实体代码生成器。

## 安装

```shell
go get github.com/jishaocong0910/cozy-orm-gen
```

## 快速开始

```go
package main

import (
	"database/sql"

	orm "github.com/jishaocong0910/cozy-orm"
	gen "github.com/jishaocong0910/cozy-orm-gen"
	"github.com/shopspring/decimal"
)

func main() {
	db, err := sql.Open("mysql", "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")
	if err != nil {
		panic(err)
	}
	
	gen.Config{
		DB:           db,
		DBType:       orm.DBType_.MySQL,
		FilePath:     "repo/entity",
		Tables:       []string{"user", "product", "order"},
		TypeMappings: gen.Mapping[decimal.Decimal]("decimal"),
	}.Gen()
}

```

## 配置项

`gen.Config`字段为生成器的配置项。

| 配置项           | 类型                  | 描述                                                                                                                     |
|------------------|-----------------------|--------------------------------------------------------------------------------------------------------------------------|
| DB               | `*sql.DB`             | 数据库连接池，必填。                                                                                                     |
| DBType           | `orm.DBType`          | 数据库类型，通过[[枚举]](../../config/db#枚举)`orm.DBType_`选择，当前支持MySQL、PostgreSQL、Oracle、SQL Server、SQLite。 |
| FilePath         | `string`              | 输出文件路径，它是`os.Getwd()`的相对路径，为空时使用默认值`entity`。                                                     |
| CoverFile        | `bool`                | 是否覆盖已有文件                                                                                                         |
| OmitNameTag      | `bool`                | 不添加`table`和`column`标签。                                                                                            |
| Tables           | `[]string`            | 需要生成的表。                                                                                                           |
| IgnoreColumns    | `map[string][]string` | 指定忽略的表字段，这些字段不会出现在实体中，key为表名，value为字段名。                                                   |
| EntityNamePrefix | `string`              | 添加实体名称前缀。                                                                                                       |
| EntityNameSubfix | `string`              | 添加实体名称后缀。                                                                                                       |
| TypeMappings     | `gen.TypeMappings`    | 自定义类型映射规则，见[[类型映射]](#类型映射)                                                                            |

## 类型映射

默认情况下，生成的实体字段是Go内置类型。在Go内置类型不满足业务的场景下，可以在生成实体后手动修改实体字段的类型，也可以配置生成器的`TypeMappings`配置项，将指定的数据库类型映射到自定义或第三方类型。

通过`gen.Mapping[T](columnType string)`创建，可链式调用指定多个映射规则，按照顺序匹配。泛型`T`为实体字段类型，必须满足[[字段类型]](../field_type)要求，为了方便，允许指针写成值类型。`columnType`参数为数据库字段类型，格式为`<data_type>[(<parameter_list>)]`，参数列表分隔符为`,`或空格，通配符`#`表示单个任意值，例子：`int`、`decimal(#, 2)`、`VARCHAR2(100 CHAR)`、`VARCHAR2(100 BYTE)`。