# Lab : Reflected XSS into a template literal with angle brackets , single , double quotes , backslash and backticks Unicode-escaped

https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped

- الفكره من الLab ده ان اغلي العلامات بتتعملها تشفير او HTML encoded الوحيده الي مش بيتعملها هي backticks

```bash
1. `${alert(1)}`
```
