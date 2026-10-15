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

```
package main

import (
	orm "github.com/jishaocong0910/cozy-orm"
	gen "github.com/jishaocong0910/cozy-orm-gen"
	"github.com/shopspring/decimal"
)

func main() {
	gen.Config{
		DBType:      orm.DBType_.MySQL,
		Dsn:         "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local",
		FilePath:    "repo/entity",
		CoverFile:   true,
		OmitNameTag: true,
		Tables:      []string{"tb_demo"},
		Ignores:     map[string][]string{"tb_demo": {"unused"}},
		Mappings:    gen.Mapping[decimal.Decimal]("decimal"),
	}.Gen()
}
```