---
layout: post
title: "Converting Datatable to List of Objects in CSharp"
date: 2018-08-10 19:29:30 +1000
comments: true
categories: 
keywords: CShar0
description: 
---

There are many cases where we will have to convert Dataset into list of objects. Below is a generic method using reflection to achieve that.

Below will work only if datatable column name and class property name are same and they match exactly.

``` Csharp

using System.Reflection
internal static List<T> ConvertDataTableToList<T>(DataTable dt)  
{  
    List<T> data = new List<T>();  
    foreach (DataRow row in dt.Rows)  
    {  
        T item = GetItem<T>(row);  
        data.Add(item);  
    }  
    return data;  
}  
internal static T GetItem<T>(DataRow dr)  
{  
    Type temp = typeof(T);  
    T obj = Activator.CreateInstance<T>();  
  
    foreach (DataColumn column in dr.Table.Columns)  
    {  
        foreach (PropertyInfo pro in temp.GetProperties())  
        {  
            if (pro.Name == column.ColumnName)  
                pro.SetValue(obj, dr[column.ColumnName], null);  
            else  
                continue;  
        }  
    }  
    return obj;  
} 
```

Usage of this will be like below

``` CSharp
orderDetailsList = ConvertDataTable< OrderDetails >(orderdetailDatatable); 

// orderdetailDataset.Table[0] can be used

```