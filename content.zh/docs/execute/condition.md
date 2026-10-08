---
title: 查询条件
weight: 5
---

# 查询条件

类型`orm.Condition`提供了各种方法用于构建条件，只能用于高级执行器，通过`orm.Cond()`创建实例，支持链式调用。

| 方法                                         | 条件操作符/表达式/描述    |
|----------------------------------------------|---------------------------|
| `Eq(column string, arg any)`                 | `=`                       |
| `Ne(column string, arg any)`                 | `<>`                      |
| `Gt(column string, arg any)`                 | `>`                       |
| `Lt(column string, arg any)`                 | `<`                       |
| `Ge(column string, arg any)`                 | `>=`                      |
| `Le(column string, arg any)`                 | `<=`                      |
| `Like(column string, str string)`            | `LIKE '%<str>%'`          |
| `LikeLeft(column string, str string)`        | `LIKE '<str>%'`           |
| `LikeRight(column string, str string)`       | `LIKE '%<str>'`           |
| `LikePattern(column string, pattern string)` | `LIKE '<pattern>'`        |
| `In(column string, args []any)`              | `IN( ... )`               |
| `Between(column string, min, max any)`       | `BETWEEN <min> AND <max>` |
| `IsNull(column string)`                      | `IS NULL`                 |
| `IsNotNull(column string)`                   | `IS NOT NULL`             |
| `Raw(expr string)`                           | 原生SQL的条件表达式       |
| `Not()`                                      | 下个条件增加`NOT`修饰     |
| `Or()`                                       | 下个条件使用`OR`连接      |
| `Custom(func(*Condition))`                   | 自定义构建                |
| `Sub(*Condition)`                            | 子条件                    |

`Custom`方法用于方便在链式调用中动态构建，如下面例子：

```
```




