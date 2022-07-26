# CREATE USER<a name="r_CREATE_USER"></a>

Creates a new database user account\. You must be a database superuser to run this command\.

## Required privileges<a name="r_CREATE_USER-privileges"></a>

Following are required privileges for CREATE USER:
+ Superuser
+ Users with the CREATE USER privilege

## Syntax<a name="r_CREATE_USER-synopsis"></a>

```
CREATE USER name [ WITH ] 
PASSWORD { 'password' | 'md5hash' | 'sha256hash' | DISABLE }
[ option [ ... ] ]

where option can be:

CREATEDB | NOCREATEDB
| CREATEUSER | NOCREATEUSER
| SYSLOG ACCESS { RESTRICTED | UNRESTRICTED }
| IN GROUP groupname [, ... ]
| VALID UNTIL 'abstime'
| CONNECTION LIMIT { limit | UNLIMITED }
| SESSION TIMEOUT limit
| EXTERNALID external_id
```

## Parameters<a name="r_CREATE_USER-parameters"></a>

 *name*   
The name of the user account to create\. The user name can't be `PUBLIC`\. For more information about valid names, see [Names and identifiers](r_names.md)\.

WITH  
Optional keyword\. WITH is ignored by Amazon Redshift

PASSWORD \{ '*password*' \| '*md5hash*' \| '*sha256hash*' \| DISABLE \}  
Sets the user's password\.   
By default, users can change their own passwords, unless the password is disabled\. To disable a user's password, specify DISABLE\. When a user's password is disabled, the password is deleted from the system and the user can log on only using temporary AWS Identity and Access Management \(IAM\) user credentials\. For more information, see [Using IAM Authentication to Generate Database User Credentials](https://docs.aws.amazon.com/redshift/latest/mgmt/generating-user-credentials.html)\. Only a superuser can enable or disable passwords\. You can't disable a superuser's password\. To enable a password, run [ALTER USER](r_ALTER_USER.md) and specify a password\.  
You can specify the password in clear text, as an MD5 hash string, or as a SHA256 hash string\.   
 When you launch a new cluster using the AWS Management Console, AWS CLI, or Amazon Redshift API, you must supply a clear text password for the initial database user\. You can change the password later by using [ALTER USER](r_ALTER_USER.md)\. 
For clear text, the password must meet the following constraints:  
+ It must be 8 to 64 characters in length\.
+ It must contain at least one uppercase letter, one lowercase letter, and one number\.
+ It can use any ASCII characters with ASCII codes 33–126, except ' \(single quotation mark\), " \(double quotation mark\), \\, /, or @\.
As a more secure alternative to passing the CREATE USER password parameter as clear text, you can specify an MD5 hash of a string that includes the password and user name\.   
When you specify an MD5 hash string, the CREATE USER command checks for a valid MD5 hash string, but it doesn't validate the password portion of the string\. It is possible in this case to create a password, such as an empty string, that you can't use to log on to the database\.
To specify an MD5 password, follow these steps:   

1. Concatenate the password and user name\. 

   For example, for password `ez` and user `user1`, the concatenated string is `ezuser1`\. 

1. Convert the concatenated string into a 32\-character MD5 hash string\. You can use any MD5 utility to create the hash string\. The following example uses the Amazon Redshift [MD5 function](r_MD5.md) and the concatenation operator \( \|\| \) to return a 32\-character MD5\-hash string\. 

   ```
   select md5('ez' || 'user1');
   md5                             
   --------------------------------
   153c434b4b77c89e6b94f12c5393af5b
   ```

1. Concatenate '`md5`' in front of the MD5 hash string and provide the concatenated string as the *md5hash* argument\.

   ```
   create user user1 password 'md5153c434b4b77c89e6b94f12c5393af5b';
   ```

1. Log on to the database using the user name and password\. 

   For this example, log on as `user1` with password `ez`\. 
Another secure alternative is to specify an SHA\-256 hash of a password string; or you can provide your own valid SHA\-256 digest and 256\-bit salt that was used to create the digest\.  
+ Digest – The output of a hashing function\.
+ Salt – Randomly generated data that is combined with the password to help reduce patterns in the hashing function output\.

```
'sha256|Mypassword'
```

```
'sha256|digest|256-bit-salt'
```
In the following example, Amazon Redshift generates and manages the salt\.   

```
CREATE USER admin PASSWORD 'sha256|Mypassword1';
```
In the following example, a valid SHA\-256 digest and 256\-bit salt that was used to create the digest are supplied\.  

```
CREATE USER admin PASSWORD 'sha256|fe95f2bc7c4a111b6f0f7d0b60bfedd1935fb295f8dce1d62708ab8d2f564baf|c721bff5d9042cf541ff7b9d48fa8a6e545c19a763e3710151f9513038b0f6c6';
```
If you set a password in plain text without specifying the hashing function, then an MD5 digest is generated using the username as the salt\. 

CREATEDB \| NOCREATEDB   
The CREATEDB option allows the new user account to create databases\. The default is NOCREATEDB\.

CREATEUSER \| NOCREATEUSER   
The CREATEUSER option creates a superuser with all database privileges, including CREATE USER\. The default is NOCREATEUSER\. For more information, see [Superusers](r_superusers.md)\.

SYSLOG ACCESS \{ RESTRICTED \| UNRESTRICTED \}  <a name="create-user-syslog-access"></a>
A clause that specifies the level of access the user has to the Amazon Redshift system tables and views\.   
If RESTRICTED is specified, the user can see only the rows generated by that user in user\-visible system tables and views\. The default is RESTRICTED\.   
If UNRESTRICTED is specified, the user can see all rows in user\-visible system tables and views, including rows generated by another user\. UNRESTRICTED doesn't give a regular user access to superuser\-visible tables\. Only superusers can see superuser\-visible tables\.   
Giving a user unrestricted access to system tables gives the user visibility to data generated by other users\. For example, STL\_QUERY and STL\_QUERYTEXT contain the full text of INSERT, UPDATE, and DELETE statements, which might contain sensitive user\-generated data\. 
All rows in SVV\_TRANSACTIONS are visible to all users\.   
For more information, see [Visibility of data in system tables and views](c_visibility-of-data.md)\.

IN GROUP *groupname*   
Specifies the name of an existing group that the user belongs to\. Multiple group names may be listed\.

VALID UNTIL *abstime*   
The VALID UNTIL option sets an absolute time after which the user account password is no longer valid\. By default the password has no time limit\.

CONNECTION LIMIT \{ *limit* \| UNLIMITED \}   
The maximum number of database connections the user is permitted to have open concurrently\. The limit isn't enforced for superusers\. Use the UNLIMITED keyword to permit the maximum number of concurrent connections\.  A limit on the number of connections for each database might also apply\. For more information, see [CREATE DATABASE](r_CREATE_DATABASE.md)\. The default is UNLIMITED\. To view current connections, query the [STV\_SESSIONS](r_STV_SESSIONS.md) system view\.  
If both user and database connection limits apply, an unused connection slot must be available that is within both limits when a user attempts to connect\.

SESSION TIMEOUT *limit*  
The maximum time in seconds that a session remains inactive or idle\. The range is 60 seconds \(one minute\) to 1,728,000 seconds \(20 days\)\. If no session timeout is set for the user, the cluster setting applies\. For more information, see [ Quotas and limits in Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/amazon-redshift-limits.html) in the *Amazon Redshift Cluster Management Guide*\.  
When you set the session timeout, it's applied to new sessions only\.  
To view information about active user sessions, including the start time, user name, and session timeout, query the [STV\_SESSIONS](r_STV_SESSIONS.md) system view\. To view information about user\-session history, query the [STL\_SESSIONS](r_STL_SESSIONS.md) view\. To retrieve information about database users, including session\-timeout values, query the [SVL\_USER\_INFO](r_SVL_USER_INFO.md) view\.

EXTERNALID *external\_id*  
The identifier for the user, which is associated with an identity provider\. The user must have their password disabled\. For more information, see [Native identity provider \(IdP\) federation for Amazon Redshift](https://docs.aws.amazon.com/redshift/latest/mgmt/redshift-iam-access-control-native-idp.html)\.

### Usage notes<a name="create_user-usage-notes"></a>

By default, all users have CREATE and USAGE privileges on the PUBLIC schema\. To disallow users from creating objects in the PUBLIC schema of a database, use the REVOKE command to remove that privilege\.

When using IAM authentication to create database user credentials, you might want to create a superuser that is able to log on only using temporary credentials\. You can't disable a superuser's password, but you can create an unknown password using a randomly generated MD5 hash string\.

```
create user iam_superuser password 'md5A1234567890123456780123456789012' createuser;
```

The case of a *username* enclosed in double quotation marks is always preserved regardless of the setting of the `enable_case_sensitive_identifier` configuration option\. For more information, see [enable\_case\_sensitive\_identifier](r_enable_case_sensitive_identifier.md)\.

## Examples<a name="r_CREATE_USER-examples"></a>

The following command creates a user account named dbuser, with the password "abcD1234", database creation privileges, and a connection limit of 30\.

```
create user dbuser with password 'abcD1234' createdb connection limit 30;
```

 Query the PG\_USER\_INFO catalog table to view details about a database user\. 

```
select * from pg_user_info;
 usename   | usesysid | usecreatedb | usesuper | usecatupd | passwd   | valuntil | useconfig | useconnlimit
-----------+----------+-------------+----------+-----------+----------+----------+-----------+-------------
 rdsdb     |        1 | true        | true     | true      | ******** | infinity |           |             
 adminuser |      100 | true        | true     | false     | ******** |          |           | UNLIMITED   
 dbuser    |      102 | true        | false    | false     | ******** |          |           | 30
```

In the following example, the account password is valid until June 10, 2017\.

```
create user dbuser with password 'abcD1234' valid until '2017-06-10';
```

 The following example creates a user with a case\-sensitive password that contains special characters\.

```
create user newman with password '@AbC4321!';
```

 To use a backslash \('\\'\) in your MD5 password, escape the backslash with a backslash in your source string\. The following example creates a user named `slashpass` with a single backslash \( '`\`'\) as the password\. 

```
select md5('\\'||'slashpass');
md5                             
--------------------------------
0c983d1a624280812631c5389e60d48c
 
create user slashpass password 'md50c983d1a624280812631c5389e60d48c';
```

The following example creates a user named `dbuser` with an idle\-session timeout set to 120 seconds\.

```
CREATE USER dbuser password 'abcD1234' SESSION TIMEOUT 120;
```

The following example creates a user named `bob`\. The namespace is `myco_aad`\.

```
CREATE USER myco_aad:bob EXTERNALID "ABC123" PASSWORD DISABLE;
```