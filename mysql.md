###start mysql
```
mysql.server start
```
###change root password
```
kill all mysql processes;
start mysql using command:

mysqld --skip-grant-tables &

login mysql 

mysql -u root

## mysql -V --->5.7
mysql>update MySQL.user set authentication_string=PASSWORD('新密码') where User='root';  

other mysql version:
mysql>update MySQL.user set password=PASSWORD('新密码') where User='root'; 


mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
    ->     WITH GRANT OPTION;





