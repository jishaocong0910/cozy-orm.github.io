---
title: 字段策略
weight: 3
---

# 字段策略

字段策略作用于**高级执行方法**，用于配置字段的在方法中的统一处理方式。通过
`orm.NewColumnPolicyConfig(column string, tables ...string)`创建配置，其中`tables`参数指定作用的表范围，若不指定则为所有表的默认配置，表字段名称与
`column`相同会自动使用。每个表的字段只会使用一个字段策略，匹配多个策略时，指定了`tables`参数的优先级更高，若都指定了`tables`
参数则取最后配置的策略。

字段通过链式调用方式，先调用`OnInsert()`、`OnUpdate()`、`OnDeleteSoftly()`指定事件，再调用事件提供的策略方法配置处理方式。

*事件策略方法*

{{< raw-html >}}
<table>
    <thead>
        <tr>
            <th>事件</th><th>策略方法</th><th>描述</th>
        </tr>
    </thead>
    <tbody> 
        <tr>
            <td rowspan="2">OnInsert</td><td><code>Value(force bool, batchReuse bool, value func() any)</code></td><td>在插入记录时生成字段值。<code>force</code>指定是否强制覆盖生成的值，若为<code>false</code>则仅在插入值为<code>nil</code>时使用。<code>batchReuse</code>指定是否在批量插入时只生成一次并复用，若为<code>false</code>则插入的每行都生成一次。<code>value</code>为字段值的生成函数。</td>
        </tr>
        <tr>
            <td><code>RawSql(force bool, batchReuse bool, rawSql func() string)</code></td><td>与<code>Value</code>方法的区别是，生成字段值为<code>rawSql</code>函数返回的原生SQL表达式。</td>
        </tr>
        <tr>
            <td rowspan="2">OnUpdate</td><td><code>Value(force bool, batchReuse bool, value func() any)</code></td><td>描述</td>
        </tr>
        <tr>
            <td><code>RawSql(force bool, batchReuse bool, rawSql func() string)</code></td><td>描述</td>
        </tr>
        <tr>
            <td rowspan="2">OnDeleteSoftly</td><td><code>PkMode[T](normalValue T)</code></td><td>描述</td>
        </tr>
        <tr>
            <td><code>NullMode[T](normalValue T)</code></td><td>描述</td>
        </tr>
    </tbody>
</table>
{{< /raw-html >}}
