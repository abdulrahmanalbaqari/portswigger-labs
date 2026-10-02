# Lab : User ID controlled by request parameter

https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter

- الفكره من الLab ده ان ال parameter المساوله عن الid account مصابه فاول محولتها لي carlos الي عاوز اتحكم فيه حولني عليه من غير اي تاكيد وده نوع من ال horizontal privilege escalation

```bash
1. my-account?id=wiener ==> my-account?id=carlos
```
