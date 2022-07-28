# is\_boolean function<a name="r_is_boolean"></a>

Checks whether a value is a Boolean\. The is\_boolean function returns true for constant JSON Booleans\. The function returns false for any other values, including null\.

## Syntax<a name="r_is_boolean-synopsis"></a>

```
is_boolean (super_expression)
```

## Arguments<a name="r_is_boolean-arguments"></a>

*super\_expression*  
A SUPER expression or column\.

## Returns<a name="r_is_boolean-returns"></a>

Boolean

## Example<a name="r_is_boolean_example"></a>

The following query shows an is\_boolean function\.

```
CREATE TABLE t(s super);

INSERT INTO t VALUES (TRUE);

SELECT s, is_boolean(s) FROM t;
   s    | is_boolean
--------+-----------
 "true" |   True
(1 row)
```