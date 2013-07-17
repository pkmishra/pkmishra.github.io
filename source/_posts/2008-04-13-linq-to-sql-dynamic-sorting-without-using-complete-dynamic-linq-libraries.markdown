---
layout: post
title: "Linq to Sql: Dynamic Sorting without using Complete Dynamic Linq Libraries"
date: 2008-04-13 23:11
comments: true
categories: techtips
---
This problem may occur while implementing sorting in GridView. If a storedprocedure is being used either dynamic sql can be created or multiple of case statements can be used. However what if you are just using linq queries. Here are the options

Using Dynamic Linq
Some work arround so that linq query can be generated at runtime.
Essentially 2nd approach is the same as that used in 1st one. But if you just want to implement sorting and do not want to digg into Dynamic Linq libraries you can follow the article…
<!-- more -->
Let’s assume following method expects sortExpression parameter directly passed by UI layer GridView.
{% codeblock  lang:csharp %}
public DataTable GetSomeData(par1, par2…., string sortExpression)
{
    var query = (//Linq query goes here )
    // We want something like this which is not possible as of now
    var query = (some query) (OrderBy SortExpression)
}
Here is the extension method you would like to follow…

public DataTable GetSomeData(par1, par2…., string sortExpression)
{
    var query = (//Linq query goes here )
    // We want something like this which is not possible as of now
    var query = (some query) (OrderBy SortExpression)
}
public static Util
{
    //Thanks to Ernesto for pointing out a small correction in method signature.
    public static IQueryable OrderBy(this IQueryable source, string sortExpression) where TEntity : class
    {
    var type = typeof(TEntity);
    // Remember that for ascending order GridView just returns the column name and for descending it returns column name followed by DESC keyword
    // Therefore we need to examine the sortExpression and separate out Column Name and order (ASC/DESC)
    string[] expressionParts = sortExpression.Split(‘ ‘); // Assuming sortExpression is like [ColoumnName DESC] or [ColumnName]
    string orderByProperty = expressionParts[0];
    string sortDirection = “ASC”;
    string methodName = “OrderBy”;

    //if sortDirection is descending
    if (expressionParts.Length > 1 && expressionParts[1] == “DESC”)
    {
        sortDirection = “Descending”;
        methodName += sortDirection; // Add sort direction at the end of Method name
    }
    var property = type.GetProperty(orderByProperty);
    var parameter = Expression.Parameter(type, “p”);
    var propertyAccess = Expression.MakeMemberAccess(parameter, property);
    var orderByExp = Expression.Lambda(propertyAccess, parameter);
    MethodCallExpression resultExp = Expression.Call(typeof(Queryable), methodName,
    new Type[] { type, property.PropertyType },
    source.Expression, Expression.Quote(orderByExp));
    return source.Provider.CreateQuery(resultExp);
}
}

//Usage will be as of follows…

public DataTable GetSomeData(par1, par2…., string sortExpression)
{
var query = (//Linq query goes here )
// We want something like this which is not possible as of now
var query = (some query)
return query.OrderBy(SortExpression).ToDataTable(rec => new object[] { query}));
}
{% endcodeblock %}
Again OrderBy is an extension method. Hope this helps!