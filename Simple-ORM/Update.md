# 简单ORM框架 —— 修改操作

按照前边的插入和查询操作，修改可以照葫芦画瓢，代码如下：

```csharp
public bool Update<T>(T t) where T : BaseModel
{
    if (!t.NoNullValidate())
    {
        throw new Exception("数据校验失败");
    }

    Type type = typeof(T);

    //string columnAndValueStrings = string.Join(",", type.GetPropertiesWithoutKey()
    //    .Select(x => $"{x.GetMappingName()}=@{x.GetMappingName()}"));
    //string sql = $"Update {type.GetMappingName()} Set {columnAndValueStrings} Where Id = @Id;";

    string sql = SqlCacheBuilder<T>.GetSql(SqlCacheBuilderEnum.Update);

    MySqlParameter[] parameter = type.GetPropertiesWithoutKey()
        .Select(x => new MySqlParameter($"@{x.GetMappingName()}", x.GetValue(t) ?? DBNull.Value))
        .Append(new MySqlParameter("@Id", t.Id)).ToArray();

    using (MySqlConnection conn = new MySqlConnection(ConnectionStringCustomers))
    {
        MySqlCommand command = new MySqlCommand(sql, conn);
        conn.Open();
        command.Parameters.AddRange(parameter);
        return 1 == command.ExecuteNonQuery();
    }
}
```

不同点在于，修改的时候需要带上唯一标识来定位要修改的数据，所以在sql参数化的时候需要`.Append(new MySqlParameter("@Id", t.Id))`

似乎这样就完成了，可是细细一想，目前所有的数据我只管打包发给数据库，交给数据库来处理。  
至于数据是否合法，是否是脏数据，只有等数据库那边检验完才能知道了。  
这要是请求量小就罢了，最怕是个长事务，噼里啪啦一顿操作后，最后来一个数据格式失败，全部回滚，费时又费力。

那如何才能在代码处完成数据的校验呢。

## 特性约束

---

在写之前，不妨想想需要哪些特性。  
1. 明确不能为空或空字符的特性
2. 明确在一定数字范围内的特性
3. 枚举类型的参数

等等等等。。。

考虑到避免代码冗杂，以及日后的拓展，不妨为他们写一个共同的基类

```csharp
    /// <summary>
    /// 校验基类
    /// </summary>
    [AttributeUsage(AttributeTargets.Property, AllowMultiple = true, Inherited = true)]
    public abstract class BaseValidateAttribute : Attribute
    {
        public abstract bool Validate(object oValue);
    }
```

定一个一个抽象类，子类只需要重写方法即可。并加上只可用在属性上的特性，运行重复调用以及可继承特性  
本文只给出可能会用到的三种子类，分别为：`非空限制`，`范围限制`，以及`枚举值限制`

```csharp
    /// <summary>
    /// 要求不为空
    /// </summary>
    public class NoNullAttribute : BaseValidateAttribute
    {
        public override bool Validate(object oValue)
        {
            return oValue != null && !string.IsNullOrWhiteSpace(oValue.ToString());
        }
    }

    /// <summary>
    /// 长度限制
    /// </summary>
    public class LengthAttribute : BaseValidateAttribute
    {
        private int min = 0, max = 0;

        /// <summary>
        /// 左闭右开区间
        /// </summary>
        /// <param name="min">最小值</param>
        /// <param name="max">最大值</param>
        public LengthAttribute(int min,int max)
        {
            this.max = max;
            this.min = min;
        }

        public override bool Validate(object oValue)
        {
            int length = oValue.ToString().Length;
            return oValue != null
                && !string.IsNullOrWhiteSpace(oValue.ToString())
                && length >= min
                && length < max;
        }
    }

    /// <summary>
    /// 枚举类型限制
    /// </summary>
	public class IntValueAttribute : BaseValidateAttribute
	{
		private int[] values = null;

		public IntValueAttribute(params int[] values)
		{
			this.values = values;
		}

        public override bool Validate(object oValue)
        {
			string value = oValue.ToString();
			return oValue != null
				&& !string.IsNullOrWhiteSpace(value)
				&& int.TryParse(value, out int iValue)
				&& this.values != null
				&& this.values.Contains(iValue);
        }
    }

```

接着，再为它们写一个拓展方法，代码如下：

```csharp
    public static class DateValidateExtend
    {
        public static bool NoNullValidate<T>(this T t)
        {
            Type type = typeof(T);
            foreach(var prop in type.GetProperties())
            {
                // 使用父类
                if (prop.IsDefined(typeof(BaseValidateAttribute),true))
                {
                    object oValue = prop.GetValue(t);
                    var attributeList = prop.GetCustomAttributes<BaseValidateAttribute>();

                    foreach(var attribute in attributeList)
                    {
                        if (!attribute.Validate(oValue))
                        {
                            return false;
                        }
                    }
                }
            }
            return true;
        }
    }
```

最后，只需要给需要限制的字段加上特性就可以啦
例如：`[NoNull]`,`[Length(1,4)]`,`[IntValue(1,2)]`