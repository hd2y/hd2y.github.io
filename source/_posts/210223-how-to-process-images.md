---
title: "如何处理图片"
date: "2021/02/23 11:54:00"
updated: "2021/02/23 11:54:00"
permalink: "how-to-process-images/"
tags:
 - LIS
 - GDI
 - 图片压缩
categories:
 - [开发, C#]
---

在实验室信息系统（LIS）中，除常见定量、定性、说明文本等结果外，图表、标本图片等也是常见的结果表现形式。

例如：血常规、电泳、病理、精子分析、阴道微生态、各种高倍镜镜检等。

常见的数据提供方式是 base64 编码的字节数组、图片路径、数据库二进制数据等，部分也需要根据仪器提供数据，LIS自行绘制折线图、柱状图。

这里以常见的图片提供方式，说明一些常见的图片处理。

## 解析 Base64 图片

常见于血常规图片的处理，部分仪器需要设置位图传输、指定图片类型为位图。

```csharp
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string base64Path = Path.Combine(desktop, "base64.txt");
string base64 = File.ReadAllText(base64Path);

var binary = Convert.FromBase64String(base64);
using (MemoryStream stream = new MemoryStream(binary))
using (Image image = Image.FromStream(stream))
{
    string imagePath = Path.Combine(desktop, $"WBC{image.GetExtension()}");
    image.Save(imagePath);
    Console.WriteLine($"原始图片大小：{binary.Length}");
    Console.WriteLine($"保存到本地图片大小：{new FileInfo(imagePath).Length}");
}
// 原始图片大小：89654
// 保存到本地图片大小：179254
```

> base64 文件内容链接: https://pan.baidu.com/s/1YPPJftih0YecJYys1Jenng  密码: dbfu

## 图片压缩

如上面演示的代码，如果我们直接使用 `Image.Save()` 方法，由于 `GDI+` 的处理，所保存图片大小与原始大小比较有大概一倍的增长。

不过这个流程还是要做，因为这样解析另外一个目的是为了获取图片的格式，以及确保二进制内容存储的是图片内容。

所以以上为错误演示，确认可以正确被解析为图片、解析到图片格式后，应该直接保存字节数组的内容到文件即可，无需使用 `Image` 对象保存。

```csharp
// image.Save(imagePath);
File.WriteAllBytes(imagePath, binary);
```

当然，以上的做法可以避免 GDI 绘图导致的图片增长，但是无法避免另外一个问题：原始图片就比较大。

常见的血常规图片因为是软件绘制，并且文件大小都在 100KB 以内，大小可以接受。

但是部分镜检的仪器，因为输出的是”照片“，所以普遍文件大小都以 MB 计。

但是作为要体现在报告单上的图片，系统并不需要太多的图片细节。而且太大的图片，还会影响报告单文件的大小，所以还需要对图片进行压缩。

其实 GDI 已经提供了一个获取图片缩略图的方法，已经封装到工具类中，可以直接使用：

```csharp
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string base64Path = Path.Combine(desktop, "base64.txt");
string base64 = File.ReadAllText(base64Path);
 
var binary = Convert.FromBase64String(base64);
using (MemoryStream stream = new MemoryStream(binary))
using (Image image = Image.FromStream(stream))
using (Image thumb = image.GetThumbnail())
{
    string imagePath = Path.Combine(desktop, $"WBC{image.GetExtension()}");
    string thumbPath = Path.Combine(desktop, $"WBC-EX{image.GetExtension()}");
    //image.Save(imagePath);
    File.WriteAllBytes(imagePath, binary);
    thumb.Save(thumbPath);
 
    Console.WriteLine($"原始图片大小：{new FileInfo(imagePath).Length}");
    Console.WriteLine($"缩略图大小：{new FileInfo(thumbPath).Length}");
}
// 原始图片大小：89654
// 缩略图大小：1501
```

![how-to-process-images](https://hd2y.oss-cn-beijing.aliyuncs.com/how-to-process-images_1614063890713.png)

可以看到，对于图片细节比较少的位图，图片可以从 88KB 缩小到不足 2K，而且由于没有设置缩略图尺寸，所以在软件中几乎看不到两张图的差异。

而对于大尺寸、高分辨率、细节比较多的图片，还可以进行如下处理：

```csharp
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string imagePath = Path.Combine(desktop, "原始图.jpg");
string thumbPath = Path.Combine(desktop, "缩略图.jpg");
 
using (Image image = Image.FromFile(imagePath))
using (Image thumb = image.GetThumbnail(1000))
{
    // 必须指定保存时的缩略图格式，否则如果按位图输出将没有明显的压缩效果
    thumb.Save(thumbPath, ImageFormat.Jpeg);
 
    Console.WriteLine($"原始图片 尺寸：{image.Width}×{image.Height} 大小：{new FileInfo(imagePath).Length}");
    Console.WriteLine($"缩略图 尺寸：{thumb.Width}×{thumb.Height} 大小：{new FileInfo(thumbPath).Length}");
}
// 原始图片 尺寸：2736×3648 大小：2652323
// 缩略图 尺寸：1000×1334 大小：98715
```

> 当然，优秀的压缩效果，也损失了大量的图片细节，具体如何使用要根据实际情况进行调整。

## 反色处理

如缩略图演示的图片，部分图片为了方便在仪器软件展示，提供的图片为黑底，但是在报告单中不可能展示黑底的图片。

这时常规的做法为将图片进行反色处理（可以在图片编辑软件中使用 Ctrl+Shift+i 看到反色效果）。

同样反色也已经封装在工具类的扩展方法中，调用如下：

```csharp
string desktop = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string imagePath = Path.Combine(desktop, "WBC.bmp");
string invertPath = Path.Combine(desktop, "WBC-EX.bmp");
 
using (Image image = Image.FromFile(imagePath))
using (Image invertImage = image.InvertColors())
{
    invertImage.Save(invertPath);
}
```

![how-to-process-images-02](https://hd2y.oss-cn-beijing.aliyuncs.com/how-to-process-images-02_1614066320084.png)

## 绘图

常见绘制的图片为柱状图、散点图，可以参考文章：[贝克曼 DxH800 血球仪图片绘制问题](https://www.hd2y.net/archives/picture-drawing-of-beckman-dxh800-blood-cell-instrument)

附常用图片相关扩展方法：

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Linq;
using System.Text;

namespace JohnSun.Util
{
    /// <summary>
    /// Image 扩展方法
    /// </summary>
    public static class ImageExtensions
    {
        /// <summary>
        /// 获取图形的字节数组内容
        /// </summary>
        /// <param name="image">原始图片</param>
        /// <param name="format">图片格式</param>
        /// <returns>图片的内容</returns>
        public static byte[] GetBytes(this Image image, ImageFormat format = default)
        {
            using (var stream = new MemoryStream())
            {
                image.Save(stream, new ImageFormat((format ?? image.RawFormat).Guid));
                return stream.ToArray();
            }
        }

        /// <summary>
        /// 获取图形的反色内容
        /// </summary>
        /// <param name="image">原始图片</param>
        /// <returns>反色的图形，同画图工具中 Ctrl+Shift+I 快捷键反色的效果</returns>
        public static Image InvertColors(this Image image)
        {
            Bitmap bitmap = new Bitmap(image);
            for (int x = 0; x < bitmap.Width; x++)
            {
                for (int y = 0; y < bitmap.Height; y++)
                {
                    var pixel = bitmap.GetPixel(x, y);
                    var invertPixel = Color.FromArgb(0xff - pixel.R, 0xff - pixel.G, 0xff - pixel.B);
                    bitmap.SetPixel(x, y, invertPixel);
                }
            }

            return bitmap;
        }

        /// <summary>
        /// 获取图像缩略图
        /// </summary>
        /// <param name="image">原始图像</param>
        /// <param name="width">缩略图宽度，默认为原图宽度，如果只设置了高度会等比缩放</param>
        /// <param name="height">缩略图高度，默认为原图高度，如果只设置了宽度会等比缩放</param>
        /// <returns>返回缩略图</returns>
        public static Image GetThumbnail(this Image image, int width = default, int height = default)
        {
            if (width <= 0) width = height <= 0 ? image.Width : (int)Math.Ceiling(image.Width * (height / (decimal)image.Height));
            if (height <= 0) height = (int)Math.Ceiling(image.Height * (width / (decimal)image.Width));

            return image.GetThumbnailImage(width, height, new Image.GetThumbnailImageAbort(() => true), IntPtr.Zero);
        }

        /// <summary>
        /// 获取图片的原始格式
        /// </summary>
        /// <param name="image">原始图像</param>
        /// <returns>返回 ImageFormat 属性中已确定的格式</returns>
        public static ImageFormat GetRawFormat(this Image image)
        {
            return image.RawFormat.GetRawFormat();
        }

        /// <summary>
        /// 获取图片格式的后缀名
        /// </summary>
        /// <param name="image">原始图像</param>
        /// <returns>返回后缀名，支持 ImageFormat 属性中已确定的格式</returns>
        public static string GetExtension(this Image image)
        {
            return image.RawFormat.GetExtension();
        }

        /// <summary>
        /// 获取图片的原始格式
        /// </summary>
        /// <param name="format">当前从图片中读取的格式</param>
        /// <returns>返回 ImageFormat 属性中已确定的格式</returns>
        public static ImageFormat GetRawFormat(this ImageFormat format)
        {
            if (format.Equals(ImageFormat.MemoryBmp)) return ImageFormat.MemoryBmp;
            else if (format.Equals(ImageFormat.Bmp)) return ImageFormat.Bmp;
            else if (format.Equals(ImageFormat.Emf)) return ImageFormat.Emf;
            else if (format.Equals(ImageFormat.Wmf)) return ImageFormat.Wmf;
            else if (format.Equals(ImageFormat.Gif)) return ImageFormat.Gif;
            else if (format.Equals(ImageFormat.Jpeg)) return ImageFormat.Jpeg;
            else if (format.Equals(ImageFormat.Png)) return ImageFormat.Png;
            else if (format.Equals(ImageFormat.Tiff)) return ImageFormat.Tiff;
            else if (format.Equals(ImageFormat.Exif)) return ImageFormat.Exif;
            else if (format.Equals(ImageFormat.Icon)) return ImageFormat.Icon;
            else return format;
        }

        /// <summary>
        /// 获取图片格式的后缀名
        /// </summary>
        /// <param name="format">当前从图片中读取的格式</param>
        /// <returns>返回后缀名，支持 ImageFormat 属性中已确定的格式</returns>
        public static string GetExtension(this ImageFormat format)
        {
            if (format.Equals(ImageFormat.MemoryBmp)) return ".bmp";
            else if (format.Equals(ImageFormat.Bmp)) return ".bmp";
            else if (format.Equals(ImageFormat.Emf)) return ".emf";
            else if (format.Equals(ImageFormat.Wmf)) return ".wmf";
            else if (format.Equals(ImageFormat.Gif)) return ".gif";
            else if (format.Equals(ImageFormat.Jpeg)) return ".jpg";
            else if (format.Equals(ImageFormat.Png)) return ".png";
            else if (format.Equals(ImageFormat.Tiff)) return ".tiff";
            else if (format.Equals(ImageFormat.Exif)) return ".exif";
            else if (format.Equals(ImageFormat.Icon)) return ".ico";
            else throw new Exception($"未知的图片格式：{format}");
        }
    }
}
```
