# Lab : Excessive trust in client - side controls

https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls

- الفكره من الLab ده ان الموقع بيtrust اي request يتبعت حتي بعد معدلته يعني في الrequest لاساسي الي اتبعت من صفحع الproduct ل cart عدلت في الprice والسعر اتقبل عادي جدا

```bash
1. productId=1&redir=PRODUCT&quantity=1&price=133700 ==> price=1
```
