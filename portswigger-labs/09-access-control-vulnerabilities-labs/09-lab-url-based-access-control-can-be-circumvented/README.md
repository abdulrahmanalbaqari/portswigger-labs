# Lab : URL - based access control can be circumvented

https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented

- الفكره من الLab ده ان الadmin panel ظاهر في ال front end بس في الback بيدي access denied فاالملوقع كان بيدعم حاجه سمها `X-Original-URL` هي بتعمل اعاده توجيه جوا الserver بس مبتطلبش اي صلاحيات فا انا استخدمتها علشان ابقي admin و اعمل action بيها برحتي

```bash
1. X-Original-Url: /admin

2. GET ?username=carlos HTTP/2
Host: 0aca00f903e4e8db82d1335200a200bf.web-security-academy.net
Cookie: session=4We6QA5FjuiJmMD5micvBD9yfRmFnUc6
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0aca00f903e4e8db82d1335200a200bf.web-security-academy.net/login
X-Original-Url: /admin/delete
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
```

اللاب ده من فئة **Broken Access Control**، وتحديدًا **URL-based access control bypass** باستخدام هيدر خاص اسمه `X-Original-URL`.

فكرة الموقع هنا معمول من طبقتين:

1. **Front-end system** (زي reverse proxy أو web server عامل فلترة) — بيمنع أي حد من الوصول المباشر لمسار `/admin` من الخارج.
2. **Back-end application** — الأدمن بانل فيه أصلاً بدون أي auth (unauthenticated)، يعني لو قدرت توصله مباشرة، هتقدر تستخدمه بحرية.

المشكلة إن الـ back-end framework (زي بعض إصدارات .NET أو Spring القديمة) بيدعم هيدر اسمه **`X-Original-URL`**، والهدف الأصلي منه إنه يُستخدم داخليًا (مثلاً في إعادة التوجيه الداخلي/rewriting)، لكن المشكلة إن الـ **front-end مش بيتحقق من الهيدر ده**، وبس بيفحص الـ **URL الظاهر في الـ request line**. أما الـ **back-end فبيثق في الهيدر ده** ويستخدمه بدل الـ path الأصلي عند تحديد أي route هيتنفذ.

يعني ببساطة: الـ front-end شايف إنك طالب `/` (مسار عادي مسموح)، لكن الـ back-end فعليًا بيعامل الطلب على إنه طلب للمسار المكتوب في الهيدر (`/admin`).

### خطوات الحل بالتفصيل

#### 1. تأكيد وجود الحماية على مستوى الـ front-end

جرب تفتح:

```
GET /admin HTTP/1.1
```

هتلاقي إنك بتتمنع (Forbidden / Not Found)، ولاحظ إن شكل صفحة الخطأ **بسيطة جدًا** (plain)، مش فيها تصميم الموقع العادي — ده مؤشر إن الرفض ده جاي من **front-end system** (زي load balancer أو reverse proxy) مش من التطبيق نفسه.

#### 2. اكتشاف إن الـ back-end بيقرا من `X-Original-URL`

ابعت الريكوست على **Burp Repeater**، وغيّره كده:

```
GET / HTTP/1.1
X-Original-URL: /invalid
```

لاحظ إن الـ response رجع **"Not Found"** بدل الصفحة الرئيسية العادية بتاعة `/`. ده معناه إن الـ **back-end فعليًا بيتجاهل الـ path في الـ request line (`/`)** وبيستخدم القيمة الموجودة في هيدر `X-Original-URL` عشان يحدد أنهي route ينفذ.

#### 3. الوصول للأدمن بانل

غيّر قيمة الهيدر لـ:

```
GET / HTTP/1.1
X-Original-URL: /admin
```

- الـ **front-end** شايف الطلب على إنه لـ `/` (مسار عادي مسموح بيه) → بيسمح بمرور الريكوست.
- الـ **back-end** بيقرا الهيدر ويشوف إن المطلوب فعليًا هو `/admin` → بيرجعلك صفحة الأدمن بانل كاملة.

#### 4. حذف المستخدم carlos

دلوقتي انت عندك وصول لصفحة الأدمن، والخطوة الطبيعية فيها (زي باقي لابات access control المشابهة) هي إن فيه endpoint بيحذف يوزر معين، شكله:

```
GET /admin/delete?username=carlos
```

فتعمل الريكوست بالشكل ده:

```
GET /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

يعني:

- الـ **query string** (`?username=carlos`) بتتبعت عادي في الـ request line الحقيقي.
- الـ **path** الفعلي المطلوب تنفيذه (`/admin/delete`) بيتحط في هيدر `X-Original-URL`.

ابعت الريكوست ده، ولو نجح هيرجع رسالة تأكيد إن `carlos` اتحذف، واللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

```
[Client] → [Front-end: يفحص الـ path في الـ request line بس] → [Back-end: بيستخدم X-Original-URL لو موجود بدل الـ path]
```

المشكلة إن فيه **عدم اتساق (inconsistency)** بين الطبقتين في تفسير "إيه هو الـ path المطلوب فعليًا":

- الـ front-end (نقطة الحماية) بتحكم على أساس الـ **request line** بس.
- الـ back-end (نقطة التنفيذ) بتحكم على أساس **هيدر مختلف** ممكن يتغير بحرية من المستخدم.

أي فحص أمني (access control) لازم يحصل على **نفس القيمة اللي هيتم التنفيذ بناءً عليها فعليًا**، مش على قيمة تانية ممكن تتلاعب بيها.

### الدرس المستفاد

- **الاعتماد على الـ front-end وحده لفرض الـ access control خطأ جسيم**، خصوصًا لو الـ back-end نفسه معندهوش أي طبقة تحقق مستقلة (زي هنا: صفحة الأدمن أصلاً unauthenticated بالكامل).
- هيدرز زي `X-Original-URL`، `X-Rewrite-URL`، `X-Forwarded-For`, إلخ هي هيدرز بيتحكم فيها **العميل بالكامل**، فمينفعش أي طبقة تثق فيها من غير تحقق أو تعقيم.
- المبدأ الصح: **defense in depth** — يعني الـ back-end نفسه لازم يكون عنده access control خاص بيه (auth + authorization check على مستوى الكود)، بدل ما يعتمد بالكامل على إن الشبكة أو الـ proxy هيحميه.
- لو فيه أكتر من مكون في المعمارية (front-end + back-end)، لازم الاتنين يتفقوا على **نفس تفسير الطلب**، وإلا بيبقى فيه فجوة (bypass surface) زي دي.
