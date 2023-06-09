# SQRT function<a name="r_SQRT"></a>

 The SQRT function returns the square root of a numeric value\. The square root is a number multiplied by itself to get the given value\.

## Syntax<a name="r_SQRT-synopsis"></a>

```
SQRT (expression)
```

## Argument<a name="r_SQRT-argument"></a>

 *expression*   
The expression must have an integer, decimal, or floating\-point data type\. The expression can include functions\. The system might perform implicit type conversions\. 

## Return type<a name="r_SQRT-return-type"></a>

SQRT returns a DOUBLE PRECISION number\.

## Examples<a name="r_SQRT-examples"></a>

The following example returns the square root of a number\. 

```
select sqrt(16);
               
sqrt
---------------
4
```

The following example performs an implicit type conversion\.

```
select sqrt('16');
               
sqrt
---------------
4
```

The following example nests functions to perform a more complex task\. 

```
select sqrt(round(16.4)); 

sqrt
---------------
4
```

The following example results in the length of the radius when given the area of a circle\. It calculates the radius in inches, for instance, when given the area in square inches\. The area in the sample is 20\. 

```
select sqrt(20/pi());
```

This returns the value 5\.046265044040321\.

The following example returns the square root for COMMISSION values from the SALES table\. The COMMISSION column is a DECIMAL column\. This example shows how you can use the function in a query with more complex conditional logic\. 

```
select sqrt(commission)
from sales where salesid < 10 order by salesid;

sqrt
------------------
10.4498803820905
3.37638860322683
7.24568837309472
5.1234753829798
...
```

The following query returns the rounded square root for the same set of COMMISSION values\. 

```
select salesid, commission, round(sqrt(commission))
from sales where salesid < 10 order by salesid;

salesid | commission | round
--------+------------+-------
      1 |     109.20 |    10
      2 |      11.40 |     3
      3 |      52.50 |     7
      4 |      26.25 |     5
...
```

For more information about sample data in Amazon Redshift, see [Sample database](https://docs.aws.amazon.com/redshift/latest/dg/c_sampledb.html)\.