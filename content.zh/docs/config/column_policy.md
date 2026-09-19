---
title: 字段策略
weight: 3
---

# 字段策略

字段策略作用于**高级执行方法**，用于配置字段的在方法中的统一处理方式。通过
{{< html >}}<code>orm.NewColumnPolicyConfig(column&nbsp;string, tables&nbsp;...string)</code>{{< /html >}}}创建配置，其中`tables`参数指定作用的表范围，若不指定则为所有表的默认配置，表字段名称与
`column`相同会自动使用。每个表的字段只会使用一个字段策略，匹配多个策略时，指定了`tables`参数的优先级更高，若都指定了`tables`
参数则取最后配置的策略。

字段通过链式调用方式，先调用`OnInsert()`、`OnUpdate()`、`OnDeleteSoftly()`指定事件，再调用事件提供的策略方法配置处理方式。

*事件策略方法*

{{< html >}}
<table>
    <thead>
        <tr>
            <th>事件</th><th>策略方法</th><th>描述</th>
        </tr>
    </thead>
    <tbody> 
        <tr>
            <td rowspan="3">OnInsert/OnUpdate</td><td><code>Value(force&nbsp;bool, batchReuse&nbsp;bool, value&nbsp;func()&nbsp;any)</code></td><td>在插入/更新记录时生成字段值。<code>force</code>指定是否强制覆盖生成的值，若为<code>false</code>则仅在插入/更新的值为<code>null</code>时使用。<code>batchReuse</code>指定是否在批量插入/更新（高级执行方法<code>Insert</code>/<code>UpdateRow</code>）时只生成一次并复用，若为<code>false</code>则插入/更新的每行都生成一次。<code>value</code>为字段值的生成函数。</td>
        </tr>
        <tr>
            <td><code>RawSql(force&nbsp;bool, batchReuse&nbsp;bool, rawSql&nbsp;func()&nbsp;string)</code></td><td>与<code>Value</code>方法的区别是，生成字段值为<code>rawSql</code>函数返回的原生SQL表达式。</td>
        </tr>
        <tr>
            <td><code>Never()</code></td><td>不赋值。将忽略插入/更新记录时对字段的赋值。</td>
        </tr>
        <tr>
            <td rowspan="2">OnDeleteSoftly</td><td><code>AssignedPkMode[T](normalValue&nbsp;T)</code></td><td>使用<strong>AssignedPkMode</strong>模式软删除，见<a href="#%E8%BD%AF%E5%88%A0%E9%99%A4">[软删除]</a>。</td>
        </tr>
        <tr>
            <td><code>AssignedNullMode[T](normalValue&nbsp;T)</code></td><td>使用<strong>AssignedNullMode</strong>模式软删除，见<a href="#%E8%BD%AF%E5%88%A0%E9%99%A4">[软删除]</a>。</td>
        </tr>
    </tbody>
</table>
{{< /html >}}

*Example*

```go
db := orm.DBConfig{
		SqlDB:  sqlDB,
		DBType: orm.DBType_.MySQL,
		ColumnPolicyConfigs: orm.ColumnPolicyConfigs{
			// 指定所有表的id字段，使用自定义的ID生成器创建，并且禁止更新。
			orm.NewColumnPolicyConfig("id").
				OnInsert().Value(true, false, func(ctx context.Context) any {
				return myIdGenerator.Int64()
			}).OnUpdate().Never(),
			
			// 指定user表的name字段在插入记录时，若值为null则随机生成用户名。
			orm.NewColumnPolicyConfig("name", "user").
				OnInsert().Value(false, false, func(ctx context.Context) any {
				return fmt.Sprintf("user_%09d", rand.Intn(1000000000))
			}),
			
			// 指定user表的region字段从Context中获取。
			orm.NewColumnPolicyConfig("region", "user").
				OnInsert().Value(true, true, func(ctx context.Context) any {
				if region, ok := ctx.Value("region").(string); ok {
					return region
				}
				return nil
			}),
		},
	}.Build()
```

## 软删除

## 内置策略

| 方法          | 描述 |
|---------------|------|
| UseCreateTime | 描述 |
| UseUpdateTime | 描述 |
| UseRowVersion | 描述 |