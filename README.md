## 顯示當前使用者
``` SQL
select USER();

select CURRENT_USER();
```
CURRENT_USER shows who you are authenticated as, while USER shows who you tried to authenticate as.


## grants
### 顯示使用者的grants
``` sql
show grants for `user_name`@`ip_address`
```


### 顯示當前使用者的grants
```sql
show grants for CURRENT_USER;

show grants;
```

### GRANT 語法
```sql
GRANT ALL PRIVILEGES ON *.* TO `user_name`@`ip_address`;
```

### 移除權限
```sql
revoke all ON *.*  From `user_name`@`ip_address`;
```

### 情境:限制帳號只能讀取某張表格
```sql
GRANT SELECT ON database.`table` TO `user_name`@`ip_address`;
```


## Privilege level
* Global: \*.\*
* Database: [db_name].*
* Table: [db_name].[table_name]
* Stored routine: [db_name].[routine_name]  
(總共有六種級別，另外兩種column、proxy級別)

global 的權限:例如`create user`，只能用在全域
```sql
GRANT CREATE USER ON *.* TO `user_name`@'localhost';
```

有些權限可以在global，也可以在db級使用
```sql
GRANT SELECT ON Database.* TO `user_name`@'localhost';
GRANT SELECT ON *.* TO `user_name`@'localhost';
```

## GRANT OPTION
```SQL
GRANT ALL PRIVILEGES ON *.* TO `user_name`@`ip_address` WITH GRANT OPTION;
```
ex:  
當沒有轉移權限時，無法使用grant 
```sql
GRANT select ON drug.* TO USER
```
給予轉移權限  
```sql
grant select,drop on drug.* to patrick with grant option
```
* 無法授予超過自己grant option級別的權限
  ```sql
  GRANT select ON *.* TO USER
  ```
* 要授予轉移權限時，必須自己擁有grant option，但不一定需要有操作權限
  ```
  GRANT USAGE ON *.* TO USER WITH GRANT OPTION
  ```


## RENAME
```sql
RENAME USER `patrick`@`localhost` TO `jeff`@`%`;
```
此語法可以連同此帳號其他grants一併修改

## FLUSH PRIVILEGES
使用 GRANT, REVOKE, SET PASSWORD, and RENAME USER 語法不需要再下 `FLUSH PRIVILEGES`
> If you modify the grant tables indirectly using an account-management statement, the server notices these changes and loads the grant tables into memory again immediately. Account-management statements are described in Section 13.7.1, “Account Management Statements”. Examples include GRANT, REVOKE, SET PASSWORD, and RENAME USER.

直接修改 grant table，需要下 `FLUSH PRIVILEGES`
> If you modify the grant tables directly using statements such as INSERT, UPDATE, or DELETE (which is not recommended), the changes have no effect on privilege checking until you either tell the server to reload the tables or restart it. Thus, if you change the grant tables directly but forget to reload them, the changes have no effect until you restart the server. This may leave you wondering why your changes seem to make no difference!

參考: https://dev.mysql.com/doc/refman/8.0/en/privilege-changes.html

## debug
* 連線時報錯
  ```
  Client does not support authentication protocol requested by server; consider upgrading MySQL client
  ```
  原因是mysql8更改了密碼加密模式，由mysql_native_password改為caching_sha2_password  
  此案例為java程式要連遠端db，可能是java函式庫太舊
  ``` sql
  ALTER USER 'USER'@'IP' IDENTIFIED WITH mysql_native_password BY 'PASSWORD';
  FLUSH PRIVILEGES;
  ```
  或者創建時直接下`CREATE USER 'USER'@'IP' IDENTIFIED WITH mysql_native_password BY 'PASSWORD'`


