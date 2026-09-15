---
title: DB配置
weight: 1
---

# DB配置

## 配置

`orm.DBConfig`的字段为DB实例的配置项。

| 配置项              | 类型                    | 描述                                                                                                                                                                                         |
|---------------------|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SqlDB               | *sql.DB                 | 数据库连接，必填。                                                                                                                                                                           |
| Logger              | orm.Logger              | 日志记录器，用于打印SQL等日志，详见[[日志]](../logger)。                                                                                                                                     |
| SqlLogLevel         | orm.SqlLogLevel         | SQL日志级别，通过枚举`orm.Level_`选择，关于枚举见[[枚举]](#枚举)。                                                                                                                           |
| TabNameMapper       | *orm.NameMapper         | 默认的**实体名->表名**映射规则，默认为转下划线，详见[[名称映射]](#名称映射)。                                                                                                                |
| ColNameMapper       | *orm.NameMapper         | 默认的**实体字段名->表字段名**映射规则，默认为转下划线，详见[[名称映射]](#名称映射)。                                                                                                        |
| ParamPrefix         | string                  | 参数占位符，需根据数据库驱动配置，默认值为`?`，可通过指定`DbType`参数快速配置。                                                                                                              |
| GetGeneratedKeyMode | orm.GetGeneratedKeyMode | 获取生成Key模式，需根据数据库驱动配置，通过枚举`orm.GetGeneratedKeyMode_`选择，或通过指定`DbType`参数快速配置，详见[[获取生成Key模式]](#获取生成Key模式)。                                   |
| PageMode            | orm.PageMode            | 分页模式，需根据数据库驱动配置，通过枚举`orm.PageMode_`选择，或通过指定`DbType`参数快速配置，详见[[分页模式]](#分页模式)。                                                                   |
| QuotedIdentifier    | orm.QuotedIdentifier    | 引用标识符，需根据数据库驱动配置，通过枚举`orm.QuotedIdentifier_`选择，或通过指定`DbType`参数快速配置，详见[[引用标识符]](#引用标识符)。                                                     |
| DbType              | orm.DbType              | 数据库类型，指定后将不再需要配置`ParamPrefix`、`GetGeneratedKeyMode`、`PageMode`、`QuotedIdentifier`参数，通过枚举`orm.DbType_`选择，当前支持MySQL、PostgreSQL、Oracle、SQL Server、SQLite。 |
| ColumnPolicyConfigs | orm.ColumnPolicyConfigs | 字段策略配置，详见[[字段策略]](../column_policy)。                                                                                                                                           |
## 枚举

CozyORM中以下划线结尾的变量为枚举（例如：`orm.DbType_`），其字段为枚举的选项，在IDE中通过`枚举变量.`的方式即可查看所有枚举选项并选择。

> [!TIP]
> 关于枚举的设计，可查看项目：[https://github.com/jishaocong0910/enum](https://github.com/jishaocong0910/enum)

## 名称映射

名称映射器通过`orm.NewNameMapper()`创建，然后链式调用其方法指定映射规则，可指定多个规则，将按调用顺序处理。

*方法/规则*
  
* `LowerCamelCase`     转小驼峰   
* `LowerSnakeCase`     转小写下划线 
* `LowerFirstLiteral`  首字母小写  
* `UpperCamelCase`     转大驼峰   
* `UpperSnakeCase`     转大写下划线 
* `UpperFirstLiteral`  首字母大写
* `AddPrefix`          添加前缀
* `AddSuffix`          添加后缀
* `SubPrefix`          删除前缀
* `SubSuffix`          删除后缀

*Example*

```go
// ...

type UserInfo struct {
	Id       *int64
	NickName *string
}

func main() {
	// ...

	db := orm.DBConfig{
		SqlDB:         sqlDB,
		DbType:        orm.DbType_.MySQL,
		TabNameMapper: orm.NewNameMapper().LowerCamelCase().AddPrefix("tb_"),
	}.Build()ui

	db.FindOne[UserInfo](nil).Must().Condition(orm.Cond().Eq("id", 1)).Do()
	// 执行SQL:
	// SELECT id, nick_name FROM tb_user_info WHERE id = ?
}
```

## 获取生成Key模式

Go中不同数据库获取生成Key没有标准，原生的`sql.Result.LastInsertId()`不能支持所有数据库，即使支持，在语义上也有所不同。`orm.GetGeneratedKeyMode_`总结了各种获取生成Key的方式，创建`orm.DB`时根据数据库特性选择。有以下选项，其中基础执行方法`orm.DB.Mutation`只支持`FirstInsertId`和`LastInsertId`，高级执行方法`orm.DB.Insert`支持所有，详见[[Mutation]]()[[Insert]]()。

* `FirstInsertId` 将`sql.Result.LastInsertId()`的返回值作为第一个插入记录的ID，适配数据库例子：MySQL、MariaDB。
* `LastInsertId` 将`sql.Result.LastInsertId()`的返回值作为最后一个插入记录的ID，适配数据库例子：SQLite。
* `Returning` 通过`INSERT ... RETURNING ...`返回语法返回自动生成Key，适配数据库例子：PostgreSQL、SQLite。
* `SQLServer` 通过SQL Server方言`INSERT ... OUTPUT INSERTED.<column> ...`返回自动生成Key。
* `Oracle` 通过Oracle的方言`INSERT ... RETURNING ... INTO...`返回自动生成Key。

## 分页模式

## 引用标识符


