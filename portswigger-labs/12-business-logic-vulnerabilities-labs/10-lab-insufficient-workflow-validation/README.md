# Lab : Insufficient workflow validation

https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-insufficient-workflow-validation

- الفكره من الLab ده ان في miss في workflow validation يعني في مشكله في تسلسل العمليات ذي اني ممكن a skip  ال checkout

```bash
1. GET /cart/order-confirmation?order-confirmed=true

2. Study the proxy history. Observe that when you place an order, the POST /cart/checkout request redirects you to an order confirmation page. Send GET /cart/order-confirmation?order-confirmation=true to Burp Repeater.
```
