# Lab : OS command injection , simple case

https://portswigger.net/web-security/os-command-injection/lab-simple

- الفكره من ال Lab ده انك بتختبر الموقع هل ينفع تنفز os command injection تخليني a access in the server

```bash
1. productId=1&storeId=3;whoami
```

### الفكرة الأساسية

اللاب ده من PortSwigger Web Security Academy، وبيوضح إزاي التطبيق لما بياخد مدخل من اليوزر ويمرره لأمر بيتنفذ على نظام التشغيل (زي shell command) من غير ما ينضف أو يتأكد من المدخل ده، بيبقى فيه ثغرة إن اليوزر يقدر يحقن أوامر إضافية جنب الأمر الأصلي.

### سيناريو اللاب

- بيكون فيه متجر إلكتروني (زي معظم لابات PortSwigger)، وفيه صفحة منتج فيها خاصية "Check stock" بتاخد الـ product ID والـ store location.
- في الباك اند، التطبيق بيبني أمر shell (زي `stockcheck.pl` أو حاجة شبهها) وبيدمج فيه القيمة اللي جاية من اليوزر مباشرة من غير فلترة.

### إزاي بتحلها

1. تدخل على صفحة أي منتج وتضغط "Check stock".
2. تفتح Burp Suite وتعمل Intercept للـ request بتاع الـ stock check.
3. تلاقي فيه parameter اسمه `storeId` (أو مشابه) بييجي كـ query string أو form field.
4. تحط قيمة injection بسيطة بعد القيمة الأصلية، زي:

```
storeId=1|whoami
```

أو

```
storeId=1;whoami
```

أو ممكن تستخدم `||` أو backtick حسب الشكل.

5. لو التطبيق ضعيف (زي اللاب)، هتلاقي نتيجة الأمر (زي اسم اليوزر اللي التطبيق شغال بيه) ظاهرة في الـ response — غالبًا في مكان زي "stock check failed" مع الناتج مطبوع جنبه.

6. اللاب بيعتبر "محلول" لما الـ output بتاع الأمر (زي `whoami`) يظهر في الرد.

#### أشهر الـ separators المستخدمة في الحقن:

| Separator | المعنى |
| --- | --- |
| `;` | ينفذ الأمر التاني بعد الأول (على Linux) |
| `|` | بياخد output الأمر الأول ويديه كـ input للتاني (لو الأول مالوش output بيتنفذ التاني عادي) |
| `||` | ينفذ الأمر التاني لو الأول فشل |
| `&&` | ينفذ الأمر التاني لو الأول نجح |
| ``` أو `$()` | command substitution — بيتنفذ جوه أمر تاني |

### الاستفادة منه (ليه مهم تتعلمه)

1. **فهم إزاي الثغرة بتحصل من الأساس**: التطبيقات كتير بتستخدم مكتبات أو سكريبتات خارجية (legacy code، أدوات نظام) وبتنده عليها من خلال shell، وده بيفتح الباب للثغرة دي لو مفيش تعقيم للمدخلات.
2. **فهم الفرق بين الأنواع**:
    - Command injection بسيط زي ده بيديك output مباشر (in-band).
    - في حالات تانية (blind command injection) مش هتشوف output، فهتحتاج تقنيات زي:
        - استخدام `ping` أو `sleep` وتقيس الوقت (time-based).
        - تخرّج نتيجة الأمر لملف أو تبعتها لسيرفر بتتحكم فيه إنت (out-of-band، زي DNS/HTTP exfiltration).
