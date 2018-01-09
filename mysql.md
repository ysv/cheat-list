
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
