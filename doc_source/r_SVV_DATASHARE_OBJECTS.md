# SVV\_DATASHARE\_OBJECTS<a name="r_SVV_DATASHARE_OBJECTS"></a>

Use SVV\_DATASHARE\_OBJECTS to view a list of objects in all datashares created on the cluster or shared with the cluster\. 

SVV\_DATASHARE\_OBJECTS is visible only to superusers\. Other users can't see any rows\.

## Table columns<a name="r_SVV_DATASHARE_OBJECTS-table-columns"></a>

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/redshift/latest/dg/r_SVV_DATASHARE_OBJECTS.html)

## Sample query<a name="r_SVV_DATASHARE_OBJECTS-sample-query"></a>

The following examples return the output for SVV\_DATASHARE\_OBJECTS\.

```
SELECT share_type,
    btrim(share_name)::varchar(16) AS share_name,
    object_type,
    object_name
FROM svv_datashare_objects
WHERE share_name LIKE 'tickit_datashare%'
AND object_name LIKE '%tickit%'
ORDER BY object_name
LIMIT 5;

 share_type |     share_name     | object_type |          object_name
------------+--------------------+-------------+---------------------------------
 OUTBOUND   |  tickit_datashare  |    table    |  public.tickit_category_redshift
 OUTBOUND   |  tickit_datashare  |    table    |  public.tickit_date_redshift
 OUTBOUND   |  tickit_datashare  |    table    |  public.tickit_event_redshift
 OUTBOUND   |  tickit_datashare  |    table    |  public.tickit_listing_redshift
 OUTBOUND   |  tickit_datashare  |    table    |  public.tickit_sales_redshift
```

```
SELECT * FROM SVV_DATASHARE_OBJECTS WHERE share_name like 'sales%';

share_type | share_name | object_type | object_name  | producer_account |          producer_namespace           | include_new
------------+------------+-------------+--------------+------------------+--------------------------------------+-------------
 OUTBOUND   | salesshare | schema      | public       | 123456789012     | 13b8833d-17c6-4f16-8fe4-1a018f5ed00d |      t
 OUTBOUND   | salesshare | table       | public.sales | 123456789012     | 13b8833d-17c6-4f16-8fe4-1a018f5ed00d |
```