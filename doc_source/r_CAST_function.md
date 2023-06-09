# CAST function<a name="r_CAST_function"></a>

The CAST function converts one data type to another compatible data type\. For instance, you can convert a string to a date, or a numeric type to a string\. CAST performs a runtime conversion, which means that the conversion doesn't change a value's data type in a source table\. It's changed only in the context of the query\.

The CAST function is very similar to [CONVERT function](r_CONVERT_function.md), in that they both convert one data type to another, but they are called differently\.

Certain data types require an explicit conversion to other data types using the CAST or CONVERT function\. Other data types can be converted implicitly, as part of another command, without using CAST or CONVERT\. See [Type compatibility and conversion](c_Supported_data_types.md#r_Type_conversion)\. 

## Syntax<a name="r_CAST_function-syntax"></a>

Use either of these two equivalent syntax forms to cast expressions from one data type to another\.

```
CAST ( expression AS type )
expression :: type
```

## Arguments<a name="r_CAST_function-arguments"></a>

 *expression*   
An expression that evaluates to one or more values, such as a column name or a literal\. Converting null values returns nulls\. The expression cannot contain blank or empty strings\. 

 *type*   
One of the supported [Data types](c_Supported_data_types.md)\. 

## Return type<a name="r_CAST_function-return-type"></a>

CAST returns the data type specified by the *type* argument\. 

**Note**  
Amazon Redshift returns an error if you try to perform a problematic conversion, such as a DECIMAL conversion that loses precision, like the following:   

```
select 123.456::decimal(2,1);
```
or an INTEGER conversion that causes an overflow:   

```
select 12345678::smallint;
```

## Examples<a name="r_CAST_function-examples"></a>

Some of the examples use the sample [TICKIT database](https://docs.aws.amazon.com/redshift/latest/dg/c_sampledb.html)\. For more information about setting up sample data, see [Getting started with Amazon Redshift clusters and data loading](https://docs.aws.amazon.com/redshift/latest/gsg/data-loading.html)\.

The following two queries are equivalent\. They both cast a decimal value to an integer: 

```
select cast(pricepaid as integer)
from sales where salesid=100;

pricepaid
-----------
162
(1 row)
```

```
select pricepaid::integer
from sales where salesid=100;

pricepaid
-----------
162
(1 row)
```

The following produces a similar result\. It doesn't require sample data to run: 

```
select cast(162.00 as integer) as pricepaid;

pricepaid
-----------
162
(1 row)
```

In this example, the values in a timestamp column are cast as dates, which results in removing the time from each result:

```
select cast(saletime as date), salesid
from sales order by salesid limit 10;

 saletime  | salesid
-----------+---------
2008-02-18 |       1
2008-06-06 |       2
2008-06-06 |       3
2008-06-09 |       4
2008-08-31 |       5
2008-07-16 |       6
2008-06-26 |       7
2008-07-10 |       8
2008-07-22 |       9
2008-08-06 |      10
(10 rows)
```

If you didn't use CAST as illustrated in the previous sample, the results would include the time: *2008\-02\-18 02:36:48*\.

The following query casts variable character data to a date\. It doesn't require sample data to run\. 

```
select cast('2008-02-18 02:36:48' as date) as mysaletime;

mysaletime    
--------------------
2008-02-18  
(1 row)
```

In this example, the values in a date column are cast as timestamps: 

```
select cast(caldate as timestamp), dateid
from date order by dateid limit 10;

      caldate       | dateid
--------------------+--------
2008-01-01 00:00:00 |   1827
2008-01-02 00:00:00 |   1828
2008-01-03 00:00:00 |   1829
2008-01-04 00:00:00 |   1830
2008-01-05 00:00:00 |   1831
2008-01-06 00:00:00 |   1832
2008-01-07 00:00:00 |   1833
2008-01-08 00:00:00 |   1834
2008-01-09 00:00:00 |   1835
2008-01-10 00:00:00 |   1836
(10 rows)
```

In a case like the previous sample, you can gain additional control over output formatting by using [TO\_CHAR](https://docs.aws.amazon.com/redshift/latest/dg/r_TO_CHAR.html)\.

In this example, an integer is cast as a character string: 

```
select cast(2008 as char(4));

bpchar
--------
2008
```

In this example, a DECIMAL\(6,3\) value is cast as a DECIMAL\(4,1\) value: 

```
select cast(109.652 as decimal(4,1));

numeric
---------
109.7
```

This example shows a more complex expression\. The PRICEPAID column \(a DECIMAL\(8,2\) column\) in the SALES table is converted to a DECIMAL\(38,2\) column and the values are multiplied by 100000000000000000000: 

```
select salesid, pricepaid::decimal(38,2)*100000000000000000000
as value from sales where salesid<10 order by salesid;


 salesid |           value
---------+----------------------------
       1 | 72800000000000000000000.00
       2 |  7600000000000000000000.00
       3 | 35000000000000000000000.00
       4 | 17500000000000000000000.00
       5 | 15400000000000000000000.00
       6 | 39400000000000000000000.00
       7 | 78800000000000000000000.00
       8 | 19700000000000000000000.00
       9 | 59100000000000000000000.00
(9 rows)
```

**Note**  
You can't perform a CAST or CONVERT operation on the `GEOMETRY` data type to change it to another data type\. However, you can provide a hexadecimal representation of a string literal in extended well\-known binary \(EWKB\) format as input to functions that accept a `GEOMETRY` argument\. For example, the `ST_AsText` function following expects a `GEOMETRY` data type\.   

```
SELECT ST_AsText('01010000000000000000001C400000000000002040');
```

```
st_astext  
------------
 POINT(7 8)
```
You can also explicitly specify the `GEOMETRY` data type\.   

```
SELECT ST_AsText('010100000000000000000014400000000000001840'::geometry);
```

```
st_astext  
------------
 POINT(5 6)
```