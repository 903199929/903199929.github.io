# 简单ORM框架 —— 条件查询

到目前为止，增删改查都实现了。  
那剩下的就是代码上的优化了，目前的查询操作只支持根据`id`进行查询，并且代码部分是被写死的拓展性太差  
有没有办法根据写入的条件进行查询呢？  
有！

逻辑是将需要查询的条件通过表达式树，将linq语句翻译为sql语句，再将sql动态拼接起来，从而达到条件查询的效果。  
先上代码。

```csharp
    public class CustomExpressionVisitor : ExpressionVisitor
    {
        private Stack<string> ConditionStack = new Stack<string>();
        private Dictionary<string, string> ValueDic = new Dictionary<string, string>();
        private Dictionary<string, int> counter = new Dictionary<string, int>();
        private Stack<string> ValueStack = new Stack<string>(1);

        public string GetWhere()
        {
            string where = string.Concat(ConditionStack.ToArray());
            ConditionStack.Clear();
            return where;
        }

        public Dictionary<string, string> GetValuesDic()
        {
            Dictionary<string, string> dic = ValueDic;
            return dic;
        }

        [return: NotNullIfNotNull("node")]
        public override Expression? Visit(Expression? node)
        {
            return base.Visit(node);
        }

        protected override Expression VisitBinary(BinaryExpression node)
        {
            //ConditionStack.Push(" ) ");
            ConditionStack.Push(" ");
            base.Visit(node.Left);
            ConditionStack.Push(node.NodeType.ToSqlOperator());
            base.Visit(node.Right);
            //ConditionStack.Push(" ( ");
            ConditionStack.Push(" ");
            return node;
        }

        protected override Expression VisitConstant(ConstantExpression node)
        {
            string key = ValueStack.Pop();

            ConditionStack.Push(key);
            ValueDic.Add(counter[key] > 1 ? key + counter[key] : key, node.Value.ToString());

            return node;
        }

        protected override Expression VisitMember(MemberExpression node)
        {
            string key = node.Member.Name;
            if (counter.ContainsKey(key))
            {
                counter[key]++;
                key += counter[key];
                ConditionStack.Push($"@{key}");
            }
            else
            {
                counter.Add(key, 1);
                ConditionStack.Push($"@{node.Member.Name}");
            }
            ValueStack.Push(node.Member.Name);
            return node;
        }

        protected override Expression VisitMethodCall(MethodCallExpression expression)
        {
            if (expression == null) throw new ArgumentNullException("MethodCallExpression");

            if (expression.Method.Name == "StartsWith" || expression.Method.Name == "Contains" || expression.Method.Name == "EndsWith")
            {
                Visit(expression.Object);
                Visit(expression.Arguments[0]);
                string right = ConditionStack.Pop();
                string left = ConditionStack.Pop();
                ConditionStack.Push($" {right} Like {left} ");

                string current = counter[right] > 1 ? right + counter[right] : right;

                switch (expression.Method.Name)
                {
                    case "StartsWith":
                        ValueDic[current] = ValueDic[current] + "%";
                        break;
                    case "Contains":
                        ValueDic[current] = "%" + ValueDic[current] + "%";
                        break;
                    case "EndsWith":
                        ValueDic[current] = "%" + ValueDic[current];
                        break;
                }

                return expression;
            }
            else
            {
                throw new NotSupportedException(expression.NodeType + "is not supported");
            }
        }
    }

    // 拓展方法
    internal static class SqlOperator
    {
        internal static string ToSqlOperator(this ExpressionType type)
        {
            switch (type)
            {
                case (ExpressionType.AndAlso):
                case (ExpressionType.And):
                    return "AND";
                case (ExpressionType.OrElse):
                case (ExpressionType.Or):
                    return "OR";
                case (ExpressionType.Not):
                    return "NOT";
                case (ExpressionType.NotEqual):
                    return "<>";
                case (ExpressionType.GreaterThan):
                    return ">";
                case (ExpressionType.GreaterThanOrEqual):
                    return ">=";
                case (ExpressionType.LessThan):
                    return "<";
                case (ExpressionType.LessThanOrEqual):
                    return "<=";
                case (ExpressionType.Equal):
                    return "=";
                default:
                    throw new Exception("不支持该方法");
            }
        }
    }
```

1. 引用`System.Linq.Expressions`包并继承`ExpressionVisitor`类
2. `GetWhere()`方法返回`sql`中`where`部分的字符串
3. `GetValuesDic()`为一个字典，用于返回`Sql`参数化中的键值对
4. `Visit()`方法调用`ExpressionVisitor`中的父类方法，可以根据传入类型对应到需要处理的模块中
5. `VisitBinary()`方法中定义了表达式树的执行顺序，并把每个叶节点中的值存入栈中，又因为栈后入先出的特性，刚好能将`Sql`按正常的顺序拼接出来
6. `VisitBinary()`中调用了拓展方法`ToSqlOperator()`用于把`linq`的表达式类型转换为`sql`
7. 根据执行顺序，会先执行`VisitMember()`方法，拿到字段名。考虑到同一个字段可能会有多个筛选条件，例如要同时满足`Name`长度要在4到6之间，写成Linq为`x=>x.Name >= 4 && x.Name <= 6`,由于要进行`Sql`参数化，把字段和值都存在了字典里，会导致因为有相同的`Key`导致报错。所以定义了一个字典`Counter`用于计每个字段所用到的次数，对于重复使用的字段，如：`Name`，会将字段命名为为`Name+使用次数`的形式来解决`key`冲突
8. `VisitBinary()`调用完成后，接着会调用`VisitConstant()`方法用于获取字段的值。为了绑定字段和值，用定义了一个长度为1的栈，用于将`VisitBinary()`中的字段名传到`VisitConstant()`中用来赋值给字典。同时在存入字典的时候也需要对`key`值是否重复进行判断
9. `VisitMethodCall`方法，用来处理`StartsWith`,`Contains`,`EndsWith`等操作，此处有坑，参数化的时候必须处理为`(@Name, "%" + Value + "%")`，sql处理为 `Name like @Name`的形式，不然可能导致编译通过但是没有值。

最后将`SqlHelper`中加入一个条件查询方法
```csharp
    public T FindByCondition<T>(Expression<Func<T, bool>> expression) where T : BaseModel //泛型约束保证类型正确
    {
        Type type = typeof(T);

        CustomExpressionVisitor visitor = new CustomExpressionVisitor();

        visitor.Visit(expression);
        string where = "where" + visitor.GetWhere();
        Dictionary<string, string> dic = visitor.GetValuesDic();

        // 从sql缓存中获取
        string sql = SqlCacheBuilder<T>.GetSql(SqlCacheBuilderEnum.Search) + where;

        // sql参数化
        MySqlParameter[] parameters = dic.Select(x => new MySqlParameter($"@{x.Key}", x.Value)).ToArray();

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
    }

    // 缓存类中的sql语句修改
    searchSql = $@"SELECT {colunmStrings} From {type.GetMappingName()} ";
```

这样条件查询就完成了，但是目前只支持查询对象，不支持多条数据查询