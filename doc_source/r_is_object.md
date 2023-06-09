# IS\_OBJECT function<a name="r_is_object"></a>

Checks whether a variable is an object\. The IS\_OBJECT function returns true for objects, including empty objects\. The function returns false for any other values, including null\.

## Syntax<a name="r_is_object-synopsis"></a>

```
IS_OBJECT (super_expression)
```

## Arguments<a name="r_is_object-arguments"></a>

*super\_expression*  
A SUPER expression or column\.

## Returns<a name="r_is_object-returns"></a>

Boolean

## Example<a name="r_is_object_example"></a>

The following query shows an IS\_OBJECT function\.

```
CREATE TABLE t(s super);

INSERT INTO t VALUES (JSON_PARSE('{"name": "Joe"}'));

SELECT s, is_object(s) FROM t;

       s        | is_object
----------------+-----------
 {"name":"Joe"} | True
(1 row)
```