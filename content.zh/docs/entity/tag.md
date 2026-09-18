---
title: 标签
weight: 2
---

# 标签

标签格式：`orm:"<tag>=<value>"`，多个标签时用`;`分隔。

*标签*

| 标签   | 描述                                                                                                                                                |
|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| table  | 指定映射表名，只能用于第一个字段，且为匿名的空结构体。                                                                                              | 
| column | 指定映射数据库字段名。                                                                                                                              |
| pk     | 标识为主键。                                                                                                                                        |
| auto   | 标识为自动生成字段。标签值为自动递增步长，默认为1，仅对[[获取生成Key模式]](../../config/db/#获取生成key模式)为`FirstInsertId`和`LastInsertId`有效。 |
| ignore | 标识忽略该字段。                                                                                                                                    |

*Example*

```go
type User struct {
	_        struct{}   `orm:"table=user"`
	Id       *int64     `orm:"column=id;pk;auto"`
	Name     *int64     `orm:"column=name"`
	CreateAt *time.Time `orm:"column=create_at"`
	Online   *bool      `orm:"ignore"`
}
```
