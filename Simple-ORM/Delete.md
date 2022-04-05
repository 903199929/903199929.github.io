# 简单ORM框架 —— 删除操作

太多相似啦！先贴代码，再想想如何优化

```csharp
public bool Delete<T>(T t) where T : BaseModel
{
    Type type = typeof(T);
    string sql = SqlCacheBuilder<T>.GetSql(SqlCacheBuilderEnum.Delete);
    MySqlParameter[] parameter = new MySqlParameter[] { new MySqlParameter("@Id", t.Id) };

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        conn.Open();
        command.Parameters.AddRange(parameter);
        return 1 == command.ExecuteNonQuery();
    }
}

// 缓存类中的方法
// Delete
{
    Type type = typeof(T);
    DeleteSql = $"Delete From {type.GetMappingName()} where Id = @Id";
}
```

```csharp
    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        conn.Open();
        command.Parameters.AddRange(parameter);
        return 1 == command.ExecuteNonQuery();
    }
```

这一部分的代码，四个方法内部都有，似乎重复的太多了，能不能优化一下呢？

能！

首先新建一个方法，把公共的部分写进去

```csharp
private S ExecuteSql<S>(string sql, MySqlParameter[] parameters, Func<MySqlCommand, S> func)
{
    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        command.Parameters.AddRange(parameters);
        conn.Open();
        return func.Invoke(command);
    }
}
```

其次，将原先多余的代码给注释掉，并把不相同的部分以委托的形式实现。

```csharp
// 增删改的方法是一样的
return this.ExecuteSql<bool>(sql, parameters, command => 1 == command.ExecuteNonQuery());

// 查询特别一些
return this.ExecuteSql<T>(sql, parameters, command =>
{
    var reader = command.ExecuteReader();
    if (reader.Read())
    {
        // 创建实体
        T t = Activator.CreateInstance<T>();

        foreach (var prop in type.GetProperties())
        {
            string propName = prop.GetMappingName();

            prop.SetValue(t, reader[propName] is DBNull ? null : reader[propName]);
        }
        return t;
    }
    else
    {
        return null;
    }
});
```

搞定!这样代码就简洁了许多