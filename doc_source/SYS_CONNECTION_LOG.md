# SYS\_CONNECTION\_LOG<a name="SYS_CONNECTION_LOG"></a>

Logs authentication attempts and connections and disconnections\.

This view is visible only to superusers\. For more information, see [Visibility of data in system tables and views](c_visibility-of-data.md)\.

## Table columns<a name="SYS_CONNECTION_LOG-table-columns2"></a>

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/redshift/latest/dg/SYS_CONNECTION_LOG.html)

## Sample queries<a name="SYS_CONNECTION_LOG-sample-queries2"></a>

To view the details for open connections, run the following query\.

```
select record_time, user_name, database_name, remote_host, remote_port
from sys_connection_log
where event = 'initiating session'
and session_id not in 
(select session_id from sys_connection_log
where event = 'disconnecting session')
order by 1 desc;

record_time         | user_name   | database_name   | remote_host   | remote_port                      
--------------------+-------------+-----------------+---------------+---------------------------------
2014-11-06 20:30:06 | rdsdb       | dev             | [local]       |                            
2014-11-06 20:29:37 | test001     | test            | 10.49.42.138  | 11111                           
2014-11-05 20:30:29 | rdsdb       | dev             | 10.49.42.138  | 33333                                                 
2014-11-05 20:28:35 | rdsdb       | dev             | [local]       |  
(4 rows)
```

The following example reflects a failed authentication attempt and a successful connection and disconnection\. 

```
select event, record_time, remote_host, user_name
from sys_connection_log order by record_time;            

            event      |         record_time        |  remote_host  | user_name                      
-----------------------+----------------------------+---------------+---------
authentication failure | 2012-10-25 14:41:56.96391  | 10.49.42.138  | john                                              
authenticated          | 2012-10-25 14:42:10.87613  | 10.49.42.138  | john                                              
initiating session     | 2012-10-25 14:42:10.87638  | 10.49.42.138  | john                                              
disconnecting session  | 2012-10-25 14:42:19.95992  | 10.49.42.138  | john                                              
(4 rows)
```

The following example shows the version of the ODBC driver, the operating system on the client machine, and the plugin used to connect to the Amazon Redshift cluster\. In this example, the plugin used is for standard ODBC driver authentication using a login name and password\.

```
select driver_version, os_version, plugin_name from sys_connection_log;
                
driver_version                          |  os_version                       | plugin_name
----------------------------------------+-----------------------------------+--------------------
Amazon Redshift ODBC Driver 1.4.15.0001 | Darwin 18.7.0 x86_64              | none
Amazon Redshift ODBC Driver 1.4.15.0001 | Linux 4.15.0-101-generic x86_64   | none
```

The following example shows the version of the operating system on the client machine, the driver version, and the protocol version\.

```
select os_version, driver_version, protocol_version from sys_connection_log;
                
os_version                      |  driver_version              | protocol_version
--------------------------------+------------------------------+--------------------
Linux 4.15.0-101-generic x86_64 | Redshift JDBC Driver 2.0.0.0 | 2
Linux 4.15.0-101-generic x86_64 | Redshift JDBC Driver 2.0.0.0 | 2 
Linux 4.15.0-101-generic x86_64 | Redshift JDBC Driver 2.0.0.0 | 2
```