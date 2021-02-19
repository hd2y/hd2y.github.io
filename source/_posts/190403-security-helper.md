---
title: "SecurityHelper 加密解密帮助类"
date: "2019/04/03 21:11:00"
updated: "2019/07/10 17:09:51"
permalink: "security-helper/"
tags:
 - 加密
categories:
 - [开发, C#]
---

## `SecurityHelper` 帮助类

该帮助类主要实现了 `MD5` 算法的 `加密`，`DES`、`3DES`、`RSA` 算法的 `加密` 与 `解密`，内部简单实现了一个混淆算法。

## `MD5` 加密算法

`MD5` 消息摘要算法：全称是 `Message-Digest Algorithm`，一种被广泛使用的密码散列函数，可以产生出一个 `128位`（`16字节`）的 `散列值`（`hash value`），用于确保信息传输完整一致。。

MD5主要用途：
+ 对一段信息生成信息摘要，该摘要对该信息具有唯一性，可以作为数字签名
+ 用于验证文件的有效性(是否有丢失或损坏的数据)
+ 对用户密码的加密
+ 在哈希函数中计算散列值

从上边的主要用途中我们看到，由于算法的某些不可逆特征，在加密应用上有较好的安全性。通过使用 `MD5` 加密算法，我们输入一个任意长度的字节串，都会生成一个`128位` 的整数。所以根据这一点 `MD5` 被广泛的用作密码加密。

`SecurityHelper` 中调用的方法：
```csharp
string str = SecurityHelper.EncryptByMD5("测试");
```

## `DES` 与 `3DES` 加密算法

`DES` 全称为 `Data Encryption Standard`，即数据加密标准，是一种使用密钥加密的块算法。`DES` 算法的入口参数有三个：`Key`、`Data`、`Mode`。其中 `Key` 为 `7个字节` 共 `56位`，是 `DES算法` 的工作密钥；`Data` 为 `8个字节` `64位`，是要被加密或被解密的数据；`Mode` 为 `DES` 的工作方式，有两种：加密或解密。其速度较快，适用于加密大量数据的场合。

`SecurityHelper` 中调用的方法：

```csharp
string encryptStr = SecurityHelper.EncryptByDES("测试");
string str = SecurityHelper.DecryptByDES(encryptStr);
```

`3DES`（即 `Triple DES`）是 `DES` 向 `AES` 过渡的加密算法，它使用 `3条` `56位` 的密钥对数据进行 `三次加密`。是 `DES` 的一个更安全的变形。它以 `DES` 为基本模块，通过组合分组方法设计出分组加密算法。比起最初的 `DES`，`3DES` 更为安全。

`SecurityHelper` 中调用的方法：

```csharp
string encryptStr = SecurityHelper.EncryptByTDES("测试");
string str = SecurityHelper.DecryptByTDES(encryptStr);
```

## `RSA` 加密算法

`RSA` 加密算法是一种 `非对称` 加密算法。在公开密钥加密和电子商业中 `RSA` 被广泛使用。

`RSA` 算法是第一个能同时用于加密和数字签名的算法，也易于理解和操作。`RSA` 是被研究得最广泛的公钥算法，从提出到现今的三十多年里，经历了各种攻击的考验，逐渐为人们接受，截止 `2017年` 被普遍认为是最优秀的公钥方案之一。

`SecurityHelper` 中调用的方法：
```csharp
// 生成公钥与私钥
var kv = SecurityHelper.GeneralRSAKeys();

string encryptStr = SecurityHelper.EncryptByRSAFromXmlString("测试", kv.Key);
string str = SecurityHelper.DecryptByRSAFromXmlString(encryptStr, kv.Value);
```

## `混淆` 与 `反混淆`

当需求对数据的传输有不高的安全加密，且加密的时间复杂度越低越好，这时我们可以使用简单的混淆算法，`SecurityHelper` 实现了一个简单的混淆算法供开发人员调用。

```csharp
string encryptStr = SecurityHelper.MixUp("测试");
string str = SecurityHelper.ClearUp(encryptStr);
```

## 源码

``` csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Security.Cryptography;
using System.Text;

namespace System
{
    /// <summary>
    /// 加密安全帮助类
    /// </summary>
    public class SecurityHelper
    {
        /// <summary>
        /// MD5加密字符串 (不可逆)
        /// </summary>
        /// <param name="text">需要加密的内容</param>
        /// <param name="key">加密密钥，防止简单密码被破解</param>
        /// <param name="binaryStyle">加密后结果内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>加密结果</returns>
        public static string EncryptByMD5(string text, string key = default(string), bool binaryStyle = true)
        {
            MD5 md5 = MD5.Create();
            byte[] bytes = md5.ComputeHash(Encoding.UTF8.GetBytes(text + key));
            return binaryStyle ? BitConverter.ToString(bytes) : Convert.ToBase64String(bytes);
        }

        /// <summary>
        /// DES加密字符串 (可逆)
        /// </summary>
        /// <param name="text">待加密的字符串</param>
        /// <param name="key">DES加密的私钥，必须是字节长度8位的字符串，否则会补0。</param>
        /// <param name="iv">DES加密偏移量，必须是>=8位长的字符串，否则会补0。</param>
        /// <param name="binaryStyle">加密后结果内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>加密后的字符串</returns>
        public static string EncryptByDES(string text, string key = "", string iv = "", bool binaryStyle = true)
        {
            List<byte> bKey = Encoding.UTF8.GetBytes(key).ToList();
            for (int i = bKey.Count; i < 8; i++)
            {
                bKey.Add((byte)i);
            }
            bKey = bKey.GetRange(0, 8);
            List<byte> bIV = Encoding.UTF8.GetBytes(iv).ToList();
            for (int i = bIV.Count; i < 8; i++)
            {
                bIV.Add((byte)i);
            }
            using (DESCryptoServiceProvider des = new DESCryptoServiceProvider())
            using (MemoryStream ms = new MemoryStream())
            {
                byte[] inData = Encoding.UTF8.GetBytes(text);
                using (CryptoStream cs = new CryptoStream(ms, des.CreateEncryptor(bKey.ToArray(), bIV.ToArray()), CryptoStreamMode.Write))
                {
                    cs.Write(inData, 0, inData.Length);
                    cs.FlushFinalBlock();
                }
                return binaryStyle ? BitConverter.ToString(ms.ToArray()) : Convert.ToBase64String(ms.ToArray());
            }
        }

        /// <summary>
        /// DES解密字符串
        /// </summary>
        /// <param name="text">待解密的字符串</param>
        /// <param name="key">DES加密的私钥，必须是字节长度8位的字符串，否则会补0。</param>
        /// <param name="iv">DES加密偏移量，必须是>=8位长的字符串，否则会补0。</param>
        /// <param name="binaryStyle">解密内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>解密后的字符串</returns>
        public static string DecryptByDES(string text, string key = "", string iv = "", bool binaryStyle = true)
        {
            List<byte> bKey = Encoding.UTF8.GetBytes(key).ToList();
            for (int i = bKey.Count; i < 8; i++)
            {
                bKey.Add((byte)i);
            }
            bKey = bKey.GetRange(0, 8);
            List<byte> bIV = Encoding.UTF8.GetBytes(iv).ToList();
            for (int i = bIV.Count; i < 8; i++)
            {
                bIV.Add((byte)i);
            }
            using (DESCryptoServiceProvider des = new DESCryptoServiceProvider())
            using (MemoryStream ms = new MemoryStream())
            {
                byte[] inData = binaryStyle ? text.GetBytesFromBitConverter() : Convert.FromBase64String(text);
                using (CryptoStream cs = new CryptoStream(ms, des.CreateDecryptor(bKey.ToArray(), bIV.ToArray()), CryptoStreamMode.Write))
                {
                    cs.Write(inData, 0, inData.Length);
                    cs.FlushFinalBlock();
                }
                return Encoding.UTF8.GetString(ms.ToArray());
            }
        }

        /// <summary>
        /// TDES加密字符串 (可逆)
        /// </summary>
        /// <param name="text">待加密的字符串</param>
        /// <param name="key">TDES加密的私钥，必须是字节长度24位的字符串，否则会自动补位。</param>
        /// <param name="iv">TDES加密偏移量，必须是字节长度>=8位的字符串，否则会自动补位。</param>
        /// <param name="binaryStyle">加密后结果内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>加密后的字符串</returns>
        public static string EncryptByTDES(string text, string key = "", string iv = "", bool binaryStyle = true)
        {
            List<byte> bKey = Encoding.UTF8.GetBytes(key).ToList();
            for (int i = bKey.Count; i < 24; i++)
            {
                bKey.Add((byte)i);
            }
            bKey = bKey.GetRange(0, 24);
            List<byte> bIV = Encoding.UTF8.GetBytes(iv).ToList();
            for (int i = bIV.Count; i < 8; i++)
            {
                bIV.Add((byte)i);
            }
            using (TripleDESCryptoServiceProvider des = new TripleDESCryptoServiceProvider())
            using (MemoryStream ms = new MemoryStream())
            {
                byte[] inData = Encoding.UTF8.GetBytes(text);
                using (CryptoStream cs = new CryptoStream(ms, des.CreateEncryptor(bKey.ToArray(), bIV.ToArray()), CryptoStreamMode.Write))
                {
                    cs.Write(inData, 0, inData.Length);
                    cs.FlushFinalBlock();
                }
                return binaryStyle ? BitConverter.ToString(ms.ToArray()) : Convert.ToBase64String(ms.ToArray());
            }
        }

        /// <summary>
        /// TDES解密字符串
        /// </summary>
        /// <param name="text">待解密的字符串</param>
        /// <param name="key">TDES加密的私钥，必须是字节长度24位的字符串，否则会自动补位。</param>
        /// <param name="iv">TDES加密偏移量，必须是字节长度>=8位的字符串，否则会自动补位。</param>
        /// <param name="binaryStyle">解密内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>解密后的字符串</returns>
        public static string DecryptByTDES(string text, string key = "", string iv = "", bool binaryStyle = true)
        {
            List<byte> bKey = Encoding.UTF8.GetBytes(key).ToList();
            for (int i = bKey.Count; i < 24; i++)
            {
                bKey.Add((byte)i);
            }
            bKey = bKey.GetRange(0, 24);
            List<byte> bIV = Encoding.UTF8.GetBytes(iv).ToList();
            for (int i = bIV.Count; i < 8; i++)
            {
                bIV.Add((byte)i);
            }
            using (TripleDESCryptoServiceProvider des = new TripleDESCryptoServiceProvider())
            using (MemoryStream ms = new MemoryStream())
            {
                byte[] inData = binaryStyle ? text.GetBytesFromBitConverter() : Convert.FromBase64String(text);
                using (CryptoStream cs = new CryptoStream(ms, des.CreateDecryptor(bKey.ToArray(), bIV.ToArray()), CryptoStreamMode.Write))
                {
                    cs.Write(inData, 0, inData.Length);
                    cs.FlushFinalBlock();
                }
                return Encoding.UTF8.GetString(ms.ToArray());
            }
        }

        /// <summary>
        /// 获取RSA加密公钥和私钥
        /// </summary>
        /// <returns>key:公钥 value:公钥和私钥</returns>
        public static KeyValuePair<string, string> GeneralRSAKeys()
        {
            using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider())
            {
                return new KeyValuePair<string, string>(rsa.ToXmlString(false), rsa.ToXmlString(true));
            }
        }

        /// <summary>
        /// RSA加密字符串 (可逆)
        /// </summary>
        /// <param name="text">待加密的字符串</param>
        /// <param name="name">密钥容器的名称。</param>
        /// <param name="binaryStyle">加密后结果内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>加密后的字符串</returns>
        public static string EncryptByRSA(string text, string name, bool binaryStyle = true)
        {
            CspParameters param = new CspParameters() { KeyContainerName = name };
            using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider(param))
            {
                byte[] plaindata = Encoding.UTF8.GetBytes(text);
                byte[] encryptdata = rsa.Encrypt(plaindata, false);
                return binaryStyle ? BitConverter.ToString(encryptdata) : Convert.ToBase64String(encryptdata);
            }
        }

        /// <summary>
        /// RSA解密字符串
        /// </summary>
        /// <param name="text">待解密的字符串</param>
        /// <param name="name">密钥容器的名称。</param>
        /// <param name="binaryStyle">解密内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>解密后的字符串</returns>
        public static string DecryptByRSA(string text, string name, bool binaryStyle = true)
        {
            CspParameters param = new CspParameters() { KeyContainerName = name };
            using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider(param))
            {
                byte[] encryptdata = binaryStyle ? text.GetBytesFromBitConverter() : Convert.FromBase64String(text);
                byte[] decryptdata = rsa.Decrypt(encryptdata, false);
                return Encoding.UTF8.GetString(decryptdata);
            }
        }

        /// <summary>
        /// RSA加密字符串 (可逆)
        /// </summary>
        /// <param name="text">待加密的字符串</param>
        /// <param name="xml">含有密钥信息的xml字符串。</param>
        /// <param name="binaryStyle">加密后结果内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>加密后的字符串</returns>
        public static string EncryptByRSAFromXmlString(string text, string xml, bool binaryStyle = true)
        {
            using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider())
            {
                rsa.FromXmlString(xml);
                byte[] plaindata = Encoding.UTF8.GetBytes(text);
                byte[] encryptdata = rsa.Encrypt(plaindata, false);
                return binaryStyle ? BitConverter.ToString(encryptdata) : Convert.ToBase64String(encryptdata);
            }
        }

        /// <summary>
        /// RSA解密字符串
        /// </summary>
        /// <param name="text">待解密的字符串</param>
        /// <param name="xml">含有密钥信息的xml字符串。</param>
        /// <param name="binaryStyle">解密内容样式。true:HEX格式加密结果，含“-”;Base64格式加密结果。</param>
        /// <returns>解密后的字符串</returns>
        public static string DecryptByRSAFromXmlString(string text, string xml, bool binaryStyle = true)
        {
            using (RSACryptoServiceProvider rsa = new RSACryptoServiceProvider())
            {
                rsa.FromXmlString(xml);
                byte[] encryptdata = binaryStyle ? text.GetBytesFromBitConverter() : Convert.FromBase64String(text);
                byte[] decryptdata = rsa.Decrypt(encryptdata, false);
                return Encoding.UTF8.GetString(decryptdata);
            }
        }

        /// <summary>
        /// 用时间简单混淆
        /// </summary>
        /// <param name="text">原始文本</param>
        /// <param name="timestampLength">混淆用时间戳长度</param>
        /// <returns>混淆后文本</returns>
        public static string MixUp(string text,int timestampLength = 36)
        {
            var timestamp = Guid.NewGuid().ToString();
            var count = text.Length + timestampLength;
            var sbd = new StringBuilder(count);
            int j = 0;
            int k = 0;
            for (int i = 0; i < count; i++)
            {
                if (j < timestampLength && k < text.Length)
                {
                    if (i % 2 == 0)
                    {
                        sbd.Append(text[k]);
                        k++;
                    }
                    else
                    {
                        sbd.Append(timestamp[j]);
                        j++;
                    }
                }
                else if (j >= timestampLength)
                {
                    sbd.Append(text[k]);
                    k++;
                }
                else if (k >= text.Length)
                {
                    break;
                }
            }

            return sbd.ToString();
        }

        /// <summary>
        /// 简单反混淆
        /// </summary>
        /// <param name="text">需要执行反混淆的文本</param>
        /// <param name="timestampLength">混淆用时间戳长度</param>
        /// <returns>原始文本</returns>
        public static string ClearUp(string text, int timestampLength = 36)
        {
            var sbd = new StringBuilder();
            int j = 0;
            for (int i = 0; i < text.Length; i++)
            {
                if (i % 2 == 0)
                {
                    sbd.Append(text[i]);
                }
                else
                {
                    j++;
                }

                if (j > timestampLength)
                {
                    sbd.Append(text.Substring(i));
                    break;
                }
            }

            return sbd.ToString();
        }
    }
}
```

**注意：** 刚刚发现部分代码调用了我自己封装的一个扩展方法的文件，主要是字符串与二进制流转换的处理，可以根据方法名含义自行调整对应方法，后面我也会将这部分代码上传上来，这里就偷个懒不做修改了。
