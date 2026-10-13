---
title: 日志纪录器
weight: 1
---

# 日志纪录器

实现`orm.Logger`并配置`orm.DBConfig.Logger`即可打印日志。

*Example*

```go
func main() {
	// ...

	db = orm.DBConfig{
		RawDB:       rawDB,
		DBType:      orm.DBType_.MySQL,
		Logger:      MyLogger{},
		SqlLogLevel: orm.Level_.Info,
	}.Build()

	ctx := context.WithValue(context.Background(), "operator", "admin1013")

	db.Mutation(ctx).Must().BuildSql(func(b *orm.SQLBuilder) {
		b.Text("Update product SET ").Column("status").Text(" = ").Arg("draft")
		b.Text(" WHERE id = ").Arg(1)
	}).Do()
	// 打印日志:
	// level: info, operator: admin1013, msg: SQL: Update product SET `status` = ? WHERE id = ?; args: draft(string) 1(int)
}

type MyLogger struct {
}

func (l MyLogger) Debug(ctx context.Context, msg string) {
	opr, _ := ctx.Value("operator").(string)
	fmt.Printf("level: debug, operator: %s, %s\n", opr, msg)
}

func (l MyLogger) Info(ctx context.Context, msg string) {
	opr, _ := ctx.Value("operator").(string)
	fmt.Printf("level: info, operator: %s, %s\n", opr, msg)
}

func (l MyLogger) Warn(ctx context.Context, msg string) {
	opr, _ := ctx.Value("operator").(string)
	fmt.Printf("level: warn, operator: %s, %s\n", opr, msg)
}

func (l MyLogger) Error(ctx context.Context, msg string) {
	opr, _ := ctx.Value("operator").(string)
	fmt.Printf("level: error, operator: %s, %s\n", opr, msg)
}

```
