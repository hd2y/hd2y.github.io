---
title: "C#中实现简单爬虫项目"
date: "2019/04/10 20:55:33"
updated: "2019/07/11 11:24:26"
permalink: "implementing-a-simple-crawler-project-in-csharp/"
tags:
 - 爬虫
categories:
 - [开发, C#]
---

## 什么是爬虫

爬虫即网络爬虫（又被称为网页蜘蛛，网络机器人，在FOAF社区中间，更经常的称为网页追逐者），是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本。另外一些不常使用的名字还有蚂蚁、自动索引、模拟程序或者蠕虫。

我们日常使用的搜索引擎、比价网站等都是通过爬虫技术获取数据，而后经数据提取、分析、优化后供用户使用。当然日常生活中，我们也可以使用爬虫技术，获取我们感兴趣的网站进行数据解析分析，做出来一些有意思的功能或小工具，比如新闻提醒、事项通知、资料整理等。

这方面我们需要了解的技术主要是：Http请求、XPath、正则等。以下是一个简单的例子，用于获取京东商城的数据，例子仅供学习。

## 抓取京东所有商品类别

### 流程

- 分析京东商品类别页面
  > Chrome浏览器打开页面：https://www.jd.com/allsort.aspx
- 查看页面源代码
  > 用XPath分析商品类别信息，包括分类ID、分类名、链接，XPath语法可参考：http://www.w3school.com.cn/xpath/xpath_syntax.asp
- 获取数据
  > 使用 `System.Net.WebRequest`/`System.Net.WebClient`/`HtmlAgilityPack.HtmlWeb` 获取数据 (`HtmlAgilityPack` 可以通过 `nuget` 添加引用)
- 解析数据
  > 使用 `HtmlAgilityPack.HtmlDocument` 解析包含商品类别信息的链接

### 实现

#### 获取数据

使用 `System.Net.WebRequest/System.Net.HttpWebRequest`

```csharp
HtmlAgilityPack.HtmlDocument document = null;
WebRequest request = WebRequest.Create("https://www.jd.com/allsort.aspx");
using (WebResponse response = request.GetResponse())
{
    using (Stream stream = response.GetResponseStream())
    {
        document = new HtmlAgilityPack.HtmlDocument();
        document.Load(stream, Encoding.UTF8);
        //MemoryStream memoryStream = (MemoryStream)stream;
        //document.LoadHtml(Encoding.UTF8.GetString(memoryStream.ToArray()));
        //memoryStream.Dispose();
    }
}
```

使用 `System.Net.Client`

```csharp
HtmlAgilityPack.HtmlDocument document = null;
using (WebClient client = new WebClient())
{
    using (Stream stream = client.OpenRead("https://www.jd.com/allsort.aspx"))
    {
        document = new HtmlAgilityPack.HtmlDocument();
        document.Load(stream, Encoding.UTF8);
    }
}
```

使用 `HtmlAgilityPack.HtmlWeb`

```csharp
HtmlWeb web = new HtmlWeb
{
    OverrideEncoding = Encoding.UTF8
};
HtmlAgilityPack.HtmlDocument document = web.Load("https://www.jd.com/allsort.aspx");
```

#### 解析数据

实体

```csharp
public class Category
{
    public int Id { get; set; }
    public string JDId { get; set; }
    public string Name { get; set; }
    public string URL { get; set; }
}
```

实现

```csharp
List<Category> listCategory = new List<Category>();
if (document != null)
{
    int iId = 1;//编号
    Regex regex = new Regex("cat=(\\d+(\\,\\d+)*)");//匹配与提取类别ID正则
    //取出所有链接
    HtmlNodeCollection collection = document.DocumentNode.SelectNodes("//a[@href]");
    foreach (var item in collection)
    {
        //正则匹配出是商品类别的链接
        Match match = regex.Match(item.Attributes["href"].Value.ToLower());
        if (match.Success)
        {
            //提取商品类别ID 商品类别链接 商品类别名称
            listCategory.Add(new Category() { Id = iId++, JDId = match.Groups[1].Value, Name = item.InnerText, URL = item.Attributes["href"].Value.ToLower().StartsWith("http") ? item.Attributes["href"].Value : "https:" + item.Attributes["href"].Value });
        }
    }
}
```

## 抓取京东商品信息

### 流程

- 选择一个商品类别页面分析
  > Chrome浏览器打开页面：https://list.jd.com/list.html?cat=1713,4855,4859
- 使用Chrome分析元素
  > 使用“F12 -> Elements -> 审查元素(Ctrl+Shift+C) -> 右键 -> Copy -> Copy XPath”分析商品总页数、商品DIV解析的XPath语法
- 换页分析不同页码连接特点
  > `GET` 参数 `page` 指定页面：https://list.jd.com/list.html?cat=1713,4855,4859&page=2
- 获取数据
  > 同抓取商品类别
- 解析数据
  > 使用 `HtmlAgilityPack.HtmlDocument` 解析商品信息

### 实现

#### 获取数据

见类别获取部分实现。

#### 解析数据

实体

```csharp
public class Product
{
    public int Id { get; set; }
    public string JDId { get; set; }
    public string Name { get; set; }
    public string Price { get; set; }
    public string ImageURL { get; set; }
    public string URL { get; set; }
}
```

实现

```csharp
if (document != null)
{
    //获取页码信息
    string sCurrentPager = string.Empty;//当前页数
    string sAllPager = string.Empty;//总页数
    //List<Product> listProduct = new List<Product>();//商品
    //获取页数信息
    HtmlNodeCollection currentPagerNode = document.DocumentNode.SelectNodes("//*[@id=\"J_topPage\"]/span/b");
    if (currentPagerNode.Count != 1)
    {
        throw new Exception("加载当前页码失败！");
    }
    else
    {
        sCurrentPager = currentPagerNode[0].InnerText;
    }
    HtmlNodeCollection allPagersNode = document.DocumentNode.SelectNodes("//*[@id=\"J_topPage\"]/span/i");
    if (allPagersNode.Count != 1)
    {
        throw new Exception("加载总页数失败！");
    }
    else
    {
        sAllPager = allPagersNode[0].InnerText;
    }
    //获取商品信息
    int iId = 1;//编号
    HtmlNodeCollection productNode = document.DocumentNode.SelectNodes("//*[@id=\"plist\"]/ul/li");//符合条件的商品XPath语法
    foreach (var item in productNode)
    {
        Product product = new Product();
        HtmlAgilityPack.HtmlDocument tempDocument = new HtmlAgilityPack.HtmlDocument();
        tempDocument.LoadHtml(item.OuterHtml);
        //商品图片链接
        HtmlNodeCollection imageNode = tempDocument.DocumentNode.SelectNodes("//*[@class=\"p-img\"]/a/img");
        if (imageNode.Count > 0 && imageNode[0].Attributes["src"] != null && !string.IsNullOrEmpty(imageNode[0].Attributes   ["src"].Value))// 正常加载图片链接
        {
            product.ImageURL = imageNode[0].Attributes["src"].Value.ToLower().StartsWith("http") ? imageNode[0].Attributes ["src"].Value :    "https:" + imageNode[0].Attributes["src"].Value;
        }
        else if (imageNode.Count > 0 && imageNode[0].Attributes["data-lazy-img"] != null && !string.IsNullOrEmpty(imageNode [0].Attributes   ["data-lazy-img"].Value))//懒加载图片链接
        {
            product.ImageURL = imageNode[0].Attributes["data-lazy-img"].Value.ToLower().StartsWith("http") ? imageNode[0].Attributes   ["data- lazy-img"].Value : "https:" + imageNode[0].Attributes["data-lazy-img"].Value;
        }
        else
        {
            continue;
        }
        //商品ID
        HtmlNodeCollection idNode = tempDocument.DocumentNode.SelectNodes("//li/div");
        if (idNode.Count > 0 && idNode[0].Attributes["data-sku"] != null && !string.IsNullOrEmpty(idNode[0].Attributes["data-  sku"].Value))
        {
            product.JDId = idNode[0].Attributes["data-sku"].Value;
        }
        else
        {
            continue;
        }
        //商品名称
        HtmlNodeCollection nameNode = tempDocument.DocumentNode.SelectNodes("//*[@class=\"p-name\"]/a/em");
        if (nameNode.Count > 0 && !string.IsNullOrEmpty(nameNode[0].InnerText.Trim()))
        {
            product.Name = nameNode[0].InnerText.Trim();
        }
        else
        {
            continue;
        }
        //商品链接
        HtmlNodeCollection urlNode = tempDocument.DocumentNode.SelectNodes("//*[@class=\"p-name\"]/a");
        if (urlNode.Count > 0 && urlNode[0].Attributes["href"] != null && !string.IsNullOrEmpty(urlNode[0].Attributes["href"].Value))
        {
            product.URL = urlNode[0].Attributes["href"].Value.ToLower().StartsWith("http") ? urlNode[0].Attributes["href"].Value :    "https:"  + urlNode[0].Attributes["href"].Value;
        }
        else
        {
            continue;
        }
        product.Id = iId++;
        listProduct.Add(product);
    }
}
```

### 注意事项

- 图片懒加载，如果解析src属性可能解析不到，需要调试查看懒加载图片属性。
- 价格信息抓取会发现为空，观察会发现其calss属性为“J_price”，怀疑为ajax请求获取，下一部分说明获取方法。

## 抓取京东商品价格信息

### 流程

- 使用Chrome分析网络请求
  > 使用“F12 -> Network -> JS”分析页面刷新时的ajax请求，发现：https://p.3.cn/prices/mgets 的请求，怀疑为查询价格的请求信息
- 分析请求链接
  > 过滤GET参数，发现传递参数skuIds即可获取信息，多个用“%2C”分隔即可，例如：https://p.3.cn/prices/mgets?skuIds=J_4609652%2CJ_5830869 ，获取的数据位JSON格式
- 获取数据
  > 同抓取商品类别
- 解析数据
  > 使用 `Newtonsoft.Json` 解析(`Newtonsoft.Json` 可以通过 `nuget` 添加引用)

### 实现

#### 获取数据

见类别获取部分实现。

#### 解析数据

```csharp
/// <summary>
/// 通过京东商品ID获取价格
/// </summary>
/// <param name="listJDId">商品ID</param>
/// <returns>商品ID对应价格的字典</returns>
private Dictionary<string, string> GetPrice(List<string> listJDId)
{
    Dictionary<string, string> dicPrice = new Dictionary<string, string>();
    if (listJDId.Count > 0)
    {
        //按照ajax请求连接格式拼接skuID
        for (int i = 0; i < listJDId.Count; i++)
        {
            listJDId[i] = "J_" + listJDId[i];
        }
        //循环每10条查询一次 避免数量过多链接过长
        for (int i = 0; i < (listJDId.Count + 9) / 10; i++)
        {
            //获取数据
            HtmlAgilityPack.HtmlDocument document = new HtmlAgilityPack.HtmlDocument();
            WebRequest request = WebRequest.Create("https://p.3.cn/prices/mgets?skuIds=" + string.Join("%2C", listJDId.Skip(i * 10).Take(10)));
            using (Stream stream = request.GetResponse().GetResponseStream())
            {
                document.Load(stream, Encoding.UTF8);
            }
            //解析数据
            if (!string.IsNullOrEmpty(document.Text))
            {
                object oPrice = Newtonsoft.Json.JsonConvert.DeserializeObject(document.Text);//转换成对象
                if (oPrice is JArray)
                {
                    foreach (var item in oPrice as JArray)
                    {
                        if (item is JObject && (item as JObject).ContainsKey("id") && (item as JObject).ContainsKey("p"))
                        {
                            //提取符合条件的数据
                            if ((item as JObject)["id"] is JValue jId && (item as JObject)["p"] is JValue jPrice && listJDId.Contains(jId.Value.ToString()) && !dicPrice.ContainsKey(jId.Value.ToString().TrimStart('J', '_')))
                            {
                                dicPrice.Add(jId.Value.ToString().TrimStart('J', '_'), jPrice.Value.ToString());
                            }
                        }
                    }
                }
            }
        }
    }
    return dicPrice;
}
```

> 若想使用jQuery选择器可以结合Fizzlerex使用：https://archive.codeplex.com/?p=fizzlerex
