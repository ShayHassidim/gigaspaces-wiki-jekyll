---
layout: post
title:  SQLQuery
categories: XAP97
parent: querying-the-space.html
weight: 300
---

{% compositionsetup %}
{% summary %}The SQLQuery class is used to query the space using `SQL`-like syntax.{% endsummary %}

# Overview

{% section %}
{% column width=40 %}
The `SQLQuery` class is used to query the space using SQL-like syntax. The query statement includes only the `WHERE` statement part - the selection aspect of a SQL statement is embedded in other parameters for a SQL query.

{% refer %} For the full documentation of the class's methods and constructors, see [Javadoc](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/index.html?com/j_spaces/core/client/SQLQuery.html).{% endrefer %}

{% endcolumn %}
{% column %}
![space-projections.jpg](/attachment_files/space-projections.jpg)
{% endcolumn %}
{% endsection %}

<iframe width="640" height="360" src="//www.youtube.com/embed/jC57mId3SMg?feature=player_detailpage" frameborder="0" allowfullscreen></iframe>

# Usage Examples

An `SQLQuery` is composed from the **type** of entry to query and an **expression** in a SQL syntax.

For example, suppose we have a class called **`MyClass`** with an `Integer` property called **num** and a `String` property called **name**:

{% highlight java %}
// Read an entry of type MyClass whose num property is greater than 500:
MyClass result1 = gigaSpace.read(
    new SQLQuery<MyClass>(MyClass.class, "num > 500"));

// Take an entry of type MyClass whose num property is less than 500:
MyClass result2 = gigaSpace.take(
    new SQLQuery<MyClass>(MyClass.class, "num < 500"));

MyClass[] results;
// Read all entries of type MyClass whose num is between 1 and 100:
results = gigapace.readMultiple(
    new SQLQuery<MyClass>(MyClass.class, "num >= 1 AND num <= 100"));

// Read all entries of type MyClass who num is between 1 and 100 using BETWEEN syntax:
results = gigapace.readMultiple(
    new SQLQuery<MyClass>(MyClass.class, "num BETWEEN 1 AND 100"));

// Read all entries of type MyClass whose num is either 1, 2, or 3:
results = gigapace.readMultiple(
    new SQLQuery<MyClass>(MyClass.class, "num IN (1,2,3)"));

// Read all entries of type MyClass whose num is greater than 1,
// and order the results by the name property:
results = gigapace.readMultiple(
    new SQLQuery<MyClass>(MyClass.class, "num > 1 ORDER BY name"));
{% endhighlight %}

{% refer %} For an example of `SQLQuery` with `EventSession`, refer to the [Session Based Messaging API](./session-based-messaging-api.html#SQLQuery Template Registration) section.{% endrefer %}

# Supported Space Operations

The following operations fully support GigaSpaces `SQLQuery`:

- `count`
- `clear`
- `read`, `readIfExists`, `readMultiple`
- `take`, `takeIfExists`, `takeMultiple`

{% comment %}
- `asyncRead`
- `asyncTake`
{% endcomment %}

The following operations support GigaSpaces `SQLQuery` only with [Simple Queries](#SimpleQueries):

- `snapshot`
- `EventSession`
- `GSIterator`

# Supported SQL Features

GigaSpaces SQLQuery supports the following:

- `AND` / `OR` operators to combine two or more conditions.
- All basic logical operations to create conditions: `=, <>, <, >, >=, <=, like, NOT like, is null, is NOT null, IN`.
- `BETWEEN`
- `ORDER BY (ASC | DESC)` for multiple properties. Supported only by readMultiple. `ORDER BY` supports also nested object fields.
- `GROUP BY` - performs DISTINCT on the properties. Supported only by readMultiple. `GROUP BY` supports also nested object fields.
- `sysdate` - current system date and time.
- `rownum` - limits the number of rows to select.
- Sub queries.
- "." used to indicate a double data type.
- [Regular Index](./indexing.html) and a [Compound Index](./indexing.html#Compound Indexing) - Index a single property or multiple properties to improve query execution time.

# Parameterized Queries

In many cases developers prefer to separate the concrete values from the SQL criteria expression. In GigaSpaces' `SQLQuery` this can be done by placing a **'?'** symbol instead of the actual value in the expression. When executing the query, the conditions that includes **'?'** are replaced with corresponding parameter values supplied via the `setParameter`/`setParameters` methods, or  the `SQLQuery` constructor. For example:

{% highlight java %}
// Option 1 - Use the fluent setParameter(int index, Object value) method:
SQLQuery<MyClass> query1 = new SQLQuery<MyClass>(MyClass.class,
    "num > ? or num < ? and name = ?")
    .setParameter(1, 2)
    .setParameter(2, 3)
    .setParameter(3, "smith");

// Option 2 - Use the setParameters(Object... parameters) method:
SQLQuery<MyClass> query2 = new SQLQuery<MyClass>(MyClass.class,
    "num > ? or num < ? and name = ?");
query.setParameters(2, 3, "smith");

// Option 3: Use the constructor to pass the parameters:
SQLQuery<MyClass> query3 = new SQLQuery<MyClass>(MyClass.class,
    "num > ? or num < ? and name = ?", 2, 3, "smith");
{% endhighlight %}

{% infosign %} The number of **'?'** symbols in the expression string must match the number of parameters set on the query. For example, when using `IN` condition:

{% highlight java %}
SQLQuery<MyClass> query = new SQLQuery<MyClass>(MyClass.class,
    "name = ? AND num IN (?,?,?)");
query.setParameters("A", 1, 2, 3);

// Is equivalent to:
SQLQuery<MyClass> query = new SQLQuery<MyClass>(MyClass.class,
    "name = 'A' AND num IN (1,2,3)");
{% endhighlight %}

{% exclamation %} Parameter assignment to the `SQLQuery` instance is not thread safe. If the query is intended to be executed on multiple threads which may change the parameters, it is recommended to use different `SQLQuery` instances. This has an analogue in JDBC, because `PreparedStatement` is not threadsafe either.

{% exclamation %} In previous options, parameters could be passed via a POJO template as well. This option is still available, but is deprecated and will be removed in future versions.

# Properties Types

## Nested Properties

GigaSpaces SQL syntax contains various extensions to support matching nested properties, maps, collections and arrays.

Some examples:

{% highlight java %}
// Query for a Person who lives in New York:
... = new SQLQuery<Person>(Person.class, "address.city = 'New York'");
// Query for a Dealer which sales a Honda:
... = new SQLQuery<Dealer>(Dealer.class, "cars[*] = 'Honda'");
{% endhighlight %}

For more information see [Query Nested Properties](./query-nested-properties.html).

## Enum Properties

An enum property can be matched either using the enum's instance value or its string representation. For example:

{% highlight java %}
public class Vehicle {
    public enum VehicleType { CAR, BIKE, TRUCK };

    private VehicleType type;
    // Getters and setters are omitted for brevity
}

// Query for vehicles of type CAR using the enum's value:
... = new SQLQuery<Vehicle>(Vehicle.class, "type = ?", VehicleType.CAR);
// Query for vehicles of type CAR using the enum's string representation:
... = new SQLQuery<Vehicle>(Vehicle.class, "type = 'CAR'");
{% endhighlight %}

{% infosign %} When using an Enum string value, the value must be identical (case sensitive) to the name of the Enum value.

## Date Properties

A `Date` property can be matched either using the Date instance value or its string representation. For example:

{% highlight java %}
// Query using a Date instance value:
... = new SQLQuery<MyClass>(MyClass.class, "birthday < ?", new java.util.Date(2020, 11, 20));
// Query using a Date string representation:
... = new SQLQuery<MyClass>(MyClass.class ,"birthday < '2020-12-20'");
{% endhighlight %}

Specifying date and time values as strings is error prone since it requires configuring the date and time format properties and adhering to the selected format. It is recommended to simply use `Date` instance parameters.

When string representation is required, the following space properties should be used:

{% highlight xml %}
space-config.QueryProcessor.date_format
space-config.QueryProcessor.datetime_format
space-config.QueryProcessor.time_format
{% endhighlight %}

For example:

{% highlight xml %}
<beans>
    <os-core:space id="space" url="/./space">
        <os-core:properties>
            <props>
                <prop key="space-config.QueryProcessor.date_format">yyyy-MM-dd HH:mm:ss</prop>
                <prop key="space-config.QueryProcessor.time_format">HH:mm:ss</prop>
            </props>
        </os-core:properties>
    </os-core:space>
</beans>
{% endhighlight %}

These space properties should be configured with a valid Java format pattern as defined in the [official Java language documentation](http://java.sun.com/docs/books/tutorial/i18n/format/simpleDateFormat.html).

{% infosign %} The `space-config.QueryProcessor.date_format` property used when your query include a String representing the date

{% plus %} Date properties are often used for comparison (greater/less than). Consider using [extended indexing](./indexing.html) to boost performance.

## sysdate

The `sysdate` value is evaluated differently when using the JDBC API vs when using it with `SQLQuery` API. When used with JDBC API it is evaluated using the space clock. When used with `SQLQuery` API it is evaluated using the client clock. If you have a partitioned space across multiple different machines and the clock across these machines is not synchronized you might not get the desired results. If you use JDBC API you should consider setting the date value as part of the SQL within the client side (since you  might write objects using the GigaSpace API). In this case , you should synchronize all the client machine time. In short - all the machines (client and server) clocks should be synchronized.

- On windows there is a [windows service](http://technet.microsoft.com/en-us/library/cc773061%28WS.10%29.aspx) that deals with clock synchronization.
- On Linux there is a [daemon service](http://www.brennan.id.au/09-Network_Time_Protocol.html#starting) that deals with clock synchronization.

{% tip %}
GigaSpaces using internally the **TimeStamp** data type to store dates. This means the date includes beyond the year, month and day, the hour/min/sec portions. If you are looking to query for a specific date you should perform a date range query.
{% endtip %}

# Blocking Operations

Blocking operations (i.e. `read` or `take` with `timeout` greater than `0`) are supported with the following restrictions:

- Blocking operations on a partitioned space require a routing value (broadcast is not supported). For more information see [Routing](#Routing).
- Blocking operations on complex queries are not supported. For more information see [Simple Queries](#SimpleQueries) definition.

{% highlight java %}
long timeout = 100000;
MyClass result = space.take(new SQLQuery<MyClass>(MyClass.class ,
     "num > 500"), timeout);
{% endhighlight %}

# Routing

When running on a partitioned space, it is important to understand how routing is determined for SQL queries.

If the routing property is part of the criteria expression with an equality operand and without ORs, its value is used for routing.

For example, suppose the routing property of **`MyClass`** is **`num`**:

{% highlight java %}
// Execute query on partition #1
SQLQuery<MyClass> query1 = new SQLQuery<MyClass>(MyClass.class,
    "num = 1");

// Execute query on all partitions - no way to tell which partitions hold matching results:
SQLQuery<MyClass> query2 = new SQLQuery<MyClass>(MyClass.class,
    "num > 1");

// Execute query on all partitions - no way to tell which partitions hold matching results:
SQLQuery<MyClass> query3 = new SQLQuery<MyClass>(MyClass.class,
    "num = 1 OR name='smith'");
{% endhighlight %}

Note that in `query1` the `num` property is used both for routing and matching.

In some scenarios we may want to execute the query on a specific partition without matching the routing property (e.g. blocking operation). This can be done via the `setRouting` method:

{% highlight java %}
SQLQuery<MyClass> query = new SQLQuery<MyClass>(MyClass.class,
    "num > 3");
query.setRouting(1);
MyClass[] result = gigaspace.readMultiple(query);
{% endhighlight %}

# Regular Expressions

You can query the space using the SQL `like` operator or [Java Regular Expression](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/regex/Pattern.html) Query syntax.

When using the SQL `like` operator you may use the following:
`%` - match any string of any length (including zero length)
`_` - match on a single character

{% highlight java %}
SQLQuery<MyClass> query = new SQLQuery<MyClass>(MyClass.class,
    "name like 'A%'")
{% endhighlight %}

Querying the space using the Java Regular Expression provides more options than the SQL `like` operator. The Query syntax is done using the `rlike` operator:

{% highlight java %}
// Match all entries of type MyClass that have a name that starts with a or c:
SQLQuery<MyClass> query = new SQLQuery<MyClass>(MyClass.class,
    "name rlike '(a|c).*'");
{% endhighlight %}

All the supported methods and options above are relevant also for using `rlike` with `SQLQuery`.

{% exclamation %} `like` and `rlike` queries are not using indexed data, hence executing such may be relatively time consuming compared to other queries that do leverage indexed data. This means the space engine iterate the potential candidate list to find matching object using the Java regular expression utilizes. A machine using 3GHz CPU may iterate 100,000-200,000 objects per second when executing regular expression query. To speed up `like` and `rlike` queries make sure your query leveraging also at least one indexed field to minimize the candidate list. Running multiple partitions will also speed up the query execution since this will allow the system to iterate over the potential matching objects in a parallel manner across the different partitions.

# Free Text Search

Free text search required almost with every application. Users placing some free text into a form and later the system allows users to search for records that includes one or more words within a free text field. A simple way to enable such fast search without using regualr expression query that my have some overhead can be done using the [Collection Indexing](./indexing.html#Collection Indexing), having an array or a collection of String values used for the query. Once the query is executed the SQL Query should use the searched words as usual. See example below:

Our Space class incldues the following - note the **words** and the **freeText** fields:

{% highlight java %}
public class MyData {
	String[] words;
	String freeText;

	@SpaceIndex (path="[*]")
	public String[] getWords() {
		return words;
	}

	public void setWords(String words[]) {
		this.words=words;
	}

	public String getFreeText() {
		return freeText;
	}
	public void setFreeText(String freeText) {
		this.freeText = freeText;
		this.words = freeText.split(" ");
	}
....
}
{% endhighlight %}

{% exclamation %} Note how the **freeText** field is broken into the **words** array before placed into the indexed field.

You may write the data into the space using the following:

{% highlight java %}
MyData data = new MyPOJO(...);
data.setFreeText(freetext);
gigaspace.write(data);
{% endhighlight %}

You can query for objects having the word **hello** as part of the freeText field using the following:

{% highlight java %}
MyData results[] = gigaspace.readMultiple(new SQLQuery<MyData>(MyData.class, words[*]='hello'));
{% endhighlight %}

You can also execute the following to seach for object having the within the freeText field the word **hello** or **everyone**:

{% highlight java %}
MyData results[] = gigaspace.readMultiple(new SQLQuery<MyData>(MyData.class, words[*]='hello' OR words[*]='everyone'));
{% endhighlight %}

With the above approach you avoid the overhead with regular expression queries.

{% tip %}
The same approach can be implemented also with the [SpaceDocument](./document-api.html).
{% endtip %}

{% comment %}
To search for specific words in a specific order within the free text field you should use the indexed field and regular expression with another field that stores the free text.
{% endcomment %}

# Case Insensitive Query

Implementing case insensitive queries can be done via:

- `like` operator or `rlike` operator. Relatively slow. Not recommended when having large amount of objects.
- Store the data in lower case and query on via lower case String value (or upper case)

# Optimization Tips

## Compound Index

When having an **AND** query or a template that use two or more fields for matching a [Compound Index](./indexing.html#Compound Indexing) may boost the query execution time. The Compound Index should be defined on multiple properties for a specific space class and will be used implictly when a SQL Query or a [Template](./template-matching.html) will be using these properties.

## Re-using SQLQuery

Constructing an `SQLQuery` instance is a relatively expensive operation. When possible, prefer using `SQLQuery.setParameters` and `SQLQuery.setParameter` to modify an existing query instead of creating a new one. However, remember that `SQLQuery` is not thread-safe.

## Minimize OR usage

When using the `OR` logical operator together with `AND` logical operator as part of your query you can speed up the query execution by minimizing the number of `OR` conditions in the query. For example:

{% highlight java %}
(A = 'X' OR A = 'Y') AND (B > '2000-10-1' AND B < '2003-11-1')
{% endhighlight %}

would be executed much faster when changing it to be:

{% highlight java %}
(A = 'X' AND B > '2000-10-1' AND B < '2003-11-1')
OR
(A = 'Y' AND B > '2000-10-1' AND B < '2003-11-1')
{% endhighlight %}

## Projecting Partial Results

You can specify that the `SQLQuery` should contain only partial results which means that the returned object should only be populated with the projected properties.
{% refer %}For details on how to use the projection API please refer to [Getting Partial Results Using Projection API](./getting-partial-results-using-projection-api.html){% endrefer %}

# Considerations

## Unsupported SQL Features

GigaSpaces SQLQuery **does not** support the following:

- Aggregate functions: COUNT, MAX, MIN, SUM, AVG are only supported in sub queries (These are fully supported with the [JDBC API](./jdbc-driver.html)).
- Multiple tables select - This is supported with the [JDBC API](./jdbc-driver.html).
- `DISTINCT` - This is supported with the [JDBC API](./jdbc-driver.html).
- The SQL statements: HAVING, VIEW, TRIGGERS, EXISTS, NOT, CREATE USER, GRANT, REVOKE, SET PASSWORD, CONNECT USER, ON.
- Constraints: NOT NULL, IDENTITY, UNIQUE, PRIMARY KEY, Foreign Key/REFERENCES, NO ACTION, CASCADE, SET NULL, SET DEFAULT, CHECK.
- Set operations: Union, Minus, Union All.
- Advanced Aggregate Functions: STDEV, STDEVP, VAR, VARP, FIRST, LAST.
- Mathematical expressions.
- `LEFT OUTER JOIN`
- `RIGHT OUTER JOIN`
- `INNER JOIN`

{% anchor SimpleQueries %}

## Simple vs. Complex Queries

Most space operations and features support any SQL query, but some support only **simple** queries and not **complex** ones.

A query is considered complex if it contains one or more of the following:

- `GROUP BY`
- `ORDER BY`
- Sub queries

The following features support only simple SQL queries

- Snapshot
- Blocking operations
- [Notifications](./session-based-messaging-api.html)
- [GSIterator](./paging-support-with-space-iterator.html)

## Interface Classes

`SQLQuery` supports concrete classes, derived classes and abstract classes. Interface classes are **not supported**.

## Reserved Words

The following are reserved keywords in the GigaSpaces SQL syntax:

{% highlight java %}
alter add all and asc avg between by create call drop desc bit tinyint
 	 end from group in is like rlike max min not null or distinct
 	 order select substr sum sysdate upper where count delete varchar2 char
 	 exception rownum index insert into set table to_char to_number smallint
 	 update union values commit rollback uid using as date datetime time
 	 float real double number decimal numeric boolean integer
 	 varchar bigint long clob blob lob true false int timestamp longvarchar
{% endhighlight %}

If a reserved word needs to be used as a property name it needs to be escaped using ``.
For example: if you need to query a property by the name of count, which is a reserved word, it can be done as following:

{% highlight java %}
new SQLQuery<MyData>(MyData.class, "`count` = 5")
{% endhighlight %}

## Reserved Separators and Operators:
{% highlight java %}
:= || ; . ROWTYPE ~ < <= >  >= => != <> \(+\) ( ) \* / + - ? \{ \}
{% endhighlight %}
