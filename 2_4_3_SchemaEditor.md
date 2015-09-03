# 模式编辑器 #

`class BaseDatabaseSchemaEditor[source]`

Django的迁移系统分为两个部分；计算和储存应该执行什么操作的逻辑 (`django.db.migrations`) ，以及用于把“创建模型”或者“删除字段”变成SQL语句的数据库抽象层 -- 后者是模式编辑器的功能。

你可能并不想像一个普通的开发者使用Django那样，直接和模型编辑器进行交互，但是如果你编写自己的迁移系统，或者有更进一步的需求，这样会比编写SQL语句更方便。

每个Django的数据库后端都提供了它们自己的模式编辑器，并且总是可以通过`connection.schema_editor()`上下文管理器来访问。

```
with connection.schema_editor() as schema_editor:
    schema_editor.delete_model(MyModel)
```

它必须通过上下文管理器来使用，因为这样可以管理一些类似于事务和延迟SQL（比如创建`ForeignKey`约束）的东西。

它会暴露所有可能的操作作为方法，这些方法应该按照执行修改的顺序调用。可能一些操作或者类型并不可用于所有数据库 -- 例如，MyISAM引擎不支持外键约束。

如果你在为Django编写一个三方的数据库后端，你需要提供`SchemaEditor`实现来使用1.7的迁移功能 -- 然而，只要你的数据库在SQL的使用和关系设计上遵循标准，你就应该能够派生Django内建的`SchemaEditor`之一，然后简单调整一下语法。同时也要注意，有一些新的数据库特性是迁移所需要的：`can_rollback_ddl`和`supports_combined_alters`都很重要。

## 方法 ##

### execute ###

`BaseDatabaseSchemaEditor.execute(sql, params=[])[source]`

执行传入的 SQL语句，如果提供了参数则会带上它们。这是对普通数据库游标的一个简单封装，如果用户希望的话，它可以从`.sql`文件中获取SQL。

### create_model ###

`BaseDatabaseSchemaEditor.create_model(model)[source]`

为提供的模型在数据库中创建新的表，带有所需的任何唯一性约束或者索引。

### delete_model ###

`BaseDatabaseSchemaEditor.delete_model(model)[source]`

删除数据库中的模型的表，以及它带有的任何唯一性约束或者索引。

### alter_unique_together ###

`BaseDatabaseSchemaEditor.alter_unique_together(model, old_unique_together, new_unique_together)[source]`

修改模型的`unique_together`值；这会向模型表中添加或者删除唯一性约束，使它们匹配新的值。

### alter_index_together ###

`BaseDatabaseSchemaEditor.alter_index_together(model, old_index_together, new_index_together)[source]`

修改模型的 `index_together`值；这会向模型表中添加或者删除索引，使它们匹配新的值。

### alter_db_table ###

`BaseDatabaseSchemaEditor.alter_db_table(model, old_db_table, new_db_table)[source]`

重命名模型的表，从`old_db_table`变成`new_db_table`。

### alter_db_tablespace ###

`BaseDatabaseSchemaEditor.alter_db_tablespace(model, old_db_tablespace, new_db_tablespace)[source]`

把模型的表从一个表空间移动到另一个中。

### add_field ###

`BaseDatabaseSchemaEditor.add_field(model, field)[source]`

向模型的表中添加一列（或者有时几列），表示新增的字段。如果该字段带有`db_index=True`或者 `unique=True`，同时会添加索引或者唯一性约束。

如果字段为`ManyToManyField并`且缺少 `through`值，会创建一个表来表示关系，而不是创建一列。如果提供了`through`值，就什么也不做。

如果字段为`ForeignKey`，同时会向列上添加一个外键约束。

### remove_field ###

`BaseDatabaseSchemaEditor.remove_field(model, field)[source]`

从模型的表中移除代表字段的列，以及列上的任何唯一性约束，外键约束，或者索引。

如果字段是`ManyToManyField`并且缺少`through`值，会移除创建用来跟踪关系的表。如果提供了`through`值，就什么也不做。

### alter_field ###

`BaseDatabaseSchemaEditor.alter_field(model, old_field, new_field, strict=False)[source]`

这会将模型的字段从旧的字段转换为新的。这包括列名称的修改（`db_column`属性）、字段类型的修改（如果修改了字段类）、字段`NULL`状态的修改、添加或者删除字段层面的唯一性约束和索引、修改主键、以及修改`ForeignKey`约束的目标。

最普遍的一个不能实现的转换，是把`ManyToManyField`变成一个普通的字段，反之亦然；Django不能在不丢失数据的情况下执行这个转换，所以会拒绝这样做。作为替代，应该单独调用`remove_field()`和`add_field()`。

如果数据库满足`supports_combined_alters`，Django会尽可能在单次数据库调用中执行所有这些操作。否则对于每个变更，都会执行一个单独的`ALTER`语句，但是如果不需要做任何改变，则不执行`ALTER`（就像South经常做的那样）。

## 属性 ##

除非另有规定，所有属性都应该是只读的。

### connection ###

`SchemaEditor.connection`

一个到数据库的连接对象。`alias`是`connection`的一个实用的属性，它用于决定要访问的数据库的名字。

当你[在多种数据库之间执行迁移](http://python.usyiyi.cn/django/howto/writing-migrations.html#data-migrations-and-multiple-databases)的时候，这是非常有用的。

> 译者：[Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)，原文：[SchemaEditor](https://docs.djangoproject.com/en/1.8/ref/schema-editor/)。
>
> 本文以 [CC BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/cn/) 协议发布，转载请保留作者署名和文章出处。
>
> [Django 文档协作翻译小组](http://python.usyiyi.cn/django/index.html)人手紧缺，有兴趣的朋友可以加入我们，完全公益性质。交流群：467338606。
