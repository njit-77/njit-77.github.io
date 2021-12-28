---
layout:     post
title:      WPF DataGrid显示MySQL查询信息，且可删除、修改、插入
category: 	blog
---

1、入行好几年了，工作中使用数据库几率很小(传统行业)。借着十一假期回家机会，学习下数据库。

2、初次了解数据库相关知识，如果本文有误，还望告知。

3、本文主要目的，记录下wpf界面显示数据库信息，且可进行删除、修改、插入命令。并反馈数据到MySQL。做个记录，以便以后工作中使用到时没个头绪。

4、MySQL的基本讲解不再介绍，安装过程什么的，我也是按照网上教程一步一步进行的，假定MySQL已安装成功，且新建有数据库，见下图:

![1](/images/WPF-DataGrid显示MySQL查询信息，且可删除、修改、插入/1.PNG)

废话不多说，直接上代码

界面代码xaml
```xaml
    <Grid>
        <DataGrid x:Name="DataGrid1" HorizontalAlignment="Left" Height="400" Margin="10,10,0,0" VerticalAlignment="Top" Width="537" LoadingRow="DataGrid_LoadingRow">
            <DataGrid.Columns>
                <DataGridTextColumn Header="id" Width="50" Binding="{Binding Path=id}"/>
                <DataGridTextColumn Header="name" Width="*" Binding="{Binding Path=name}"/>
                <DataGridTextColumn Header="phone" Width="*" Binding="{Binding Path=phone}"/>
                <DataGridTextColumn Header="email" Width="*" Binding="{Binding Path=email}"/>
            </DataGrid.Columns>
        </DataGrid>
        <Button x:Name="DeleteButton" Content="删除" Margin="0,40,10,0" VerticalAlignment="Top" Click="DeleteButton_Click" HorizontalAlignment="Right" Width="75"/>
        <Button x:Name="UpdateButton" Content="修改" Margin="0,80,10,0" VerticalAlignment="Top" Click="UpdateButton_Click" HorizontalAlignment="Right" Width="75"/>
        <Button x:Name="InsertButton" Content="插入" Margin="0,120,10,0" VerticalAlignment="Top" Click="InsertButton_Click" HorizontalAlignment="Right" Width="75"/>
    </Grid>
```

后端代码cs
```csharp
    public partial class MainWindow : Window
    {
        //SQLBulkCopy
        Random rd = new Random();
        string sqlstr = "Data Source=127.0.0.1;User ID=root;Password=root;DataBase=test;Charset=utf8;";
        MySql.Data.MySqlClient.MySqlConnection con;
        MySql.Data.MySqlClient.MySqlDataAdapter adapter;
        System.Data.DataSet ds;
        System.Data.DataTable dt;


        public MainWindow()
        {
            InitializeComponent();

            UpdateMySQLData();
        }

        private void DataGrid_LoadingRow(object sender, System.Windows.Controls.DataGridRowEventArgs e)
        {
            e.Row.Header = e.Row.GetIndex() + 1;
        }

        private void UpdateMySQLData()
        {
            if (con == null)
            {
                con = new MySql.Data.MySqlClient.MySqlConnection(sqlstr);
                con.Open();
            }
            if (adapter == null)
            {
                adapter = new MySql.Data.MySqlClient.MySqlDataAdapter("select * from user", con);
            }
            if (ds == null)
            {
                ds = new System.Data.DataSet();
            }
            ds.Clear();
            adapter.Fill(ds, "user");
            if (dt == null)
            {
                dt = ds.Tables["user"];
            }
            DataGrid1.ItemsSource = dt.DefaultView;
        }

        private void DeleteButton_Click(object sender, RoutedEventArgs e)
        {
            int index = DataGrid1.SelectedIndex;
            if (index == -1) return;
#if MySQLCommand
            string DeleteSqlCommand = string.Format("delete from user where id = '{0}'", dt.Rows[index]["id"]);
            MySql.Data.MySqlClient.MySqlCommand cmd = new MySql.Data.MySqlClient.MySqlCommand(DeleteSqlCommand, con);
            cmd.ExecuteNonQuery();

            UpdateMySQLData();
#else
            dt.Rows[index].Delete();
            //dt.Rows.RemoveAt(index);==dt.Rows[index].Delete() + dt.AcceptChanges()
            MySql.Data.MySqlClient.MySqlCommandBuilder builder = new MySql.Data.MySqlClient.MySqlCommandBuilder(adapter);
            adapter.Update(dt);
            dt.AcceptChanges();
#endif
        }

        private void UpdateButton_Click(object sender, RoutedEventArgs e)
        {
#if MySQLCommand
            int index = DataGrid1.SelectedIndex;
            string UpdateSqlCommand = string.Format("update user set id = '{0}', name = '{1}', phone = '{2}', email = '{3}' where id = '{0}'",
                dt.Rows[index]["id"], dt.Rows[index]["name"], dt.Rows[index]["phone"], dt.Rows[index]["email"]);
            MySql.Data.MySqlClient.MySqlCommand cmd = new MySql.Data.MySqlClient.MySqlCommand(UpdateSqlCommand, con);
            cmd.ExecuteNonQuery();

            UpdateMySQLData();
#else
            MySql.Data.MySqlClient.MySqlCommandBuilder builder = new MySql.Data.MySqlClient.MySqlCommandBuilder(adapter);
            adapter.Update(dt);
            dt.AcceptChanges();
#endif
        }

        private void InsertButton_Click(object sender, RoutedEventArgs e)
        {
#if MySQLCommand
            string InsertSqlCommand = string.Format("insert into user(id, name, phone,email) values('{0}','{1}','{2}','{3}')", rd.Next(100), "ZhangSan", 12332112345, "zhangsan@qq.com");
            MySql.Data.MySqlClient.MySqlCommand cmd = new MySql.Data.MySqlClient.MySqlCommand(InsertSqlCommand, con);
            cmd.ExecuteNonQuery();

            string InsertSqlCommand2 = string.Format("insert into user(id, name, phone,email) values('{0}','{1}','{2}','{3}')", rd.Next(100), "LiSi", 12332112345, "lisi@yahoo.com");
            MySql.Data.MySqlClient.MySqlCommand cmd2 = new MySql.Data.MySqlClient.MySqlCommand(InsertSqlCommand2, con);
            cmd2.ExecuteNonQuery();

            UpdateMySQLData();
#else
            System.Data.DataRow dr = dt.NewRow();
            dr[0] = rd.Next(100);
            dr[1] = "ZhangSan";
            dr[2] = "12332112345";
            dr[3] = "zhangsan@qq.com";
            dt.Rows.Add(dr);

            System.Data.DataRow dr2 = dt.NewRow();
            dr2[0] = rd.Next(100);
            dr2[1] = "LiSi";
            dr2[2] = "12332154321";
            dr2[3] = "lisi@yahoo.com";
            dt.Rows.Add(dr2);

            MySql.Data.MySqlClient.MySqlCommandBuilder builder = new MySql.Data.MySqlClient.MySqlCommandBuilder(adapter);
            adapter.Update(ds, "user");
            dt.AcceptChanges();
#endif
        }

    }
```
软件打开界面
![2](/images/WPF-DataGrid显示MySQL查询信息，且可删除、修改、插入/2.PNG)

删除时一直不失败，网上找了好久才找到答案
参考资料
https://blog.csdn.net/sz101/article/details/5837950
https://bbs.csdn.net/wap/topics/390845652
http://www.cnblogs.com/perfect/archive/2007/08/06/844634.html
