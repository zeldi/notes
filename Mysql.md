## Check mysql database collation and charset
```
mysql -ne "SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'mysql'"
```

## Tunning 

http://www.tecmint.com/install-secure-performance-tuning-mariadb-database-server/
