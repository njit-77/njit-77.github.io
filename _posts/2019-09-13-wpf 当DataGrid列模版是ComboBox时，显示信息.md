---
layout:     post
title:      wpf 当DataGrid列模版是ComboBox时，显示信息
category: 	blog
---

	实际工作中，有时DataGrid控件某一列显示数据是从Enum集合里面选择出来的，那这时候设置列模版为ComboBox就能满足需求。而关于显示的实际内容，直接是Enum的string()返回值可能不太适合，这时候采用System.ComponentModel.Description是一个很好用的方法。

 代码中定义的显示类型是Enum，实际结果在Description中声明。

定义 Enum Week

```c#
    [System.ComponentModel.Description("星期")]
    public enum Week
    {
        [System.ComponentModel.Description("星期一")]
        Monday,

        [System.ComponentModel.Description("星期二")]
        Tuesday,

        [System.ComponentModel.Description("星期三")]
        Wednesday,

        [System.ComponentModel.Description("星期四")]
        Thursday,

        [System.ComponentModel.Description("星期五")]
        Firday,

        [System.ComponentModel.Description("星期六")]
        Saturday,

        [System.ComponentModel.Description("星期日")]
        Sunday,
    }
```

DataGrid模版：

```xaml
<Grid.Resources>
    <Style x:Key="DataGridTextColumnStyle" TargetType="{x:Type TextBlock}">
        <Setter Property="VerticalAlignment" Value="Center"/>
        <Setter Property="HorizontalAlignment" Value="Center"/>
    </Style>
    
    <Style TargetType="{x:Type DataGrid}">
        <Setter Property="Margin" Value="0"/>
        <Setter Property="Background" Value="#FF8CEB87"/>
        <Setter Property="AutoGenerateColumns" Value="False"/>
        <Setter Property="CanUserAddRows" Value="False"/>
        <Setter Property="CanUserReorderColumns" Value="False"/>
        <Setter Property="CanUserSortColumns" Value="False"/>
        <Setter Property="CanUserResizeColumns" Value="False"/>
        <Setter Property="CanUserResizeRows" Value="False"/>
        <Setter Property="RowHeaderWidth" Value="30"/>
        <Setter Property="RowHeight" Value="30"/>
        <Setter Property="IsReadOnly" Value="True"/>
        <Setter Property="CellStyle">
            <Setter.Value>
                <Style TargetType="DataGridCell">
                    <Style.Triggers>
                        <Trigger Property="IsSelected" Value="True">
                            <Setter Property="Background" Value="Red"/>
                        </Trigger>
                    </Style.Triggers>
                </Style>
            </Setter.Value>
        </Setter>
        <Setter Property="ColumnHeaderStyle">
            <Setter.Value>
                <Style TargetType="DataGridColumnHeader">
                    <Setter Property="HorizontalContentAlignment" Value="Center"/>
                </Style>
            </Setter.Value>
        </Setter>
    </Style>
</Grid.Resources>

<DataGrid ItemsSource="{Binding TestDatas}" LoadingRow="DataGrid_LoadingRow" Margin="0,0,403.6,10">
    <DataGrid.Resources>
        <ObjectDataProvider x:Key="Weeks" MethodName="GetNames" ObjectType="{x:Type sys:Enum}">
            <ObjectDataProvider.MethodParameters>
                <x:Type TypeName="local:Week"/>
            </ObjectDataProvider.MethodParameters>
        </ObjectDataProvider>
        
        <local:WeekEnumToDescriptionConvertor x:Key="WeekEnumToDescription"/>
        <local:WeekEnumToComboBoxIndexConvertor x:Key="WeekEnumToComboBoxIndex"/>
    </DataGrid.Resources>
    
    <DataGrid.Columns>
        <DataGridTemplateColumn Header="Week" Width ="*">
            <DataGridTemplateColumn.CellTemplate>
                <DataTemplate>
                    <ComboBox ItemsSource="{Binding Source={StaticResource Weeks}, Converter={StaticResource WeekEnumToDescription}}"
                              SelectedIndex="{Binding TestWeek, Converter={StaticResource WeekEnumToComboBoxIndex}, UpdateSourceTrigger=PropertyChanged}"/>
                </DataTemplate>
            </DataGridTemplateColumn.CellTemplate>
        </DataGridTemplateColumn>
        
        <DataGridTextColumn Header="Message" 
                            Binding="{Binding TestMsg}" 
                            IsReadOnly="True"
                            ElementStyle ="{StaticResource DataGridTextColumnStyle}"
                            Width ="*"/>
    </DataGrid.Columns>
</DataGrid>
```

WeekEnumToDescriptionConvertor、WeekEnumToComboBoxIndexConvertor实现代码：

```c#
    class WeekEnumToComboBoxIndexConvertor : System.Windows.Data.IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            return ((int)(Week)value);
        }

        public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            return ((Week)(int)value);
        }
    }

    class WeekEnumToDescriptionConvertor : System.Windows.Data.IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            var strValue = value as string[];
            if (strValue != null)
            {
                var enValue = new Week[strValue.Length];
                for (int i = 0; i < strValue.Length; i++)
                {
                    if (Enum.TryParse(strValue[i], out enValue[i]))
                        strValue[i] = enValue[i].GetDescription();
                }
            }
            return value;
        }

        public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
```

EnumHelper代码：

```c#
    public static class EnumHelper
    {
        public static string GetDescription<T>(this T value) where T : struct
        {
            string result = value.ToString();

            var fi = typeof(T).GetField(result);

            var attributes = (System.ComponentModel.DescriptionAttribute[])fi.GetCustomAttributes(
                typeof(System.ComponentModel.DescriptionAttribute), true);

            if (attributes != null && attributes.Length > 0)
            {
                return attributes[0].Description;
            }
            return result;
        }

        public static T GetValueByDescription<T>(this string description) where T : struct
        {
            Type type = typeof(T);
            foreach (var field in type.GetFields())
            {
                if (field.Name == description)
                {
                    return (T)field.GetValue(null);
                }

                var attributes = (System.ComponentModel.DescriptionAttribute[])field.GetCustomAttributes(
                    typeof(System.ComponentModel.DescriptionAttribute), true);

                if (attributes != null && attributes.Length > 0)
                {
                    if (attributes[0].Description == description)
                    {
                        return (T)field.GetValue(null);
                    }
                }
            }
            throw new ArgumentException(string.Format($"{description} 未能找到对应的枚举"), "Description");
        }

        public static T GetValue<T>(this string value) where T : struct
        {
            T result;
            if (Enum.TryParse(value, true, out result))
            {
                return result;
            }
            throw new ArgumentException(string.Format($"{value} 未能找到对应的枚举"), "Value");
        }

    }
```

最终效果图
![](/images/wpf当DataGrid列模版是ComboBox时显示信息/1.gif)

[完整代码](https://files.cnblogs.com/files/njit-77/WpfApp1.7z)
