# MySQL Cheatsheet

log into MySQL: `mysql -u <target> -p`

- `SHOW DATABASES;` = list all databases.
- `USE database_name;` = work inside a specific database.
- `SHOW TABLES;` = see all tables inside a database.
- `DESCRIBE table_name;` or `SHOW COLUMNS FROM table_name;` = see the columns of a specific table.
- `SELECT * FROM table_name;` = display all data from a table.
- `SELECT * FROM table_name LIMIT 10;` = limit number of rows.
- `SELECT column_name FROM table_name;` = see all value inside a praticular column.
- `SELECT column_name FROM table_name WHERE condition;` = see data only for specific condition.
