---
title: "年龄的计算问题 C# 篇"
date: "2019/03/30 15:42:00"
updated: "2019/03/30 15:42:00"
permalink: "the-problem-of-calculating-age-at-csharp"
tags:
 - 年龄
categories:
 - [开发, C#]
---

## 前言

程序中经常会遇到需要计算具体年龄的问题，所以花费了一些时间思考了解决方案，根据网上已有的计算方式，做了简单封装与测试调优，最后封装了一个`Age`类完成年龄的计算。

## 单位问题

目前在项目中接触到的年龄单位分别是 岁、月、周、天、时、分，但是由于有些系统要求，年龄需要尽量具体，所以这里提供了一个标志枚举。至于具体什么是标志枚举，这里还没有特别做过介绍，所以大家可以搜索`枚举`与`Flags`特性了解。以后有机会介绍`Logger`帮助类的时候再具体展开为大家介绍。

`AgeUnit` 枚举代码：
```csharp
/// <summary>
/// 年龄单位
/// </summary>
[Flags]
public enum AgeUnit
{
    /// <summary>
    /// 岁
    /// </summary>
    Year = 64,

    /// <summary>
    /// 月
    /// </summary>
    Month = 32,

    /// <summary>
    /// 周
    /// </summary>
    Week = 16,

    /// <summary>
    /// 天
    /// </summary>
    Day = 8,

    /// <summary>
    /// 时
    /// </summary>
    Hour = 4,

    /// <summary>
    /// 分
    /// </summary>
    Minute = 2,

    /// <summary>
    /// 秒
    /// </summary>
    Second = 1
}
```

## 周岁年龄

年龄计算其实很简单，这个和我们小学时候做减法是一样的，首先从小单位着手运算，当相减为负数时向上一个单位`借1`。

例如：当前时间是 `2019-03-30 16:00:00`，宝宝的出生日期是 `2019-03-28 20:00:00`，这个时候我们用 `16时 - 20时` 是不可行的，所以向日期借一天，就可以知道宝宝的实际年龄是 `1天20时`。

这样看来是不是其实就是小学问题，但是有些人被年月日时分秒不同的进制搞得晕头转向，于是简单的问题被复杂化。

另外一点就是年月日之间的换算，进制是在变化的，但是这主要是由于每个月的天数不同，但是程序中已经提供了方法可以轻而易举的获取到某一年的某一个月的天数，所以这点我们也不用担心，下面我们看一下周岁年龄的计算：

`Age` 类型代码：
```csharp
/// <summary>
/// 周岁年龄
/// </summary>
public class Age
{
    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="age">年龄的书面表达形式</param>
    /// <param name="current">截止时间 用来计算出生日期</param>
    public Age(string age, DateTime? current = null)
    {
        Current = current == null ? DateTime.Now : current.Value;
        ParseAge(age);
    }

    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="birthday">生日</param>
    /// <param name="current">截止时间 默认当前时间</param>
    public Age(DateTime birthday, DateTime? current = null)
    {
        Birthday = birthday;
        Current = current == null ? DateTime.Now : current.Value;
        GetAge();
    }

    /// <summary>
    /// 生日
    /// </summary>
    public DateTime Birthday { get; set; }

    /// <summary>
    /// 截止时间
    /// </summary>
    public DateTime Current { get; set; }

    /// <summary>
    /// 差值
    /// </summary>
    public TimeSpan TimeSpan { get; set; }

    /// <summary>
    /// 年龄单位
    /// </summary>
    public AgeUnit AgeUnit { get; set; }

    /// <summary>
    /// 岁
    /// </summary>
    public int Year { get; set; }

    /// <summary>
    /// 月
    /// </summary>
    public int Month { get; set; }

    /// <summary>
    /// 周
    /// </summary>
    public int Week { get; set; }

    /// <summary>
    /// 天
    /// </summary>
    public int Day { get; set; }

    /// <summary>
    /// 时
    /// </summary>
    public int Hour { get; set; }

    /// <summary>
    /// 分
    /// </summary>
    public int Minute { get; set; }

    /// <summary>
    /// 秒
    /// </summary>
    public int Second { get; set; }

    /// <summary>
    /// 获取年龄指定单位的数值（非总年龄）
    /// </summary>
    /// <param name="unit">年龄单位</param>
    /// <returns>该单位下的年龄</returns>
    public virtual int GetAgeByAgeUnit(AgeUnit unit)
    {
        if (unit == AgeUnit.Year)
            return Year;
        else if (unit == AgeUnit.Month)
            return Month;
        else if (unit == AgeUnit.Week)
            return Week;
        else if (unit == AgeUnit.Day)
            return Day;
        else if (unit == AgeUnit.Hour)
            return Hour;
        else if (unit == AgeUnit.Minute)
            return Minute;
        else if (unit == AgeUnit.Second)
            return Second;
        else
            throw new ArgumentOutOfRangeException(nameof(unit));
    }

    /// <summary>
    /// 年龄单位
    /// </summary>
    protected virtual Dictionary<AgeUnit, string> AgeUnits { get; }
        = new Dictionary<AgeUnit, string>()
        {
            [AgeUnit.Year] = "岁",
            [AgeUnit.Month] = "月",
            [AgeUnit.Week] = "周",
            [AgeUnit.Day] = "天",
            [AgeUnit.Hour] = "时",
            [AgeUnit.Minute] = "分",
            [AgeUnit.Second] = "秒",
        };

    /// <summary>
    /// 用于提取年龄信息的正则字段
    /// </summary>
    protected Regex _regex;
    /// <summary>
    /// 用于提取年龄信息的正则
    /// </summary>
    protected virtual Regex Regex
    {
        get
        {
            if (_regex == null)
            {
                _regex = new Regex($@"(\d+)\D*(({string.Join(")|(", AgeUnits.Values.ToArray())}))\D*");
            }
            return _regex;
        }
    }

    /// <summary>
    /// 重写ToString方法
    /// </summary>
    /// <returns>年龄</returns>
    public override string ToString()
    {
        StringBuilder builder = new StringBuilder();

        for (int i = (int)AgeUnit.Year; i > 0; i /= 2)
            if (((AgeUnit)i & AgeUnit) != 0 && GetAgeByAgeUnit((AgeUnit)i) != 0)
                builder.Append(GetAgeByAgeUnit((AgeUnit)i)).Append(AgeUnits[(AgeUnit)i]);

        if (string.IsNullOrEmpty(builder.ToString()))
            builder.Append(0).Append(AgeUnits[AgeUnit.Year]);

        return builder.ToString();
    }

    /// <summary>
    /// 计算详细的周岁年龄
    /// </summary>
    protected virtual void GetAge()
    {
        if (Birthday > Current)
        {
            throw new ArgumentException("出生日期不能大于当前时间！");
        }

        //出生时间信息
        int bYear = Birthday.Year;
        int bMonth = Birthday.Month;
        int bDay = Birthday.Day;
        int bHour = Birthday.Hour;
        int bMinute = Birthday.Minute;
        int bSecond = Birthday.Second;

        //当前时间信息
        int nYear = Current.Year;
        int nMonth = Current.Month;
        int nDay = Current.Day;
        int nHour = Current.Hour;
        int nMinute = Current.Minute;
        int nSecond = Current.Second;

        //若出生时间信息小于当前时间 调整其差值为正整数
        if (nSecond < bSecond)
        {
            nMinute--;
            nSecond += 60;
        }
        if (nMinute < bMinute)
        {
            nHour--;
            nMinute += 60;
        }
        if (nHour < bHour)
        {
            nDay--;
            nHour += 24;
        }
        if (nDay < bDay)
        {
            nMonth--;
            nDay += DateTime.DaysInMonth(bYear, bMonth);
        }
        if (nMonth < bMonth)
        {
            nYear--;
            nMonth += 12;
        }

        //属性赋值
        Year = nYear - bYear;
        Month = nMonth - bMonth;
        Day = nDay - bDay;
        Hour = nHour - bHour;
        Minute = nMinute - bMinute;
        Second = nSecond - bSecond;
        TimeSpan = Current - Birthday;
        if (Year > 0)
            AgeUnit = AgeUnit | AgeUnit.Year;
        if (Month > 0)
            AgeUnit = AgeUnit | AgeUnit.Month;
        if (Week > 0)
            AgeUnit = AgeUnit | AgeUnit.Week;
        if (Day > 0)
            AgeUnit = AgeUnit | AgeUnit.Day;
        if (Hour > 0)
            AgeUnit = AgeUnit | AgeUnit.Hour;
        if (Minute > 0)
            AgeUnit = AgeUnit | AgeUnit.Minute;
        if (Second > 0)
            AgeUnit = AgeUnit | AgeUnit.Second;
    }

    /// <summary>
    /// 年龄转换
    /// </summary>
    protected virtual void ParseAge(string input)
    {
        Match match = Regex.Match(input);
        MatchCollection collection = Regex.Matches(input);

        if (collection.Count <= 0)
            throw new ArgumentException(nameof(input));
        else
        {
            foreach (Match item in collection)
            {
                if (int.TryParse(item.Groups[1].Value, out int age))
                {
                    AgeUnit unit = AgeUnits.FirstOrDefault(kv => kv.Value == item.Groups[2].Value).Key;
                    SetAge(unit, age);
                }
                else
                    throw new ArgumentException(nameof(input));
            }
        }

        GetBirthdayByAge();
        TimeSpan = Current - Birthday;
    }

    /// <summary>
    /// 通过年龄信息获取出生日期信息
    /// </summary>
    protected virtual void GetBirthdayByAge()
    {
        Birthday = Current.AddYears(-Year).AddMonths(-Month).AddDays(-Week * 7).AddDays(-Day).AddHours(-Hour).AddMinutes(-Minute).AddSeconds(-Second);
    }

    /// <summary>
    /// 设置年龄信息
    /// </summary>
    /// <param name="unit">年龄单位</param>
    /// <param name="age">年龄</param>
    protected virtual void SetAge(AgeUnit unit, int age)
    {
        if (age == 0)
            return;
        AgeUnit = AgeUnit | unit;
        if (unit == AgeUnit.Year)
            Year += age;
        else if (unit == AgeUnit.Month)
            Month += age;
        else if (unit == AgeUnit.Week)
            Week += age;
        else if (unit == AgeUnit.Day)
            Day += age;
        else if (unit == AgeUnit.Hour)
            Hour += age;
        else if (unit == AgeUnit.Minute)
            Minute += age;
        else if (unit == AgeUnit.Second)
            Second += age;
        else
            throw new ArgumentException(nameof(unit));
    }
}
```

以上主要需要看一下 `GetAge` 方法，通过 ` DateTime.DaysInMonth(year, month)` 获取到出生月份的总天数，后面的运算就水到渠成了。

## 医学年龄

使用`Age`类进行计算虽然可以准确的计算一个人的周岁年龄，但是碰到一些极端情况，仍然存在问题。

例如一个人的年龄是 `1月30天`，这是由于出生月份的天数是`31天`导致的；再比如一个人的年龄是`1月`，但实际上只有`29天`或者`28天`，这是由于出生月份在`2月份`。

这样可能会在一些对年龄计算有要求的系统中比如医疗机构的系统，引起用户的误解。当然我们可以与客户沟通，了解对方需求后对`Age`类型继承，然后提供出满足用户需求的计算年龄的解决方案。

例如以下是`HIS`提供商与医院沟通后确定的：固定`1岁=365天`，`1月=30天`进行年龄计算的方案：

`MedicalAge` 类型代码：
```csharp
/// <summary>
/// 医学年龄
/// </summary>
public class MedicalAge : Age
{
    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="age">年龄的书面表达形式</param>
    /// <param name="current">截止时间 用来计算出生日期</param>
    public MedicalAge(string age, DateTime? current = null) : base(age, current)
    {

    }

    /// <summary>
    /// 构造函数
    /// </summary>
    /// <param name="birthday">生日</param>
    /// <param name="current">截止时间 默认当前时间</param>
    public MedicalAge(DateTime birthday, DateTime? current = null) : base(birthday, current)
    {

    }

    /// <summary>
    /// 年龄1
    /// </summary>
    public int? Age1 { get; set; }

    /// <summary>
    /// 年龄2
    /// </summary>
    public int? Age2 { get; set; }

    /// <summary>
    /// 年龄单位1
    /// </summary>
    public AgeUnit? AgeUnit1 { get; set; }

    /// <summary>
    /// 年龄单位2
    /// </summary>
    public AgeUnit? AgeUnit2 { get; set; }

    /// <summary>
    /// 重写ToString方法
    /// </summary>
    /// <returns>年龄</returns>
    public override string ToString()
    {
        StringBuilder builder = new StringBuilder();
        if (Age1 != null)
            builder.Append(Age1.Value).Append(AgeUnits[AgeUnit1.Value]);
        if (Age2 != null)
            builder.Append(Age2.Value).Append(AgeUnits[AgeUnit2.Value]);

        if (string.IsNullOrEmpty(builder.ToString()))
            builder.Append(0).Append(AgeUnits[AgeUnit.Year]);

        return builder.ToString();
    }

    /// <summary>
    /// 计算医学年龄
    /// </summary>
    protected override void GetAge()
    {
        if (Birthday > Current)
        {
            throw new ArgumentException("出生日期不能大于当前时间！");
        }

        //属性赋值
        TimeSpan = Current - Birthday;
        Year = TimeSpan.Days / 365;
        Month = TimeSpan.Days % 365 / 30;
        Week = TimeSpan.Days % 365 % 30 / 7;
        Day = TimeSpan.Days % 365 % 30 % 7;
        Hour = TimeSpan.Hours;
        Minute = TimeSpan.Minutes;
        Second = TimeSpan.Seconds;
        if (Year > 0)
            AgeUnit = AgeUnit | AgeUnit.Year;
        if (Month > 0)
            AgeUnit = AgeUnit | AgeUnit.Month;
        if (Week > 0)
            AgeUnit = AgeUnit | AgeUnit.Week;
        if (Day > 0)
            AgeUnit = AgeUnit | AgeUnit.Day;
        if (Hour > 0)
            AgeUnit = AgeUnit | AgeUnit.Hour;
        if (Minute > 0)
            AgeUnit = AgeUnit | AgeUnit.Minute;
        if (Second > 0)
            AgeUnit = AgeUnit | AgeUnit.Second;

        for (int i = (int)AgeUnit.Year; i > 0; i /= 2)
        {
            if (((AgeUnit)i & AgeUnit) != 0)
            {
                if (AgeUnit1 == null)
                {
                    AgeUnit1 = (AgeUnit)i;
                    Age1 = GetAgeByAgeUnit(AgeUnit1.Value);
                }
                else if (AgeUnit2 == null)
                {
                    AgeUnit2 = (AgeUnit)i;
                    Age2 = GetAgeByAgeUnit(AgeUnit2.Value);
                }
                else
                    break;
            }
        }
    }

    /// <summary>
    /// 通过年龄信息获取出生日期信息
    /// </summary>
    protected override void GetBirthdayByAge()
    {
        Birthday = Current.AddDays(-Year * 365 - Month * 30 - Week * 7 - Day).AddHours(-Hour).AddMinutes(-Minute).AddSeconds(-Second);
        GetMedicalAge();
    }

    /// <summary>
    /// 获取年龄1与年龄2
    /// </summary>
    protected virtual void GetMedicalAge()
    {
        for (int i = (int)AgeUnit.Year; i > 0; i /= 2)
        {
            if (((AgeUnit)i & AgeUnit) != 0)
            {
                if (AgeUnit1 == null)
                {
                    AgeUnit1 = (AgeUnit)i;
                    Age1 = GetAgeByAgeUnit(AgeUnit1.Value);
                }
                else if (AgeUnit2 == null)
                {
                    AgeUnit2 = (AgeUnit)i;
                    Age2 = GetAgeByAgeUnit(AgeUnit2.Value);
                }
                else
                    break;
            }
        }
    }
}
```

## 测试

除提供了根据出生日期计算年龄以外，上文还简单提供了一个根据年龄获取出生日期的方案，可以参看上文`Age`与`MedicalAge`第一个入参为字符串类型的构造函数进行了解。

至于年龄输出的话，暂时重写了`ToString`方法，进行格式化的输出，以下是简单的测试代码：

```csharp
public static void Main()
{
    TestGetAge(new DateTime(1931, 09, 18));
    TestGetAge(new DateTime(1931, 09, 18, 23, 59, 59));
    TestGetAge(new DateTime(1949, 10, 01));
    TestGetAge(DateTime.Now.Date.AddDays(-50));
    TestGetAge(DateTime.Now);
    Console.ReadKey();
}

public static void TestGetAge(DateTime birthday)
{
    Age age = new Age(birthday);
    Console.WriteLine($"出生时间为：{age.Birthday.ToString("yyyy-MM-dd HH:mm:ss")} 的 年龄为：{age.ToString()}");
    Age tempAge = new Age(age.ToString());
    Console.WriteLine($"出生时间为：{tempAge.Birthday.ToString("yyyy-MM-dd HH:mm:ss")} 的 年龄为：{tempAge.ToString()}");
    Age medicalAge = new MedicalAge(birthday);
    Console.WriteLine($"出生时间为：{medicalAge.Birthday.ToString("yyyy-MM-dd HH:mm:ss")} 的 年龄为：{medicalAge.ToString()}");
    Age tempMedicalAge = new MedicalAge($"{medicalAge.Year}岁{medicalAge.Month}月{medicalAge.Week}周{medicalAge.Day}天{medicalAge.Hour}时{medicalAge.Minute}分{medicalAge.Second}秒");
    Console.WriteLine($"出生时间为：{tempMedicalAge.Birthday.ToString("yyyy-MM-dd HH:mm:ss")} 的 年龄为：{tempMedicalAge.ToString()}");
}
```

## 总结

年龄计算从上文来看其实从技术上来讲并不困难，但是主要的问题在于，要与用户进行沟通达成共识。

否则从日常理解来说，特别是小单位需要细致到月周天的年龄的计算，很容易让人误解。
