# Lab : Reflected XSS protected by CSP , with CSP bypass

https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-bypass

- الفكره من الLab ده اعمل bypass the CSP واخلي الموقع يعمل alert

```bash
1. https://YOUR-LAB-ID.web-security-academy.net/?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'

2. ده payload حل الlab الفكره انCSP كان دايما بينتهي ب ; بس المرادي منتهي ب element من غير مينتهي ب; فاانا هجرب احقن اة اعدل فيه واضيف الي انا عاوزه ونفعت 

3. ولاكن الاني كررت ال script في الاول فا هيلغو بعض فا هطر ادور علي script تانيه علي موقع CSP directive علشان ميحصلش لغبته 
```
