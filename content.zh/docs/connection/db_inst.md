---
title: DB实例
weight: 1
---

# DB实例

## 创建DB实例

DB实例即`orm.DbInst`类型的变量，用于对绑定的数据库进行操作。通过`orm.DbInstConfig`指定配置参数并调用`Build`方法创建。

Example:

```go
sqlDB, err := sql.Open("mysql", "root:12345678@tcp(127.0.0.1:3306)/test?charset=utf8mb4&parseTime=True&loc=Local")
if err != nil {
    panic(err)
}

db := orm.DbInstConfig{
    SqlDB:  sqlDB,
    DbType: orm.DbType_.MySQL,
}.Build()
```

## 配置

| 配置项              | 类型                    | 描述                                                                                                                                                                                         |
|---------------------|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SqlDB               | *sql.DB                 | 数据库连接，必填。                                                                                                                                                                           |
| Logger              | orm.Logger              | 日志记录器，用于打印SQL等日志，详见[[日志]](../logger)。                                                                                                                                     |
| SqlLogLevel         | orm.SqlLogLevel         | SQL日志级别，通过枚举`orm.Level_`选择，关于枚举见[[枚举]](#枚举)。                                                                                                                           |
| TabNameMapper       | *orm.NameMapper         | 默认的**实体名->表名**映射规则，默认为转下划线，详见[[名称映射]](#名称映射)。                                                                                                                |
| ColNameMapper       | *orm.NameMapper         | 默认的**实体字段名->表字段名**映射规则，默认为转下划线，详见[[名称映射]](#名称映射)。                                                                                                        |
| ParamPrefix         | string                  | 参数占位符，需根据数据库驱动配置，默认值为`?`，可通过指定`DbType`参数快速配置。                                                                                                              |
| GetGeneratedKeyMode | orm.GetGeneratedKeyMode | 获取生成Key模式，需根据数据库驱动配置，通过枚举`orm.GetGeneratedKeyMode_`选择，或通过指定`DbType`参数快速配置，详见[[获取生成Key]](#获取生成Key)。                                           |
| PageMode            | orm.PageMode            | 分页模式，需根据数据库驱动配置，通过枚举`orm.PageMode_`选择，或通过指定`DbType`参数快速配置，详见[[分页]](#分页)。                                                                           |
| QuotedIdentifier    | orm.QuotedIdentifier    | 引用标识符，需根据数据库驱动配置，通过枚举`orm.QuotedIdentifier_`选择，或通过指定`DbType`参数快速配置，详见[[引用标识符]](#引用标识符)。                                                     |
| DbType              | orm.DbType              | 数据库类型，指定后将不再需要配置`ParamPrefix`、`GetGeneratedKeyMode`、`PageMode`、`QuotedIdentifier`参数，通过枚举`orm.DbType_`选择，当前支持MySQL、PostgreSQL、Oracle、SQL Server、SQLite。 |
| ColumnPolicyConfigs | orm.ColumnPolicyConfigs | 字段策略配置，详见[[字段策略]](../column_policy)。                                                                                                                                           |
## 枚举

CozyORM中以下划线结尾的变量为枚举（例如：`orm.DbType_`），其字段为枚举的选项，在IDE中通过`枚举变量.`的方式即可查看所有枚举选项并选择。

> [!TIP]
> 
> 关于枚举的设计，可查看项目：[https://github.com/jishaocong0910/enum](https://github.com/jishaocong0910/enum)

## 名称映射

## 获取生成Key

## 分页

## 引用标识符


