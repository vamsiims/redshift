# TO\_TIMESTAMP function<a name="r_TO_TIMESTAMP"></a>

TO\_TIMESTAMP converts a TIMESTAMP string to TIMESTAMPTZ\.

## Syntax<a name="r_TO_TIMESTAMP-syntax"></a>

```
to_timestamp (timestamp, format)
```

```
to_timestamp (timestamp, format, is_strict)
```

## Arguments<a name="r_TO_TIMESTAMP-arguments"></a>

*timestamp*  
A string that represents a timestamp value in the format specified by *format*\. 

*format*  
A string literal that defines the format of the *timestamp* value\. Formats that include a time zone \(**TZ**, **tz**, or **OF**\) are not supported as input\. For valid timestamp formats, see [Datetime format strings](r_FORMAT_strings.md)\.

*is\_strict*  
An optional Boolean value that specifies whether an error is returned if an input timestamp value is out of range\. When *is\_strict* is set to TRUE, an error is returned if there is an out of range value\. When *is\_strict* is set to FALSE, which is the default, then overflow values are accepted\.

## Return type<a name="r_TO_TIMESTAMP-return-type"></a>

TIMESTAMPTZ

## Examples<a name="r_TO_TIMESTAMP-examples"></a>

The following example demonstrates using the TO\_TIMESTAMP function to convert a TIMESTAMP string to a TIMESTAMPTZ\. 

```
select sysdate, to_timestamp(sysdate, 'YYYY-MM-DD HH24:MI:SS') as seconds;


timestamp                  | seconds
-------------------------- | ----------------------
2021-04-05 19:27:53.281812 | 2021-04-05 19:27:53+00
```

The following SQL statement converts the string '2011\-12\-18 24:38:15' to a TIMESTAMPTZ:

```
SELECT to_timestamp('2011-12-18 24:38:15', 'YYYY-MM-DD HH24:MI:SS');
```

The result is a TIMESTAMPTZ that falls on the next day because the number of hours is more than 24 hours\.

```
to_timestamp
----------------------
2011-12-19 00:38:15+00
```

The following SQL statement converts the string '2011\-12\-18 24:38:15' to a TIMESTAMPTZ:

```
SELECT to_timestamp('2011-12-18 24:38:15', 'YYYY-MM-DD HH24:MI:SS', TRUE);
```

The result is an error because the time value in the timestamp is more than 24 hours\.

```
ERROR:  date/time field time value out of range: 24:38:15.0
```