```markdown

#region Mapper

public static IEnumerable<T> ReaderTo<T>(this IDataReader dataReader)
{
    System.Diagnostics.Stopwatch sw = new System.Diagnostics.Stopwatch();
    sw.Start();
    var mapper = CreateMappingFunction(dataReader, typeof(T));
    while (dataReader.Read())
    {
        var result = mapper(dataReader as DbDataReader);
        yield return result is T ? (T)result : default(T);
    }
    sw.Stop();
    Console.WriteLine("耗时: {0} 毫秒.", sw.ElapsedMilliseconds);
}

private static Func<DbDataReader, object> CreateMappingFunction(IDataReader reader, Type type)
{
    //1. 取得sql select所有栏位名称
    var names = Enumerable.Range(0, reader.FieldCount).Select(index => reader.GetName(index)).ToArray();

    //2. 取得mapping类别的属性资料 >  将index,sql栏位,class属性资料做好对应封装在一个变量内方便后面使用
    var props = type.GetProperties().ToList();
    var members = names.Select((columnName, index) =>
    {
        var property = props.Find(p => string.Equals(p.Name, columnName, StringComparison.OrdinalIgnoreCase));
        return new
        {
            index,
            columnName,
            property
        };
    }).Where(x => x.property != null);

    //3. 动态建立方法 : 从数据库Reader按照顺序读取我们要的资料
    /*方法逻辑 : 
        User 动态方法(IDataReader reader)
        {
        var user = new User();
        var value = reader[0];
        if( !(value is System.DBNull) )
            user.Name = (string)value;
        value = reader[1];
        if( !(value is System.DBNull) )
            user.Age = (int)value;  
        return user;
        }
    */
    var exBodys = new List<Expression>();

    // 方法(IDataReader reader)
    var exParam = Expression.Parameter(typeof(DbDataReader), "reader");

    // Mapping类别 物件 = new Mapping类别();
    var exVar = Expression.Variable(type, "mappingObj");
    var exNew = Expression.New(type);
    exBodys.Add(Expression.Assign(exVar, exNew));

    // var value = defalut(object);
    var exValueVar = Expression.Variable(typeof(object), "value");
    exBodys.Add(Expression.Assign(exValueVar, Expression.Constant(null)));

    var getItemMethod = typeof(DbDataReader)
        .GetMethods()
        .Where(w => w.Name == "get_Item")
        .First(w => w.GetParameters().First().ParameterType == typeof(int));

    foreach (var m in members)
    {
        if (m.property == null)
            continue;
        //reader[0]
        var exCall = Expression.Call(
            exParam,
            getItemMethod,
            Expression.Constant(m.index)
        );

        // value = reader[0];
        exBodys.Add(Expression.Assign(exValueVar, exCall));

        //user.Name = (string)value;
        var exProp = Expression.Property(exVar, m.property.Name);

        //https://stackoverflow.com/questions/2850265/calling-a-generic-method-using-lambda-expressions-and-a-type-only-known-at-runt
        var exParseCall = Expression.Call(typeof(ExpressionMapper), "Parse", new Type[] { m.property.PropertyType }, exValueVar);               
        var assign = Expression.Assign(exProp, exParseCall);
        var exIfThenElse = Expression.IfThenElse(
                            Expression.TypeIs(exValueVar, typeof(System.DBNull)), Expression.Default(m.property.PropertyType), assign
                        );

        exBodys.Add(exIfThenElse);
    }

    // return user;  
    exBodys.Add(exVar);

    // Compiler Expression 
    var lambda = Expression.Lambda<Func<DbDataReader, object>>(
        Expression.Block(
        new[] { exVar, exValueVar },
        exBodys
        ), exParam
    );

    return lambda.Compile();
}

public static T Parse<T>(object value)
{
    if (value == null || value == DBNull.Value)
        return default(T);

    if (value is T)
        return (T)value;

    var type = typeof(T);
    type = Nullable.GetUnderlyingType(type) ?? type;
    if (type.IsEnum)
    {
        if (value is float || value is double || value is decimal)
        {
            value = Convert.ChangeType(value, Enum.GetUnderlyingType(type), CultureInfo.InvariantCulture);
        }
        return (T)Enum.ToObject(type, value);
    }
    //if (typeHandlers.TryGetValue(type, out ITypeHandler handler))
    //{
    //    return (T)handler.Parse(type, value);
    //}
    return (T)Convert.ChangeType(value, type, CultureInfo.InvariantCulture);
}

#endregion

```
