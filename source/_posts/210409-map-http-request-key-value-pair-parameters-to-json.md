---
title: "映射 HTTP 请求入参键值对到 JSON"
date: "2021/04/08 19:24:40"
updated: "2021/04/08 19:24:40"
permalink: "map-http-request-key-value-pair-parameters-to-json/"
tags:
 - json
categories:
 - [开发, C#]
---

## 前言

当对接一些 HTTP API 时，如果请求参数使用 `GET` 传递，或者使用 `POST` 但是入参格式为 `x-www-form-urlencoded`，如果参数较多时，可能需要将内容作为对象输入或输出。

最近有一个接口对接，对方参数达到一百多个，市面上找不到合适的工具来进行解析，所以尝试自己手撸一个工具类来用。

## 键值对参数到 JSON 对象

### 分析

打开浏览器控制台，引用 `jQuery` 提交一个 `ajax` 请求，查看请求内容：

```js
var obj = {
    c: [[1, 2, 3], [4, 5, 6, {
        d: 1,
        e: 2
    }]]
};

$.ajax({
    url: 'https://www.baidu.com',
    data: obj
});
```

然后查看网络窗口的请求信息，可以找到请求的查询字符串参数：

```html
c[0][]: 1
c[0][]: 2
c[0][]: 3
c[1][]: 4
c[1][]: 5
c[1][]: 6
c[1][3][d]: 1
c[1][3][e]: 2
```

根据浏览器中解析规则，数据一定是一个对象，入参信息在属性中，属性内容可以是数值、字符串、对象、数组等，可以嵌套。

开始内容一定是一个属性名，后续内容如果是数组，中括号中将体现为数字，并且数组中每个值拆分为一个键值对，键以 `[]` 结尾。如果是对象属性，中括号中内容为属性名称。

由此，可以写出一个简单的正则表达式匹配这个规则：`^(?<n1>[^\[\]]+?)(?<n2>(\[([^\[\]]+?)\])*)(?<n3>\[\])?$`。

其中提取项目 `n1` 是属性名，`n2` 为多层嵌套的数组下标或属性名，`n3` 内容用于判断该值是否为数组对象。

注意我们需要解析的是源信息，调整为 `查看源` 可以获取：`c%5B0%5D%5B%5D=1&c%5B0%5D%5B%5D=2&c%5B0%5D%5B%5D=3&c%5B1%5D%5B%5D=4&c%5B1%5D%5B%5D=5&c%5B1%5D%5B%5D=6&c%5B1%5D%5B3%5D%5Bd%5D=1&c%5B1%5D%5B3%5D%5Be%5D=2`。

### 实现

思路为解析每个键值对内容，根据其嵌套关系设置每层属性的信息，内容保存到 `ExpandoObject`（其本质是一个 `IDictionary<string, object>`），每解析一个与上一个键值对解析到的内容进行合并。

所有键值对内容解析完成后返回，因为使用的是 `ExpandoObject` 对象保存信息，作为 `dynamic` 返回后可以方便后续使用或反序列化为 Json 对象。

解析代码如下：

```csharp
public static dynamic ToQueryObject(this string query)
{
    if (query == null || string.IsNullOrWhiteSpace(query)) return null;

    IDictionary<string, object> obj = null;
    var collection = HttpUtility.ParseQueryString(query);
    for (int i = 0; i < collection.Keys.Count; i++)
    {
        // 偷懒的写法，解析后再合并，相对容易理解
        var key = collection.Keys[i];
        var values = collection.GetValues(key);
        obj = obj.Combine(GetDictionary(key, values));
    }

    return obj;
}

private static IDictionary<string, object> GetDictionary(string key, string[] values)
{
    if (key == null)
    {
        throw new ArgumentNullException(nameof(key));
    }

    // 用于匹配属性或数组的正则
    var regex = new Regex(@"^(?<n1>[^\[\]]+?)(?<n2>(\[([^\[\]]+?)\])*)(?<n3>\[\])?$");
    var match = regex.Match(key);

    if (!match.Success)
    {
        throw new Exception($"Property \"{match}\" failed to match.");
    }
    else
    {
        // 解析 Key 内容
        var firstKey = match.Result("${n1}");
        var lastKey = match.Result("${n2}");
        bool isAarry = match.Result("${n3}") == "[]";

        if (!isAarry && values.Length > 1)
        {
            throw new Exception($"Property \"{key}\" value are not allowed to be arrays.");
        }
        else
        {
            object value = isAarry ? new List<object>(values) : (values.Length > 0 && !string.IsNullOrEmpty(values[0]) ? values[0] : null);

            // 属性路径上所有的内容
            var keys = new List<string> { firstKey };
            if (!string.IsNullOrEmpty(lastKey))
            {
                keys.AddRange(lastKey.Trim('[', ']').Split("]["));
            }

            // 由内向外迭代赋值
            for (int i = keys.Count - 1; i >= 0; i--)
            {
                if (i != 0 && int.TryParse(keys[i], out var index))
                {
                    // 如果是数字则说明是索引
                    var list = new List<object>(new object[index + 1]);
                    list[index] = value;
                    value = list;
                }
                else
                {
                    // 否则说明是属性
                    IDictionary<string, object> dict = new ExpandoObject();
                    dict.TryAdd(keys[i], value);
                    value = dict;
                }
            }

            return value as IDictionary<string, object>;
        }
    }
}

private static IDictionary<string, object> Combine(this IDictionary<string, object> target, IDictionary<string, object> from)
{
    if (target == null)
    {
        target = from;
    }
    else if (target != null && from != null)
    {
        foreach (var fromItem in from)
        {
            if (fromItem.Value == null) continue;
            if (target.TryGetValue(fromItem.Key, out object targetValue) && targetValue != null)
            {
                if (targetValue is List<object> targetList && fromItem.Value is List<object> fromList)
                {
                    // 合并集合
                    var list = targetList.Combine(fromList);
                    target[fromItem.Key] = list;
                }
                else if (targetValue is IDictionary<string, object> targetDict && fromItem.Value is IDictionary<string, object> fromDict)
                {
                    // 合并字典
                    var dict = targetDict.Combine(fromDict);
                    target[fromItem.Key] = dict;
                }
                else
                {
                    throw new Exception($"Type \"{targetValue.GetType()}\" and type \"{fromItem.Value.GetType()}\" cannot be merged.");
                }
            }
            else
            {
                target[fromItem.Key] = fromItem.Value;
            }
        }
    }

    return target;
}

private static List<object> Combine(this List<object> target, List<object> from)
{
    if (target == null)
    {
        target = from;
    }
    else if (target != null && from != null)
    {
        for (int i = 0; i < Math.Max(target.Count, from.Count); i++)
        {
            var targetItem = target.Count > i ? target[i] : null;
            var fromItem = from.Count > i ? from[i] : null;
            if (targetItem == null || fromItem == null)
            {
                var value = targetItem ?? fromItem;
                if (target.Count > i) target[i] = value;
                else target.Add(value);
            }
            else if (targetItem is List<object> targetList && fromItem is List<object> fromList)
            {
                // 合并集合
                var list = targetList.Combine(fromList);
                if (target.Count > i) target[i] = list;
                else target.Add(list);
            }
            else if (targetItem is IDictionary<string, object> targetDict && fromItem is IDictionary<string, object> fromDict)
            {
                // 合并字典
                var dict = targetDict.Combine(fromDict);
                if (target.Count > i) target[i] = dict;
                else target.Add(dict);
            }
            else
            {
                throw new Exception($"Type \"{targetItem.GetType()}\" and type \"{fromItem.GetType()}\" cannot be merged.");
            }
        }
    }

    return target;
}
```

### 测试

参考文章开头请求的入参，进行测试：

```csharp
var query = "c%5B0%5D%5B%5D=1&c%5B0%5D%5B%5D=2&c%5B0%5D%5B%5D=3&c%5B1%5D%5B%5D=4&c%5B1%5D%5B%5D=5&c%5B1%5D%5B%5D=6&c%5B1%5D%5B3%5D%5Bd%5D=1&c%5B1%5D%5B3%5D%5Be%5D=2";
var obj = query.ToQueryObject();
var json = JsonConvert.SerializeObject(obj, Formatting.Indented);

// 输出内容为：
// {
//   "c": [
//     [
//       "1",
//       "2",
//       "3"
//     ],
//     [
//       "4",
//       "5",
//       "6",
//       {
//         "d": "1",
//         "e": "2"
//       }
//     ]
//   ]
// }
```

## JSON 对象到键值对参数

### 分析

同样的，我们解析后的对象，在填值以后，在请求 HTTP API 后需要按照以上键值对的格式输出。

这时我们需要使用 `Newtonsoft.Json` 中的 `JToken` 将对象进行解析。这里有个好处，在解析时如果对象属性设置了别名，会自动将键中的属性名映射为别名。

### 实现

实现思路就是递归，逐层获取属性或数组内容，判断当内容为 `JObject` 或 `JArray` 继续获取成员值，并将上级内容作为前缀内容拼接作为参数传递。

当递归到所属层级为 `JValue` 时，拼接前缀内容进行返回，需要注意的是数组内容要根据下级内容是否为 `JValue` 判断中括号中内容是否应该给数组下标。

```csharp
public static string ToQueryString<T>(this T @this) where T : class
{
    var queries = ToQueryString("", JToken.FromObject(@this));
    return queries.Any() ? $"?{string.Join("&", queries)}" : "";
}

private static IEnumerable<string> ToQueryString(string prefix, JToken token)
{
    if (token is JObject obj)
    {
        foreach (var property in obj.Properties())
        {
            var subToken = obj[property.Name];
            var name = string.IsNullOrEmpty(prefix) ? property.Name : $"{prefix}[{property.Name}]";
            foreach (var item in ToQueryString(name, subToken))
            {
                yield return item;
            }
        }
    }
    else if (!string.IsNullOrEmpty(prefix))
    {
        if (token is JValue value)
        {
            yield return $"{Uri.EscapeDataString(prefix)}={Uri.EscapeDataString(value.ToString())}";
        }
        else if (token is JArray array)
        {
            for (int i = 0; i < array.Count; i++)
            {
                var subToken = array[i];
                var name = $"{prefix + (subToken is JValue ? "[]" : $"[{i}]")}";
                foreach (var item in ToQueryString(name, array[i]))
                {
                    yield return item;
                }
            }
        }
    }
}
```

### 测试

同样，使用前文中对象测试输出是否与浏览器一致：

```csharp
var obj = new
{
    c = new object[]
    {
        new[] { 1, 2, 3, },
        new object[]
        { 
            4, 5, 6,
            new
            {
                d = 1,
                e = 2
            }
        }
    }
};
var query = obj.ToQueryString();

// 输出内容为：
// ?c%5B0%5D%5B%5D=1&c%5B0%5D%5B%5D=2&c%5B0%5D%5B%5D=3&c%5B1%5D%5B%5D=4&c%5B1%5D%5B%5D=5&c%5B1%5D%5B%5D=6&c%5B1%5D%5B3%5D%5Bd%5D=1&c%5B1%5D%5B3%5D%5Be%5D=2
```

测试属性名设置特性后的输出效果：

```csharp
public class Person
{
    [JsonProperty("first_name")]
    public string FirstName { get; set; }

    [JsonProperty("last_name")]
    public string LastName { get; set; }

    [JsonProperty("full_name")]
    public string FullName { get; set; }

    [JsonProperty("age")]
    public int? Age { get; set; }
}

var obj = new Person
{
    FirstName = "John",
    LastName = "Sun",
    FullName = "John Sun",
    Age = 3
};
var query = obj.ToQueryString();

// 输出内容为：
// ?first_name=John&last_name=Sun&full_name=John%20Sun&age=3
```
