---
title: "汉字帮助类——获取拼音与五笔简码"
date: "2021/02/23 16:29:00"
updated: "2021/02/23 16:29:00"
permalink: "chinese-helper/"
tags:
 - 汉字
 - 五笔
 - 拼音
categories:
 - [开发, C#]
---

## 前言

系统各类的数据字典、业务数据检索，普遍是通过下拉框实现，部分会提供模糊检索，但是很少会提供通过五笔或拼音首码来检索的方式。

在过去参与的系统设计中，发现对于计算机系统不熟悉、键盘操作不熟练的用户来说，提供五笔或拼音首码检索的方式，可以大幅度提高表单录入的速度。

本文主要介绍如果通过汉字信息，获取拼音、五笔首码的信息。

> 字典数据的检索建议提供 ID、名称、代码、拼音首码、五笔首码 等方式，提升用户在表单录入时的录入速度。

## 准备

数据字典中的 ID、名称、代码 都是原生信息，很方便获取，但是 拼音首码 与 五笔首码 需要维护字典库存储用于生成数据字典。

目前已经通过 `百度汉语` 以及 `汉典网` 提取到基本汉字的信息，包括 拼音、五笔 等信息。

链接: https://pan.baidu.com/s/17OOspKdKk5MJi3YT3Nhsjw  密码: tgd1

## 帮助类

已经预先导出了一份包含 汉字、拼音首码、五笔首码 信息的文件。

链接: https://pan.baidu.com/s/14Z5waW6OIGo2_PMFf7cvow  密码: g37t

五笔与拼音首码获取帮助类，可以参考：

```csharp
/// <summary>
/// 拼音五笔帮助类
/// </summary>
public static class PinyinWubiHelper
{
    private static Dictionary<char, (char[] pinyin, char[] wubi)> _dicHanzi = new Dictionary<char, (char[] pinyin, char[] wubi)>();
    static PinyinWubiHelper()
    {
        string text = Resource.chinese;
        // 记录没有五笔或拼音的字符，存在需要抛出异常
        List<char> listWubi = new List<char>();
        List<char> listPinyin = new List<char>();
        foreach (string hanzi in text.Split('\n', StringSplitOptions.RemoveEmptyEntries))
        {
            string[] info = hanzi.Split(',');
            _dicHanzi.Add(info[0][0]
                , (pinyin: info[2].Split('|', StringSplitOptions.RemoveEmptyEntries).Select(py => py[0]).ToArray()
                , wubi: info[1].Split('|', StringSplitOptions.RemoveEmptyEntries).Select(wb => wb[0]).ToArray()));
            if (_dicHanzi[info[0][0]].wubi.Length == 0) listWubi.Add(info[0][0]);
            if (_dicHanzi[info[0][0]].pinyin.Length == 0) listPinyin.Add(info[0][0]);
        }
        if (listWubi.Count > 0 || listPinyin.Count > 0)
        {
            throw new Exception($"{(listWubi.Count > 0 ? $"存在无五笔首码的字符：{string.Concat(listWubi)}" : "")}\r\n{(listPinyin.Count > 0 ? $"存在无拼音首码的字符：{string.Concat(listPinyin)}" : "")}");
        }
    }
 
    /// <summary>
    /// 获取一段中文的五笔首码
    /// </summary>
    /// <param name="text">中文内容</param>
    /// <param name="length">期望返回五笔首码的最大长度，默认 20</param>
    /// <returns>处理后的五笔首码</returns>
    public static IEnumerable<string> GetWubiCode(string text, int length = 20)
    {
        text = HandlingText(text, length);
        if (!string.IsNullOrEmpty(text))
        {
            // 默认第一位首码时的内容并返回
            var def = text
                .ToCharArray()
                .Select(ch => _dicHanzi.ContainsKey(ch) ? _dicHanzi[ch].wubi[0] : ch);
 
            yield return new string(def.ToArray());
 
            // 记录迭代到指定位置时还未返回的所有可能
            List<List<char>> list = new List<List<char>> { new List<char>() };
 
            // 循环字符串的所有字符
            for (int i = 0; i < text.Length; i++)
            {
                if (_dicHanzi.ContainsKey(text[i]))
                {
                    // 当前字符存在首码
                    char[] wubi = _dicHanzi[text[i]].wubi;
                    if (wubi.Length > 1)
                    {
                        // 1. 存在多个首码，需要将每种可能的结果返回，并且更新所有组合的可能性
                        // 获取当前位置之后的默认字符，用于返回
                        IEnumerable<char> last = def.Take(i + 1);
 
                        // 更新截止到当前字符之前的所有可能
                        List<List<char>> update = new List<List<char>>();
 
                        // 循环截止到当前字符之前的所有可能
                        for (int j = 0; j < list.Count; j++)
                        {
                            // 循环当前字符的所有首码
                            for (int k = 0; k < wubi.Length; k++)
                            {
                                var current = new List<char>(list[j]) { wubi[k] };
 
                                // 将截至到当前位置的所有默认可能返回
                                if (k > 0)
                                {
                                    var result = new List<char>(current);
                                    result.AddRange(last);
                                    yield return new string(result.ToArray());
                                }
 
                                update.Add(current);
                            }
                        }
 
                        // 可能性增加，需要更新
                        list = update;
                    }
                    else
                    {
                        // 2. 只存在一个首码，将当前字符的首码添加到内容中
                        list.ForEach(l => l.Add(_dicHanzi[text[i]].wubi[0]));
                    }
                }
                else
                {
                    // 当前字符不存在五笔首码，则将当前字符添加到内容中
                    list.ForEach(l => l.Add(text[i]));
                }
            }
        }
    }
 
    /// <summary>
    /// 获取一段中文的拼音首码
    /// </summary>
    /// <param name="text">中文内容</param>
    /// <param name="length">期望返回拼音首码的最大长度，默认 20</param>
    /// <returns>处理后的拼音首码</returns>
    public static IEnumerable<string> GetPinyinCode(string text, int length = 20)
    {
        text = HandlingText(text, length);
        if (!string.IsNullOrEmpty(text))
        {
            // 默认第一位首码时的内容并返回
            var def = text
                .ToCharArray()
                .Select(ch => _dicHanzi.ContainsKey(ch) ? _dicHanzi[ch].pinyin[0] : ch);
 
            yield return new string(def.ToArray());
 
            // 记录迭代到指定位置时还未返回的所有可能
            List<List<char>> list = new List<List<char>> { new List<char>() };
 
            // 循环字符串的所有字符
            for (int i = 0; i < text.Length; i++)
            {
                if (_dicHanzi.ContainsKey(text[i]))
                {
                    // 当前字符存在首码
                    char[] pinyin = _dicHanzi[text[i]].pinyin;
                    if (pinyin.Length > 1)
                    {
                        // 1. 存在多个首码，需要将每种可能的结果返回，并且更新所有组合的可能性
                        // 获取当前位置之后的默认字符，用于返回
                        IEnumerable<char> last = def.Skip(i + 1);
 
                        // 更新截止到当前字符之前的所有可能
                        List<List<char>> update = new List<List<char>>();
 
                        // 循环截止到当前字符之前的所有可能
                        for (int j = 0; j < list.Count; j++)
                        {
                            // 循环当前字符的所有首码
                            for (int k = 0; k < pinyin.Length; k++)
                            {
                                var current = new List<char>(list[j]) { pinyin[k] };
 
                                // 将截至到当前位置的所有默认可能返回
                                if (k > 0)
                                {
                                    var result = new List<char>(current);
                                    result.AddRange(last);
                                    yield return new string(result.ToArray());
                                }
 
                                update.Add(current);
                            }
                        }
 
                        // 可能性增加，需要更新
                        list = update;
                    }
                    else
                    {
                        // 2. 只存在一个首码，将当前字符的首码添加到内容中
                        list.ForEach(l => l.Add(_dicHanzi[text[i]].pinyin[0]));
                    }
                }
                else
                {
                    // 当前字符不存在拼音首码，则将当前字符添加到内容中
                    list.ForEach(l => l.Add(text[i]));
                }
            }
        }
    }
 
    // 特殊字符字典
    private static Dictionary<char, char> _dicChar = new Dictionary<char, char>
        {
            ['Ａ'] = 'A',
            ['Ｂ'] = 'B',
            ['Ｃ'] = 'C',
            ['Ｄ'] = 'D',
            ['Ｅ'] = 'E',
            ['Ｆ'] = 'F',
            ['Ｇ'] = 'G',
            ['Ｈ'] = 'H',
            ['Ｉ'] = 'I',
            ['Ｊ'] = 'J',
            ['Ｋ'] = 'K',
            ['Ｌ'] = 'L',
            ['Ｍ'] = 'M',
            ['Ｎ'] = 'N',
            ['Ｏ'] = 'O',
            ['Ｐ'] = 'P',
            ['Ｑ'] = 'Q',
            ['Ｒ'] = 'R',
            ['Ｓ'] = 'S',
            ['Ｔ'] = 'T',
            ['Ｕ'] = 'U',
            ['Ｖ'] = 'V',
            ['Ｗ'] = 'W',
            ['Ｘ'] = 'X',
            ['Ｙ'] = 'Y',
            ['Ｚ'] = 'Z',
            ['ａ'] = 'a',
            ['ｂ'] = 'b',
            ['ｃ'] = 'c',
            ['ｄ'] = 'd',
            ['ｅ'] = 'e',
            ['ｆ'] = 'f',
            ['ｇ'] = 'g',
            ['ｈ'] = 'h',
            ['ｉ'] = 'i',
            ['ｊ'] = 'j',
            ['ｋ'] = 'k',
            ['ｌ'] = 'l',
            ['ｍ'] = 'm',
            ['ｎ'] = 'n',
            ['ｏ'] = 'o',
            ['ｐ'] = 'p',
            ['ｑ'] = 'q',
            ['ｒ'] = 'r',
            ['ｓ'] = 's',
            ['ｔ'] = 't',
            ['ｕ'] = 'u',
            ['ｖ'] = 'v',
            ['ｗ'] = 'w',
            ['ｘ'] = 'x',
            ['ｙ'] = 'y',
            ['ｚ'] = 'z',
            ['０'] = '0',
            ['１'] = '1',
            ['２'] = '2',
            ['３'] = '3',
            ['４'] = '4',
            ['５'] = '5',
            ['６'] = '6',
            ['７'] = '7',
            ['８'] = '8',
            ['９'] = '9',
        };
    // 用于替换内容，保留数字、小写字母、大写字母、基本汉字、〇、全角数字、全角小写字母、全角大写字母
    private static readonly Regex _regex = new Regex(@"[^0-9A-Za-z\u4E00-\u9FA5\u3007０-９ａ-ｚＡ-Ｚ]");
    private static string HandlingText(string text, int length = 20)
    {
        if (string.IsNullOrWhiteSpace(text)) return "";
 
        // 将非数字字母汉字等移除
        text = _regex.Replace(text, "");
 
        List<char> content = new List<char>();
        // 正则会移除高位字符，这里进行迭代已经不会出现高位字符被拆成两个特殊字符的情况
        foreach (char ch in text)
        {
            // 将特殊字符替换，例如全角字符替换为半角
            if (_dicChar.ContainsKey(ch))
            {
                content.Add(_dicChar[ch]);
            }
            else
            {
                content.Add(ch);
            }
 
            if (content.Count >= length) break;
        }
 
        return new string(content.ToArray()).ToUpper();
    }
}
```

测试代码如下：

```csharp
string text = "南京市长江大桥参观南京市长江大桥)!@$#ＡａＢｂＣｃ１２３AaBbCc123";
Console.WriteLine($"[{text}]");
Console.WriteLine("拼音：");
foreach (var py in PinyinWubiHelper.GetPinyinCode(text, 50))
{
    Console.WriteLine(py);
}
Console.WriteLine("五笔：");
foreach (var wb in PinyinWubiHelper.GetWubiCode(text, 50))
{
    Console.WriteLine(wb);
}
 
// 控制台输出内容：
// [南京市长江大桥参观南京市长江大桥)!@$#ＡａＢｂＣｃ１２３AaBbCc123]
// 拼音：
// NJSCJDQCGNJSCJDQAABBCC123AABBCC123
// NJSZJDQCGNJSCJDQAABBCC123AABBCC123
// NJSCJDQSGNJSCJDQAABBCC123AABBCC123
// NJSZJDQSGNJSCJDQAABBCC123AABBCC123
// NJSCJDQCGNJSZJDQAABBCC123AABBCC123
// NJSCJDQSGNJSZJDQAABBCC123AABBCC123
// NJSZJDQCGNJSZJDQAABBCC123AABBCC123
// NJSZJDQSGNJSZJDQAABBCC123AABBCC123
// 五笔：
// FYYTIDSCCFYYTIDSAABBCC123AABBCC123
```
