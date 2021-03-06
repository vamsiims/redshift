# is\_integer function<a name="r_is_integer"></a>

Returns true for numbers of scale 0 in the 32\-bit range, and false for anything else \(including null and floating point numbers\)\.

The is\_integer function is a superset of the is\_smallint function\.

## Syntax<a name="r_is_integer-synopsis"></a>

```
is_integer (super_expression)
```

## Arguments<a name="r_is_integer-arguments"></a>

*super\_expression*  
A SUPER expression or column\.

## Returns<a name="r_is_integer-returns"></a>

Boolean

## Example<a name="r_is_integer_example"></a>

The following example shows an is\_integer function\.

```
CREATE TABLE t(s super);

INSERT INTO t VALUES (5);

SELECT s, is_integer(s) FROM t;

 s | is_integer
---+------------
 5 | True
(1 row)
```