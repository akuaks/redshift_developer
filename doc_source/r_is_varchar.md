# IS\_VARCHAR function<a name="r_is_varchar"></a>

Checks whether a variable is a VARCHAR\. The IS\_VARCHAR function returns true for all strings\. The function returns false for any other values\.

The IS\_VARCHAR function is a superset of the IS\_CHAR function\.

## Syntax<a name="r_is_varchar-synopsis"></a>

```
IS_VARCHAR (super_expression)
```

## Arguments<a name="r_is_varchar-arguments"></a>

*super\_expression*  
A SUPER expression or column\.

## Returns<a name="r_is_varchar-returns"></a>

Boolean

## Example<a name="r_is_varchar_example"></a>

The following query shows an IS\_VARCHAR function\.

```
CREATE TABLE t(s super);

INSERT INTO t VALUES ('abc');

SELECT s, is_varchar(s) FROM t;

   s   | is_varchar
-------+------------
 "abc" | True
(1 row)
```