# Lab : Information disclosure on debug page

https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page

- الفكره من الLab ده ان الdevelop نسي صفحه حسايه فيها بيانات حساسه من غير تامين كويس ونسي الرابط بتاعها كمان في back ground page

```bash
1. <!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->

2. https://0a7a0004048e33e8804c49e500c100ee.web-security-academy.net/cgi-bin/phpinfo.php
```
