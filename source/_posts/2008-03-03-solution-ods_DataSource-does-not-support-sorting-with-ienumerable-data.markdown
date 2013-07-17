---
layout: post
title: "Solution to Error: The data source ‘ods_DataSource’ does not support sorting with IEnumerable data"
date: 2008-03-01 20:22
comments: true
categories: techtips
---
Suppose you have a grid view and associated datasource as ods_DataSource. The select method is defined as
{% codeblock DemoObject lang:csharp %}
public ISingleResult SelectMethod(){
//Get DAL Instance DALInstance
return DALInstance.GetDATA(){}
}
{% endcodeblock %}
<!-- more -->
Now if you try to sort on some column the following error pops up.The data source ‘ods_DataSource’ does not support sorting with IEnumerable data. Automatic sorting is only supported with DataView, DataTable, and DataSet.This is because to implement sorting you must have datasource of type DataView/DataTable/DataSet. Now if you are using traditional 3 tier architecture and returning one of these three datatypes everything works fine. The problem arises when you are using Linq. There are two possible solution for this..

+ Implement custom sorting.
+ Change IEnumerable datatype into one of these datatypes.

Here we will try to follow 2nd approach. In our data access layer we will change IEnumerable into a DataTable using an static utility class. Here is the code to perform that operation
{% codeblock  lang:csharp %}
public static class Util
{
public static DataTable ToDataTable(this IEnumerable varlist, CreateRowDelegate fn)
{
DataTable dtReturn = new DataTable();
// column names
PropertyInfo[] oProps = null;
// Could add a check to verify that there is an element 0
foreach (T rec in varlist)
{
// Use reflection to get property names, to create table, Only first time, others will follow
if (oProps == null)
{
oProps = ((Type)rec.GetType()).GetProperties();
foreach (PropertyInfo pi in oProps)
{
// Note that we must check a nullable type else method will throw and error
Type colType = pi.PropertyType; if ((colType.IsGenericType) && (colType.GetGenericTypeDefinition() == typeof(Nullable)))
{
// Since all the elements have same type you can just take the first element and get type
colType = colType.GetGenericArguments()[0];
}
dtReturn.Columns.Add(new DataColumn(pi.Name, colType));
}
}
DataRow dr = dtReturn.NewRow();
//Iterate through each property in PropertyInfo
foreach (PropertyInfo pi in oProps)
{
// Handle null values accordingly
dr[pi.Name] = pi.GetValue(rec, null) == null ? DBNull.Value : pi.GetValue(rec, null);
}
dtReturn.Rows.Add(dr);
}
return (dtReturn);
}
public delegate object[] CreateRowDelegate(T t);
}
Changes in the Select method

public ISingleResult SelectMethod(){
//Get DAL Instance DALInstance
return DALInstance.GetSomeData(){}
}
public DataTable GetSomeData()
{
ISingle result = //code to get result;
return (result.ToDataTable(rec => new object[] { result}));
}
{% endcodeblock %}
As you can see ToDataTable is an extenstion method and will be available to all IEnumerable types.Obviously there are some performance overhead due to use of reflection but still this approach can be used. However for real time applications performance testing must be performed.

Hope this helps!