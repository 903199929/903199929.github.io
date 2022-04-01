# 简单ORM框架 —— 插入操作

书接上回，废话不多说，直接亮代码。在SqlHelper中新加一个Insert方法

```csharp
public bool Insert<T>(T t) where T : BaseModel
{
    Type type = typeof(T);
    string columnStrings = string.Join(",", type.GetProperties().Select(x => $"{x.GetMappingName()}"));

    string valueStrings = string.Join(",", type.GetProperties().Select(x => $"{x.GetValue(t)}"));

    string sql = $"Insert into `{type.GetMappingName()}` ({columnStrings}) Values ({valueStrings})";

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        conn.Open();
        int result = command.ExecuteNonQuery();
        return result == 1;
    }
}
```

cv大法，唰唰唰搞定。点击运行，报错！一气呵成。  
定睛一看，噢！`type.GetProperties()`把主键也作为参数拼接进sql了，我们的数据库的主键又是自增的。  
知道了问题所在，那就想办法把主键给剔除出去。

## 主键特性

---

首先，新建一个主键特性
```csharp
[AttributeUsage(AttributeTargets.Property)]
public class KeyAttribute : Attribute
{
    public KeyAttribute()
    {
    }
}
```

接着，新建一个拓展方法，用于把主键给剔除

```csharp
public static class DBFilterExtend
{
    public static IEnumerable<PropertyInfo> GetPropertiesWithoutKey(this Type type)
    {
        return type.GetProperties().Where(x => !x.IsDefined(typeof(KeyAttribute), true));
    }
}
```

最后，给主键加上特性，并把`GetProperties()`方法改为拓展方法`GetPropertiesWithoutKey()`。

```csharp
public class BaseModel
{
    [Key]
    public int Id { get; set; }
}
```

修改完成，点击运行。  
又报错啦！
sql语句执行失败，分析原因后，原来是string类型的sql在执行时需要加上`''`，除此之外，如此的拼接字符串有可能会导致Sql注入问题。

## Sql参数化，解决Sql注入问题

---

```csharp
public bool Insert<T>(T t) where T : BaseModel
{
    Type type = typeof(T);
    string columnStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"{x.GetMappingName()}"));
    // sql参数化，防止sql注入
    string valueStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"@{x.GetMappingName()}"));
    var parameters = type.GetProperties().Select(x => new MySqlParameter($"@{x.GetMappingName()}", x.GetValue(t) ?? DBNull.Value)).ToArray();

    string sql = $"Insert into `{type.GetMappingName()}` ({columnStrings}) Values ({valueStrings})";

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        command.Parameters.AddRange(parameters);
        conn.Open();
        int result = command.ExecuteNonQuery();
        return result == 1;
    }
}
```

代码如上，注意事项如下：
1. `valueStrings`获取的`Select`语句由原来的`Select(x => $"{x.GetValue(t)}")`修改为`Select(x => $"@{x.GetMappingName()}")` 此处多了一个`@`,且方法也修改了。
2. 获取参数数组，`var parameters = type.GetProperties().Select(x => new MySqlParameter($"@{x.GetMappingName()}", x.GetValue(t) ?? DBNull.Value)).ToArray()`此处，如果程序内的值是`null`，在数据库赋值的时候会报错，所以在此处用`x.GetValue(t) ?? DBNull.Value)`预先处理一下
3. 在执行sql语句前加入参数`command.Parameters.AddRange(parameters);`

此时再运行就会发现，问题都已经解决啦！  
虽然用是能用了，但是总感觉效率太差。  
在每一次调用插入方法的时候都要动态拼装Sql从而反复调用`GetPropertiesWithoutKey()`和`GetMappingName()`感觉挺影响性能的,尤其是不断操作同一个表的时候，有什么办法可以优化一下吗？

有！

## 泛型缓存

---

使用缓存的话，就只需要在第一次调用的时候执行，以后的调用直接访问缓存即可。  
泛型缓存和字典缓存都不错但是此处用的是泛型缓存  
首先，将SqlHelper中的部分代码复制起来，然后新建一个缓存类

```csharp
public class SqlCacheBuilder<T> where T : BaseModel
{

    private static string insertSql = null;

    static SqlCacheBuilder()
    {
        #region Insert
        Type type = typeof(T);
        string columnStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"{x.GetMappingName()}"));
        // sql参数化，防止sql注入
        string valueStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"@{x.GetMappingName()}"));

        insertSql = $"Insert into `{type.GetMappingName()}` ({columnStrings}) Values ({valueStrings})";
        #endregion
    }

    public static string GetSql()
    {
        return insertSql;
    }
}
```

其次，将SqlHelper中的重复代码注释掉

```csharp
public bool Insert<T>(T t) where T : BaseModel
{
    Type type = typeof(T);

    //string columnStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"{x.GetMappingName()}"));
    //string valueStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"@{x.GetMappingName()}"));
    //string sql = $"Insert into `{type.GetMappingName()}` ({columnStrings}) Values ({valueStrings})";

    string sql = SqlCacheBuilder<T>.GetSql();
    // sql参数化，防止sql注入
    var parameters = type.GetProperties().Select(x => new MySqlParameter($"@{x.GetMappingName()}", x.GetValue(t) ?? DBNull.Value)).ToArray();

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        command.Parameters.AddRange(parameters);
        conn.Open();
        int result = command.ExecuteNonQuery();
        return result == 1;
    }
}
```

大功告成，由于`static`特性，只有在第一次运行代码的时候会执行`private static string insertSql = null;`，而泛型的特性由保证不同的类之间相互独立，从而实现缓存。

同理，之前的查询也可以写入缓存，代码如下

```csharp
public class SqlCacheBuilder<T> where T : BaseModel
{

    private static string searchSql = null;
    private static string insertSql = null;


    static SqlCacheBuilder()
    {
        {
            Type type = typeof(T);

            // 通过拓展方法获取特性里的名字
            string colunmStrings = string.Join(",", type.GetProperties().Select(x => $"{x.GetMappingName()}"));

            searchSql = $@"SELECT {colunmStrings} From {type.GetMappingName()} WHERE Id = @Id";
        }

        {
            #region Insert
            Type type = typeof(T);
            string columnStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"{x.GetMappingName()}"));
            // sql参数化，防止sql注入
            string valueStrings = string.Join(",", type.GetPropertiesWithoutKey().Select(x => $"@{x.GetMappingName()}"));

            insertSql = $"Insert into `{type.GetMappingName()}` ({columnStrings}) Values ({valueStrings})";
            #endregion
        }
    }

    public static string GetSql(SqlCacheBuilderEnum type)
    {
        switch (type)
        {
            case (SqlCacheBuilderEnum.Search):
                return searchSql;
            case (SqlCacheBuilderEnum.Insert):
                return insertSql;
            default:
                throw new Exception("Unkown SqlCacheType");
        }
        return insertSql;
    }
}

public enum SqlCacheBuilderEnum
{
    Search,
    Insert
}
```

使用了枚举类型来判断需要的是查询还是插入sql，最后将`SqlHelper`方法修改为如下：

```csharp
public T Find<T>(int id) where T : BaseModel //泛型约束保证类型正确
{
    Type type = typeof(T);

    // 从sql缓存中获取
    string sql = SqlCacheBuilder<T>.GetSql(SqlCacheBuilderEnum.Search);
    // sql参数化
    MySqlParameter[] parameters = new MySqlParameter[] { new MySqlParameter("@Id", id) };

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        conn.Open();
        command.Parameters.AddRange(parameters);
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
    }
}
```