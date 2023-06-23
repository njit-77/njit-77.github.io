---
layout:     post
title:      C# 简单 操作Sqlite、Mysql数据库
category: 	blog
---

####  准备工作

1、使用Visual Studio 2022创建控制台应用（.NET Framework）,框架选择‘.NET Framework 4.6.2’

2、通过nuget下载 NewLife.Core、NewLife.XCode库

3、在Main函数添加以下代码：

```c#
static void Main(string[] args)
{
    XTrace.UseConsole();
    
    Console.WriteLine("Done");
    Console.ReadLine();
}
```

4、定义测试类如下：

```c#
public class Database
{
    public string id { get; set; }
    
    public DateTime datetime { get; set; }
    
    public int index { get; set; }
    
    public short short_index { get; set; }
    
    public float value { get; set; }
    
    public double dd_value { get; set; }
}
```

#### Sqlite数据库

1、在Database类中添加创建表sql语句

```c#
        public static string GetCreateTableString_Sqlite(string table)
        {
            return $"CREATE TABLE IF NOT EXISTS {table}(" +
                $"id VARCHAR(20) NULL," +
                $"datetime DATETIME NOT NULL," +
                $"int_index INT NULL DEFAULT NULL," +
                $"short_index SMALLINT DEFAULT NULL," +
                $"value FLOAT(9, 3) NULL DEFAULT NULL," +
                $"dd_value DOUBLE(9, 3) NULL DEFAULT NULL);";
        }
```

2、在Database类中添加插入行数据sql语句

```c#
        public string GetInsertString(string table)
        {
            return $"INSERT INTO {table} values" + ValueToString();
        }

        private string ValueToString()
        {
            return $"('{id}','{datetime}','{int_index}','{short_index}','{value}','{dd_value}')";
        }
```

3、添加测试代码

```c#
    public static class TestXCode
    {
        public static void TestSqlite()
        {
            const string database = "testsqlite";

            DAL.AddConnStr(database, $"Data Source={database}.db;", null, "Sqlite");

            /// 创建数据库{database}
            var dal = DAL.Create(database);

            /// 创建表
            dal.Execute(Database.GetCreateTableString_Sqlite(nameof(Database)));

            /// 插入数据
            for (int i = 0; i < 10; i++)
            {
                var data = new Database
                {
                    id = NewLife.Security.Rand.NextString(7),
                    datetime = DateTime.Now,
                    int_index = NewLife.Security.Rand.Next(int.MinValue, int.MaxValue),
                    short_index = (short)NewLife.Security.Rand.Next(short.MinValue, short.MaxValue),
                    value = (float)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                    dd_value = (double)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                };
                dal.Execute(data.GetInsertString(nameof(Database)));
            }
        }
    }
```

结果如下

![sqlite-1](/images/C#简单操作Sqlite、Mysql数据库/sqlite-1.png)

![sqlite-2](/images/C#简单操作Sqlite、Mysql数据库/sqlite-2.png)


#### MySql数据库

1、在Database类中添加创建表sql语句

```c#
        public static string GetCreateTableString_Mysql(string database, string table)
        {
            return $"CREATE TABLE IF NOT EXISTS {database}.{table}(" +
                $"`id` VARCHAR(20) CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_general_ci' NULL COMMENT 'id'," +
                $"`datetime` DATETIME NOT NULL COMMENT 'datetime'," +
                $"`int_index` INT NULL DEFAULT NULL COMMENT 'int_index'," +
                $"`short_index` SMALLINT NULL DEFAULT NULL COMMENT 'short_index'," +
                $"`value` FLOAT(9, 1) NULL DEFAULT NULL COMMENT 'value'," +
                $"`dd_value` DOUBLE(9, 1) NULL DEFAULT NULL COMMENT 'dd_value');";
        }
```

2、在Database类中添加插入行数据sql语句

```c#
        public string GetInsertString(string table)
        {
            return $"INSERT INTO {table} values" + ValueToString();
        }

        private string ValueToString()
        {
            return $"('{id}','{datetime}','{int_index}','{short_index}','{value}','{dd_value}')";
        }
```

3、添加测试代码

```c#
        public static void TestMysql()
        {
            const string database = "testmysql";

            /// 通过数据库sys连接上MySql数据库
            DAL.AddConnStr(database, $"Server=127.0.0.1;Port=3306;User ID=root;Password=root;DataBase=sys;", null, "Mysql");
            var dal = DAL.Create(database);

            /// 创建数据库{database}
            dal.Execute($"CREATE SCHEMA IF NOT EXISTS {database} DEFAULT CHARACTER SET utf8mb4;");

            /// 创建表 position
            dal.Execute(Database.GetCreateTableString_Mysql(database, nameof(Database)));

            /// 此处重新设置连接字符串，指向数据库：{database}
            DAL.AddConnStr(database, $"Server=127.0.0.1;Port=3306;User ID=root;Password=root;DataBase={database};", null, "Mysql");

            /// 插入数据
            for (int i = 0; i < 10; i++)
            {
                var data = new Database
                {
                    id = NewLife.Security.Rand.NextString(7),
                    datetime = DateTime.Now,
                    int_index = NewLife.Security.Rand.Next(int.MinValue, int.MaxValue),
                    short_index = (short)NewLife.Security.Rand.Next(short.MinValue, short.MaxValue),
                    value = (float)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                    dd_value = (double)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                };
                dal.Execute(data.GetInsertString($"{database}.{nameof(Database)}"));
            }
        }
```

结果如下
![mysql-1](/images/C#简单操作Sqlite、Mysql数据库/mysql-1.png)

![mysql-2](/images/C#简单操作Sqlite、Mysql数据库/mysql-2.png)


#### 完整代码

```c#
using NewLife.Log;
using System;
using XCode.DataAccessLayer;

namespace ConsoleApp1
{
    internal class Program
    {
        static void Main(string[] args)
        {
            XTrace.UseConsole();

            TestXCode.TestSqlite();
            TestXCode.TestMysql();

            Console.WriteLine("Done");
            Console.ReadLine();
        }
    }

    public class Database
    {

        public string id { get; set; }

        public DateTime datetime { get; set; }

        public int int_index { get; set; }

        public short short_index { get; set; }

        public float value { get; set; }

        public double dd_value { get; set; }



        public static string GetCreateTableString_Sqlite(string table)
        {
            return $"CREATE TABLE IF NOT EXISTS {table}(" +
                $"id VARCHAR(20) NULL," +
                $"datetime DATETIME NOT NULL," +
                $"int_index INT NULL DEFAULT NULL," +
                $"short_index SMALLINT DEFAULT NULL," +
                $"value FLOAT(9, 3) NULL DEFAULT NULL," +
                $"dd_value DOUBLE(9, 3) NULL DEFAULT NULL);";
        }

        public static string GetCreateTableString_Mysql(string database, string table)
        {
            return $"CREATE TABLE IF NOT EXISTS {database}.{table}(" +
                $"`id` VARCHAR(20) CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_general_ci' NULL COMMENT 'id'," +
                $"`datetime` DATETIME NOT NULL COMMENT 'datetime'," +
                $"`int_index` INT NULL DEFAULT NULL COMMENT 'int_index'," +
                $"`short_index` SMALLINT NULL DEFAULT NULL COMMENT 'short_index'," +
                $"`value` FLOAT(9, 1) NULL DEFAULT NULL COMMENT 'value'," +
                $"`dd_value` DOUBLE(9, 1) NULL DEFAULT NULL COMMENT 'dd_value');";
        }

        public string GetInsertString(string table)
        {
            return $"INSERT INTO {table} values" + ValueToString();
        }

        private string ValueToString()
        {
            return $"('{id}','{datetime}','{int_index}','{short_index}','{value}','{dd_value}')";
        }

    }

    public static class TestXCode
    {
        public static void TestSqlite()
        {
            const string database = "testsqlite";

            DAL.AddConnStr(database, $"Data Source={database}.db;", null, "Sqlite");

            /// 创建数据库{database}
            var dal = DAL.Create(database);

            /// 创建表
            dal.Execute(Database.GetCreateTableString_Sqlite(nameof(Database)));

            /// 插入数据
            for (int i = 0; i < 10; i++)
            {
                var data = new Database
                {
                    id = NewLife.Security.Rand.NextString(7),
                    datetime = DateTime.Now,
                    int_index = NewLife.Security.Rand.Next(int.MinValue, int.MaxValue),
                    short_index = (short)NewLife.Security.Rand.Next(short.MinValue, short.MaxValue),
                    value = (float)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                    dd_value = (double)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                };
                dal.Execute(data.GetInsertString(nameof(Database)));
            }
        }

        public static void TestMysql()
        {
            const string database = "testmysql";

            /// 通过数据库sys连接上MySql数据库
            DAL.AddConnStr(database, $"Server=127.0.0.1;Port=3306;User ID=root;Password=root;DataBase=sys;", null, "Mysql");
            var dal = DAL.Create(database);

            /// 创建数据库{database}
            dal.Execute($"CREATE SCHEMA IF NOT EXISTS {database} DEFAULT CHARACTER SET utf8mb4;");

            /// 创建表 position
            dal.Execute(Database.GetCreateTableString_Mysql(database, nameof(Database)));

            /// 此处重新设置连接字符串，指向数据库：{database}
            DAL.AddConnStr(database, $"Server=127.0.0.1;Port=3306;User ID=root;Password=root;DataBase={database};", null, "Mysql");

            /// 插入数据
            for (int i = 0; i < 10; i++)
            {
                var data = new Database
                {
                    id = NewLife.Security.Rand.NextString(7),
                    datetime = DateTime.Now,
                    int_index = NewLife.Security.Rand.Next(int.MinValue, int.MaxValue),
                    short_index = (short)NewLife.Security.Rand.Next(short.MinValue, short.MaxValue),
                    value = (float)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                    dd_value = (double)(NewLife.Security.Rand.Next(0, 999999999) * Math.Pow(10, -NewLife.Security.Rand.Next(2, 5))),
                };
                dal.Execute(data.GetInsertString($"{database}.{nameof(Database)}"));
            }
        }

    }

}
```

