---
title: "映射 JSON 文件的实体类"
date: "2021/04/06 19:24:40"
updated: "2021/04/06 19:24:40"
permalink: "map-json-file-to-entity-class/"
tags:
 - json
categories:
 - [开发, C#]
---

## 前言

想要使用接口返回的内容，直接生成实体类，用于后期开发。

如果简单使用可以参考：http://tools.jb51.net/code/json2csharp/

但是因为其生成的属性名不规范，以及对于特殊属性名例如 `Fflag.appType.brands` 没有做好处理，所以自己写了一个工具类。

## 实现代码

首先需要引用 `Newtonsoft.Json`，`json` 文件的解析，以及生成实体的内容都依赖这个包。

另外为了处理单复数问题，还引用了 `PluralizeService.Core`。

工具类代码内容如下：

```csharp
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using PluralizeService.Core;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;

namespace Utils
{
    /// <summary>
    /// Json 工具类
    /// </summary>
    public static class JsonUtil
    {
        /// <summary>
        /// 将 Json 内容转换为 C# 实体类
        /// </summary>
        /// <param name="json">json 内容</param>
        /// <returns>转化后的实体类内容</returns>
        public static string ToEntityClass(string json)
        {
            var obj = JsonConvert.DeserializeObject(json);

            var entities = new Dictionary<string, EntityInfo>();
            MapEntityInfo("BaseEntity", entities, obj as JToken);

            var content = new StringBuilder();

            foreach (var entity in entities)
            {
                // 开始
                content.AppendLine($"public class {PluralizationProvider.Singularize(GetPascalName(entity.Key))}");
                content.AppendLine("{");

                foreach (var property in entity.Value.Properties)
                {
                    content.AppendLine($"    [JsonProperty(\"{property.Key}\")]");
                    content.AppendLine($"    public {GetTypeName(property.Value)} {GetPascalName(property.Key)} {{ get; set; }}");
                    content.AppendLine();
                }

                // 结束
                content.AppendLine("}");
                content.AppendLine();
            }

            return content.ToString();
        }

        private static string GetTypeName(PropertyInfo property)
        {
            string name = property.Type;
            var type = Type.GetType(name, false);
            var typeNameMap = new TypeNameMap();
            return $"{(property.IsArray ? "List<" : "")}{(type != null ? (typeNameMap[type.Name] ?? type.Name) : PluralizationProvider.Singularize(GetPascalName(name)))}{(property.Nullable && type != null && type.IsSubclassOf(typeof(ValueType)) ? "?" : "")}{(property.IsArray ? ">" : "")}";
        }

        private static string GetPascalName(string name)
        {

            if (string.IsNullOrEmpty(name)) return string.Empty;

            // 匹配 特殊字符+小写字母
            var regex = new Regex("[^0-9a-zA-Z]+([0-9a-zA-Z]{1})");
            var matches = regex.Matches(name);
            foreach (Match match in matches)
            {
                name = name.Replace(match.Value, match.Result("$1").ToUpper());
            }

            // 移除非数字、字母的内容
            name = new Regex("[^0-9a-zA-Z]+").Replace(name, string.Empty);

            // 首字母转大写
            if (name.Length > 0) name = string.Concat(name.Substring(0, 1).ToUpper(), name.Substring(1));

            return name;
        }

        private static EntityInfo MapEntityInfo(string name, Dictionary<string, EntityInfo> entities, JToken token)
        {
            // 获取实体信息
            if (!entities.TryGetValue(name, out EntityInfo entity))
            {
                entity = new EntityInfo();
                entities.TryAdd(name, entity);
            }

            // 实体的属性信息
            var properties = entity.Properties;

            if (token is JObject obj)
            {
                // 内容是对象时，需要获取属性信息
                foreach (var kv in obj)
                {
                    string key = kv.Key;
                    var value = kv.Value;
                    if (!properties.TryGetValue(key, out PropertyInfo property))
                    {
                        property = new PropertyInfo();
                        property.Name = key;
                        properties.TryAdd(key, property);
                    }

                    if (value != null)
                    {
                        if (value is JValue jValue)
                        {
                            switch (jValue.Type)
                            {
                                case JTokenType.Integer:
                                    property.Type = typeof(long).FullName;
                                    break;
                                case JTokenType.Float:
                                    property.Type = typeof(decimal).FullName;
                                    break;
                                case JTokenType.Null:
                                case JTokenType.String:
                                    property.Type = typeof(string).FullName;
                                    break;
                                case JTokenType.Boolean:
                                    property.Type = typeof(bool).FullName;
                                    break;
                                case JTokenType.Date:
                                    property.Type = typeof(DateTime).FullName;
                                    break;
                                case JTokenType.Guid:
                                    property.Type = typeof(Guid).FullName;
                                    break;
                                case JTokenType.Uri:
                                    property.Type = typeof(Uri).FullName;
                                    break;
                                case JTokenType.TimeSpan:
                                    property.Type = typeof(TimeSpan).FullName;
                                    break;
                                default:
                                    throw new Exception("解析数值出现未知的数据类型！");
                            }
                        }
                        else if (value is JArray jArray)
                        {
                            property.IsArray = true;
                            property.Nullable = jArray.Any(a => a.Type == JTokenType.Null);
                            if (jArray.Count > 0 && jArray.Any(a => a.Type != JTokenType.Null))
                            {
                                var firstType = jArray.First(a => a.Type != JTokenType.Null).Type;
                                if (jArray.Any(a => a.Type != firstType && a.Type != JTokenType.Null))
                                {
                                    throw new Exception("解析数组成员类型不唯一！");
                                }

                                switch (firstType)
                                {
                                    case JTokenType.Object:
                                        property.Type = key;
                                        foreach (var item in jArray.Where(a => a.Type == JTokenType.Object))
                                        {
                                            MapEntityInfo(key, entities, item);
                                        }
                                        break;
                                    case JTokenType.Integer:
                                        property.Type = typeof(long).FullName;
                                        break;
                                    case JTokenType.Float:
                                        property.Type = typeof(decimal).FullName;
                                        break;
                                    case JTokenType.String:
                                        property.Type = typeof(string).FullName;
                                        break;
                                    case JTokenType.Boolean:
                                        property.Type = typeof(bool).FullName;
                                        break;
                                    case JTokenType.Date:
                                        property.Type = typeof(DateTime).FullName;
                                        break;
                                    case JTokenType.Guid:
                                        property.Type = typeof(Guid).FullName;
                                        break;
                                    case JTokenType.Uri:
                                        property.Type = typeof(Uri).FullName;
                                        break;
                                    case JTokenType.TimeSpan:
                                        property.Type = typeof(TimeSpan).FullName;
                                        break;
                                    default:
                                        throw new Exception("解析数组出现未知的数据类型！");
                                }
                            }
                        }
                        else
                        {
                            property.Type = key;
                            MapEntityInfo(key, entities, value);
                        }
                    }
                    else
                    {
                        property.Nullable = true;
                    }
                }
            }

            return entity;
        }
    }

    class EntityInfo
    {
        public string Name { get; set; }

        public Dictionary<string, PropertyInfo> Properties { get; set; } = new Dictionary<string, PropertyInfo>();
    }

    class PropertyInfo
    {
        public string Name { get; set; }

        public string Type { get; set; } = typeof(string).FullName;

        public bool IsArray { get; set; }

        public bool Nullable { get; set; }
    }

    class TypeNameMap
    {
        private static readonly Dictionary<string, string> typeNameMap = new()
        {
            [typeof(bool).Name] = "bool",
            [typeof(byte).Name] = "byte",
            [typeof(sbyte).Name] = "sbyte",
            [typeof(short).Name] = "short",
            [typeof(ushort).Name] = "ushort",
            [typeof(int).Name] = "int",
            [typeof(uint).Name] = "uint",
            [typeof(long).Name] = "long",
            [typeof(ulong).Name] = "ulong",
            [typeof(float).Name] = "float",
            [typeof(double).Name] = "double",
            [typeof(decimal).Name] = "decimal",
            [typeof(string).Name] = "string",
        };

        internal string this[string name]
        {
            get
            {
                typeNameMap.TryGetValue(name, out string value);
                return value;
            }
        }
    }
}
```

## 测试

测试 `json` 内容：

```json
{
    "ProductInfoList": [
        {
            "Sku": "58396277",
            "CategoryFirstCode": "4841",
            "CategoryFirstName": "家居和园艺",
            "CategoryFirstNameEN": "Home & Garden",
            "CategorySecondCode": "48410020",
            "CategorySecondName": "家装五金",
            "CategorySecondNameEN": "Home Improvement",
            "CategoryThirdCode": "484100206798",
            "CategoryThirdName": "家庭安全",
            "CategoryThirdNameEN": "Home Security",
            "CnName": "英文字典迷你保险柜存钱盒大号钥匙款 蓝色 27*20*6.5CM",
            "EnName": "Cute Simulation English Dictionary Style Mini Safety Storage Box Blue L",
            "SpecLength": 27.50,
            "SpecWidth": 21.00,
            "SpecHeight": 7.50,
            "SpecWeight": 1120.00,
            "Published": false,
            "IsClear": false,
            "FreightAttrCode": "X0018",
            "FreightAttrName": "普货",
            "PlatformCommodityCode": "C13145463",
            "CommodityWarehouseMode": 0,
            "GoodsImageList": [
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550097585_f535c1c5-5bea-4a83-8c43-00f325527a00.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550102897_18e38ad6-5f9e-46b6-ad67-f4847aa0c705.JPG",
                    "Sort": 0,
                    "IsMainImage": true,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/dcda886b-72c1-41d8-8f03-f712c1ab233f.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/ada5247f-9d95-46d7-8223-cc3ae733a89b.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550103366_12dbbb9d-0bc0-4154-a206-0f37a4011f23.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550109616_b993bbff-68e4-4b4d-8bba-fadcc07a1194.JPG",
                    "Sort": 1,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/6f7fe5e7-6935-4062-8828-52bdc74feb01.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/bccf0963-319d-469e-b12b-541888df755a.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550110241_087e4b65-314e-4035-9eb3-5970d6192211.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550116960_a2932fc9-5fc2-4d6f-9552-4421ea6e79ff.JPG",
                    "Sort": 2,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/2ca3f29f-775f-4b24-86b2-88bd778f11b1.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/ce3fd469-b08e-4364-a185-d13245b5ee57.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550117429_636348d8-aca3-4628-89c8-05aaf63be259.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550121022_2dcdc4c0-bcc3-412b-b242-450839220052.JPG",
                    "Sort": 3,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/199bb9ce-9a48-4493-8d1d-a591e7c56cf7.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/54ed6d08-8b9f-4b9c-89ae-5f6c19e25c78.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550121491_8198fac2-13da-48a6-8286-afde185c7fba.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550127429_7153e854-d5f4-4ab2-a868-e1b6b22a480a.JPG",
                    "Sort": 4,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/af946cc4-2e2a-4bfb-81e3-8ee4e0334d4f.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/c86dd714-0dab-477b-ab20-7e7246ee6003.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550128210_85dccec4-ebb7-48dd-b673-ffa9ed0ba5d0.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550131647_eadb06be-0400-46fb-b7c6-f104120c61e0.JPG",
                    "Sort": 5,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/b2f9779f-c143-45a6-ac7c-a57170f1ebc6.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/c79e35bb-7ca5-4527-b60f-523a2990b863.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550132116_da004c50-bb32-4a7f-bcc6-9e431d21caf8.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550138678_169e8bc0-7f67-4c8e-88e3-6dfde45890e1.JPG",
                    "Sort": 6,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/521e7c4e-68fb-4de7-ba6b-4b9029c7c160.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/18172593-beda-478c-b455-295672e1b3a4.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550139147_a80119d9-24f8-4da4-abc1-fbf4590090b8.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550142428_545ff06e-2db3-4988-b2b9-955d577b7449.JPG",
                    "Sort": 7,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/ec9b4d9d-1fbf-4062-bad2-8f5c304affba.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/ea0690d1-9c53-4cdd-a912-aea15509689e.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550142897_eae670d9-9615-4e41-823a-7f1170c7283e.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550149616_7b451919-ee54-4ea2-a6db-32b7bb179317.JPG",
                    "Sort": 8,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/045580ad-87fc-4be8-ab4a-c60cad68ee64.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/82412afb-0f76-47e0-bf60-82e4b85732ec.JPG"
                        }
                    ]
                },
                {
                    "ImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550150397_226c7997-6b93-4f82-a7ae-ceeb40076dce.JPG",
                    "ThumbnailUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/202012081550157741_ea1c450b-5984-45d0-b126-43096e220325.JPG",
                    "Sort": 9,
                    "IsMainImage": false,
                    "GooodsThumbnailList": [
                        {
                            "StandardLength": 350,
                            "StandardWidth": 350,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/eaac56d0-1a00-4d2c-9950-0d6249031bd4.JPG"
                        },
                        {
                            "StandardLength": 600,
                            "StandardWidth": 600,
                            "StandardImageUrl": "https://img.goten.com/Resources/GoodsImages//2020/202012/dd6b6757-0dc5-41f3-995b-40dcb4775fbe.JPG"
                        }
                    ]
                }
            ],
            "GoodsAttachmentList": [],
            "GoodsDescriptionList": [
                {
                    "Title": "Cute Simulation English Dictionary Style Mini Safety Storage Box Blue L",
                    "GoodsDescriptionKeywordList": [
                        {
                            "KeyWord": "Safety Box"
                        }
                    ],
                    "GoodsDescriptionLabelList": [
                        {
                            "LabelName": "英文"
                        }
                    ],
                    "GoodsDescriptionParagraphList": [
                        {
                            "ParagraphName": "首段描述",
                            "SortNo": 1,
                            "GoodsDescription": "<p><STRONG>Introductions:</STRONG><BR>Fantastic dictionary look makes this storage box rather distinctive from ordinary ones. Due to this feature, it can well protect your secrets without being found easily. Made with high-class material as well exquisite craftsmanship, this storage box is wear-resistant for long-term use. With quite spacious space, it allows you to put important things inside. Close and then place it on the book shelf, it seems to be a real English dictionary. Such a mini gadget can also be a good decoration for your room. Well, wanna give it a try? Just click to buy our Cute Simulation English Dictionary Style Mini Safety Storage Box!<BR></p>"
                        },
                        {
                            "ParagraphName": "特征",
                            "SortNo": 2,
                            "GoodsDescription": "<p><STRONG>Features:</STRONG><BR>1. Specially designed into English dictionary style, quite mini and cute<BR>2. Great for deceiving the public so as to keep your belongings safe and unnoticed<BR>3. Made with first-rate material and exquisite workmanship, of great durability and reliability<BR>4. Compact size, lightweight and room-saving<BR>5. Hollow dictionary, can be used for storing important gadgets<BR>6. Also a nice decoration for room fitment&nbsp;&nbsp;<BR></p>"
                        },
                        {
                            "ParagraphName": "规格",
                            "SortNo": 3,
                            "GoodsDescription": "<p><P><STRONG>Specifications:</STRONG><BR>1. Material: Iron &amp; ABS &amp; Specialty Paper/Imitation Cloth<BR>2. Color: Blue<BR>3. Dimensions: (10.62 x 7.87 x 2.56)\" / (27 x 20 x 6.5)cm (L x W x H)<BR>4. Weight: 35.27oz / 1000g</P>\n<P>&nbsp;</P></p>"
                        },
                        {
                            "ParagraphName": "包装内含",
                            "SortNo": 4,
                            "GoodsDescription": "<p><STRONG>Package Includes:</STRONG><BR>1 x English Dictionary Mini Safety Box</p>"
                        },
                        {
                            "ParagraphName": "通用性",
                            "SortNo": 5,
                            "GoodsDescription": "<p></p>"
                        },
                        {
                            "ParagraphName": "附加信息",
                            "SortNo": 6,
                            "GoodsDescription": "<p></p>"
                        }
                    ]
                }
            ],
            "CreateTime": "2018-03-12T12:31:20",
            "UpdateTime": "2021-03-01T03:20:20",
            "TortInfo": {
                "TortReasons": null,
                "TortTypes": null,
                "TortStatus": 2
            },
            "IsProductAuth": true,
            "ProductSiteModelList": [
                {
                    "SiteHosts": "www.gotenchina.com",
                    "WarehouseCodeList": [
                        "SZ0001"
                    ]
                }
            ],
            "BrandName": null
        }
    ],
    "PageIndex": 1,
    "TotalCount": 6656,
    "PageTotal": 666,
    "PageSize": 10
}
```

生成的实体类内容：

```csharp
public class BaseEntity
{
    [JsonProperty("ProductInfoList")]
    public List<ProductInfoList> ProductInfoList { get; set; }

    [JsonProperty("PageIndex")]
    public long PageIndex { get; set; }

    [JsonProperty("TotalCount")]
    public long TotalCount { get; set; }

    [JsonProperty("PageTotal")]
    public long PageTotal { get; set; }

    [JsonProperty("PageSize")]
    public long PageSize { get; set; }

}

public class ProductInfoList
{
    [JsonProperty("Sku")]
    public string Sku { get; set; }

    [JsonProperty("CategoryFirstCode")]
    public string CategoryFirstCode { get; set; }

    [JsonProperty("CategoryFirstName")]
    public string CategoryFirstName { get; set; }

    [JsonProperty("CategoryFirstNameEN")]
    public string CategoryFirstNameEN { get; set; }

    [JsonProperty("CategorySecondCode")]
    public string CategorySecondCode { get; set; }

    [JsonProperty("CategorySecondName")]
    public string CategorySecondName { get; set; }

    [JsonProperty("CategorySecondNameEN")]
    public string CategorySecondNameEN { get; set; }

    [JsonProperty("CategoryThirdCode")]
    public string CategoryThirdCode { get; set; }

    [JsonProperty("CategoryThirdName")]
    public string CategoryThirdName { get; set; }

    [JsonProperty("CategoryThirdNameEN")]
    public string CategoryThirdNameEN { get; set; }

    [JsonProperty("CnName")]
    public string CnName { get; set; }

    [JsonProperty("EnName")]
    public string EnName { get; set; }

    [JsonProperty("SpecLength")]
    public decimal SpecLength { get; set; }

    [JsonProperty("SpecWidth")]
    public decimal SpecWidth { get; set; }

    [JsonProperty("SpecHeight")]
    public decimal SpecHeight { get; set; }

    [JsonProperty("SpecWeight")]
    public decimal SpecWeight { get; set; }

    [JsonProperty("Published")]
    public bool Published { get; set; }

    [JsonProperty("IsClear")]
    public bool IsClear { get; set; }

    [JsonProperty("FreightAttrCode")]
    public string FreightAttrCode { get; set; }

    [JsonProperty("FreightAttrName")]
    public string FreightAttrName { get; set; }

    [JsonProperty("PlatformCommodityCode")]
    public string PlatformCommodityCode { get; set; }

    [JsonProperty("CommodityWarehouseMode")]
    public long CommodityWarehouseMode { get; set; }

    [JsonProperty("GoodsImageList")]
    public List<GoodsImageList> GoodsImageList { get; set; }

    [JsonProperty("GoodsAttachmentList")]
    public List<string> GoodsAttachmentList { get; set; }

    [JsonProperty("GoodsDescriptionList")]
    public List<GoodsDescriptionList> GoodsDescriptionList { get; set; }

    [JsonProperty("CreateTime")]
    public DateTime CreateTime { get; set; }

    [JsonProperty("UpdateTime")]
    public DateTime UpdateTime { get; set; }

    [JsonProperty("TortInfo")]
    public TortInfo TortInfo { get; set; }

    [JsonProperty("IsProductAuth")]
    public bool IsProductAuth { get; set; }

    [JsonProperty("ProductSiteModelList")]
    public List<ProductSiteModelList> ProductSiteModelList { get; set; }

    [JsonProperty("BrandName")]
    public string BrandName { get; set; }

}

public class GoodsImageList
{
    [JsonProperty("ImageUrl")]
    public string ImageUrl { get; set; }

    [JsonProperty("ThumbnailUrl")]
    public string ThumbnailUrl { get; set; }

    [JsonProperty("Sort")]
    public long Sort { get; set; }

    [JsonProperty("IsMainImage")]
    public bool IsMainImage { get; set; }

    [JsonProperty("GooodsThumbnailList")]
    public List<GooodsThumbnailList> GooodsThumbnailList { get; set; }

}

public class GooodsThumbnailList
{
    [JsonProperty("StandardLength")]
    public long StandardLength { get; set; }

    [JsonProperty("StandardWidth")]
    public long StandardWidth { get; set; }

    [JsonProperty("StandardImageUrl")]
    public string StandardImageUrl { get; set; }

}

public class GoodsDescriptionList
{
    [JsonProperty("Title")]
    public string Title { get; set; }

    [JsonProperty("GoodsDescriptionKeywordList")]
    public List<GoodsDescriptionKeywordList> GoodsDescriptionKeywordList { get; set; }

    [JsonProperty("GoodsDescriptionLabelList")]
    public List<GoodsDescriptionLabelList> GoodsDescriptionLabelList { get; set; }

    [JsonProperty("GoodsDescriptionParagraphList")]
    public List<GoodsDescriptionParagraphList> GoodsDescriptionParagraphList { get; set; }

}

public class GoodsDescriptionKeywordList
{
    [JsonProperty("KeyWord")]
    public string KeyWord { get; set; }

}

public class GoodsDescriptionLabelList
{
    [JsonProperty("LabelName")]
    public string LabelName { get; set; }

}

public class GoodsDescriptionParagraphList
{
    [JsonProperty("ParagraphName")]
    public string ParagraphName { get; set; }

    [JsonProperty("SortNo")]
    public long SortNo { get; set; }

    [JsonProperty("GoodsDescription")]
    public string GoodsDescription { get; set; }

}

public class TortInfo
{
    [JsonProperty("TortReasons")]
    public string TortReasons { get; set; }

    [JsonProperty("TortTypes")]
    public string TortTypes { get; set; }

    [JsonProperty("TortStatus")]
    public long TortStatus { get; set; }

}

public class ProductSiteModelList
{
    [JsonProperty("SiteHosts")]
    public string SiteHosts { get; set; }

    [JsonProperty("WarehouseCodeList")]
    public List<string> WarehouseCodeList { get; set; }

}
```
