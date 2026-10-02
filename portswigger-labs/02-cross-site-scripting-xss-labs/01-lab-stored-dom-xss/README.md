# Lab: Stored DOM XSS

https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored

# الفرق الأساسي بسرعة

- **Stored XSS** → الكود بيتخزن في السيرفر (DB)
- **DOM XSS** → الكود بيتنفذ بسبب JavaScript في المتصفح (client-side)

# طب إيه هو Stored DOM XSS؟

هو ببساطة **دمج بين الاتنين** 👇

> الكود الخبيث بيتخزن (Stored)
> 
> 
> لكن التنفيذ بيحصل من خلال DOM (Client-side JavaScript)
> 
> ```bash
> 1. <><img src=1 onerror=alert(1)>
> ```
>
