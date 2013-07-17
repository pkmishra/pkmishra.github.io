---
layout: post
title: "How To Check if DataReader has certain field"
date: 2008-03-01 20:22
comments: true
categories: techtips
---
During development I encountered this problem. So here I am posting a solution for the same.
Suppose a reader is reading result set which normally returns fields ‘A’ ‘ B’ AND ‘C’ and in some cases it also returns ‘W’.So your dataread method is likeLet’s have some object for the data to persist.
<!-- more -->
{% codeblock DemoObject lang:csharp %}
Public Object DemoObject
{
//string Property A
//string Property B
//string Property C
//string Property W

//Constructor
Object(string a, string b, string c)
{
A = a;
B = b;
C = c;
W = string.Empty;
}
Object(string a, string b, string c, string w)
{
A = a;
B = b;
C = c;
W = w;
}
}

private DemoObject DataRead(SqlDataReader reader)
{
//Some Code
//
returns object as Object
}

private Object DataRead(SqlDataReader reader)
{
//Some Code
//

if(reader["A"]!=null)
string a = reader.GetString(“A”);
if(reader["A"]!=null)
string b = reader.GetString(“B”);
if(reader["A"]!=null)
string c = reader.GetString(“C”);
//Following line of code throws IndexOutOfRangeException
if(reader["W"]!=null)
string w = reader.GetString(“W”);
//Some Code
//……

returns new DemoObject(a,b,c,w);
}
{% endcodeblock %}

It has the GetOrdinal() method as well, but it throws an exception if the reader doesn’t contain the field
So the solution is to use GetSchemaTable. It returns a table holding the schema of the reader. There is one row in the table for each column returned in the reader, and the columns of the schema table define properties of the reader’s result set, such as the column name, size, data type and so on. We need to filter the rows in that table to just the row matching the column we want, theschema table holds 1 row per column. The easiest way to do this is with the default view. e.g. if I were looking for a row called “myrow” in a reader’s results, I could do this
{% codeblock DemoObject lang:csharp %}
DataView myView = reader.GetSchemaTable().DefaultView;myView.RowFilter = “ColumnName = ‘myrow’ “;
{% endcodeblock %}
So final set of code is
{% codeblock DemoObject lang:csharp %}
//Create a method which verifies if a column exists in a particular row being read by datareader

private bool ColumnExists(SqlDataReader reader, string columnName)
{
reader.GetSchemaTable().DefaultView.RowFilter = “ColumnName= ‘” + columnName + “‘”;
return (reader.GetSchemaTable().DefaultView.Count > 0);
}

So now ReadData method will look like

private Object DataRead(SqlDataReader reader)
{
//Some Code
//

if(reader["A"]!=null)
string a = reader.GetString(“A”);
if(reader["A"]!=null)
string b = reader.GetString(“B”);
if(reader["A"]!=null)
string c = reader.GetString(“C”);
//Check only those columns where you have doubts
if(ColumnExists(reader, “W”))
{
if( reader["W"]!=null)
string w = reader.GetString(“W”);
}
//Some Code
//……

returns new DemoObject(a,b,c,w);
}
{% endcodeblock %}
