
* ### Connect to mysql database
```bash
mysql -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name
```

***

* ### Dump mysql database
```bash
mysqldump -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name > dump.sql
```

***

* ### Restore mysql database from dump
```bash
mysql -Smysql_socket -hmysql_host -umysql_user -pmyssql_password database_name < dump.sql
```

* ### Swap MySQL tables
```sql
RENAME TABLE operations_deposits TO tmp_operations_deposits
             __operations_deposits TO operations_deposits
             tmp_operations_deposits TO __operations_deposits;
```
