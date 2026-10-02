# Lab : SSRF via flawed request paring

https://portswigger.net/web-security/host-header/exploiting/lab-host-header-ssrf-via-flawed-request-parsing

- الفكره من الLab ده

```bash
1. GET https://0a9600d803bca5e381bc0c1f00bb00d7.web-security-academy.net/admin HTTP/2
Host: 192.168.0.202

2. POST https://0a9600d803bca5e381bc0c1f00bb00d7.web-security-academy.net/admin/delete HTTP/2
Host: 192.168.0.202
csrf=jlRBUXvMqTvhZpP2O1BgGuWHy392AaBh&username=carlos
```

اللاب ده **نسخة متقدمة** من اللاب السابق (Routing-based SSRF)، لكن هنا الموقع **أذكى شوية** — بيتحقق فعليًا من هيدر `Host` ومش بيسمح بتعديله مباشرة. المشكلة هنا مختلفة: **تناقض بين مصدرين مختلفين للـ "Host" في نفس الريكوست** — الـ **`Host` header** العادي، مقابل الـ **Absolute URL** المكتوب في سطر الطلب نفسه (Request line).

### الفرق الجوهري عن اللاب السابق

|  | **اللاب السابق (Routing-based)** | **اللاب ده (Flawed parsing)** |
| --- | --- | --- |
| **آلية الحماية** | مفيش فحص على Host header خالص | **فيه فحص فعلي** على Host header، وبيرفض أي تعديل عليه |
| **نقطة الاستغلال** | تعديل Host header مباشرة | استخدام **Absolute URL في الـ request line** بدل الاعتماد على Host header |

### طبيعة الثغرة

في HTTP، فيه طريقتين لكتابة الـ request line:

```
GET / HTTP/1.1              ← الشكل العادي (relative path)
Host: example.com
```

أو:

```
GET https://example.com/ HTTP/1.1    ← absolute URL (شكل غير شائع لكنه قياسي وصحيح)
Host: example.com
```

الشكل الثاني (Absolute URL في سطر الطلب) **نادر الاستخدام من المتصفحات العادية** (بيُستخدم غالبًا لما تكلم proxy مباشرة)، لكنه **صحيح تمامًا حسب معيار HTTP**.

المشكلة هنا: الموقع بيتحقق من **الـ Host header** بس عادةً، لكن لما تبعت **absolute URL في الـ request line**، السيرفر بيستخدم **الجزء الخاص بالدومين في الـ URL ده** (مش الـ Host header) عشان يحدد **لأي سيرفر داخلي يوجّه الطلب**. يعني فيه **مصدرين مختلفين للمعلومة نفسها (الـ host)**، والتحقق (validation) بيحصل على واحد بس، بينما الـ **routing الفعلي** بيعتمد على التاني.

### خطوات الحل بالتفصيل

#### 1. تأكد من سلوك الحماية الأساسي

ابعت `GET /` (اللي بيرجع 200) على **Burp Repeater**. جرب تعدّل قيمة الـ **`Host` header** مباشرة — هتلاقي الطلب **بيترفض** فورًا. يبقى تأكدنا إن فيه فحص صارم على الـ Host header العادي.

#### 2. اكتشف طريقة بديلة: Absolute URL في الـ Request Line

جرب تغيّر شكل سطر الطلب نفسه لصيغة **absolute URL**:

```
GET https://YOUR-LAB-ID.web-security-academy.net/ HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
```

هتلاحظ إن الصفحة الرئيسية **لسه شغالة عادي**.

#### 3. اختبر التعديل على الـ Host header **مع وجود absolute URL**

دلوقتي جرب تعدّل قيمة الـ **`Host` header** (مش الـ absolute URL)، مع إبقاء الـ absolute URL كما هو:

```
GET https://YOUR-LAB-ID.web-security-academy.net/ HTTP/1.1
Host: modified-value
```

هتلاحظ حاجة غريبة: **الطلب مبيترفضش فورًا زي قبل كده** — بدل كده، بترجعلك **رسالة timeout**. ده مؤشر قوي جدًا على إن:

- **الفحص الأمني (validation)** بقى بيحصل على **الـ Host header** لسه (وممكن يكون عدى لأنه غير مطابق لأي whitelist بشكل مختلف، أو الفحص بقى بيبص على حاجة تانية).
- لكن **الـ routing الفعلي** بقى بيعتمد على **الـ absolute URL** في سطر الطلب، مش على الـ Host header — وده اللي بيفسر الـ timeout (لأن السيرفر بيحاول يوصل لمكان مبني من الـ absolute URL، مش من الـ Host المعدّل).

#### 4. تأكيد الثغرة باستخدام Burp Collaborator

عشان نتأكد بشكل قاطع إن السيرفر بيعمل طلبات خارجية بناءً على الـ **absolute URL**، نستخدم تقنية مشابهة للاب السابق لكن بتلاعب في **الـ Host الموجود داخل الـ absolute URL نفسه** (مش هيدر Host العادي):

```
GET https://YOUR-LAB-ID.web-security-academy.net/ HTTP/1.1
Host: BURP-COLLABORATOR-SUBDOMAIN
```

استخدم **"Insert Collaborator payload"** في المكان المناسب، وابعت الطلب. روح لتبويب **Collaborator** ودوس **Poll now** — لو شفت تفاعلات HTTP، يبقى تأكدنا إن السيرفر فعلاً بيعمل طلب خارجي للدومين اللي حطيناه.

#### 5. امسح الشبكة الداخلية بنفس طريقة Burp Intruder

ابعت الريكوست (بصيغة absolute URL) على **Burp Intruder**:

1. شيل تفعيل **"Update Host header to match target"**.
2. حط **payload position** على الجزء المطلوب فحصه من الـ Host (سواء في الـ Host header أو جوه الـ absolute URL — حسب أنهي جزء هو اللي بيتحكم فعليًا في الـ routing).
3. استخدم نوع **Numbers** من `0` لـ `255` عشان تمسح range `192.168.0.0/24` بالكامل.
4. شغّل الهجوم ودوّر على أي response مختلف (زي `302` بدل الـ timeout المعتاد).

#### 6. الوصول للوحة الإدارية

لما تلاقي الـ IP الصح (اللي رجع رد مختلف)، ابعته على **Repeater**، وضيف `/admin` للـ **absolute URL**:

```
GET https://192.168.0.X/admin HTTP/1.1
```

(أو حسب الصيغة اللي اتأكدنا إنها الصح — absolute URL أو Host header، على حسب نتيجة التجربة). ابعت الطلب، وهتلاقي نفسك دخلت لصفحة الأدمن.

#### 7. احذف carlos

زي بالظبط اللاب السابق:

1. غيّر الـ absolute URL لـ `/admin/delete`.
2. أضف الـ **CSRF token** كـ query parameter.
3. أضف `username=carlos`.
4. أضف الـ **session cookie** من الـ `Set-Cookie` header.
5. استخدم **"Change request method"** لتحويله لـ `POST`.
6. ابعت الريكوست.

اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

python

```python
def handle_request(request):    # المشكلة: مصدرين مختلفين للـ "host"    host_header = request.headers.get("Host")    absolute_url_host = extract_host_from_request_line(request.request_line)    # الفحص الأمني بيحصل على واحد بس:    if not is_valid_host(host_header):   # ✅ فحص صارم هنا        return "Forbidden", 403    # لكن الـ routing الفعلي بيعتمد على التاني!    target = absolute_url_host if absolute_url_host else host_header   # ❌ منطق مختلف تمامًا    backend_response = internal_network.send_request(target=target)    return backend_response
```

المشكلة الجوهرية: **الكود بيفحص مصدر واحد للمعلومة (الـ Host header)، لكن بيستخدم مصدر مختلف تمامًا (الـ Absolute URL) لاتخاذ القرار الفعلي (الـ routing)**. ده **بالظبط** نفس فكرة الـ **Parser Discrepancy** اللي شرحناها في لاب الإيميل (UTF-7) قبل كده — بس هنا التناقض مش بين مكتبتين، لكن بين **جزئين مختلفين من نفس الريكوست HTTP** (الـ request line مقابل الـ headers).

### الدرس المستفاد

- **HTTP بيدّي أكتر من طريقة لتمثيل نفس المعلومة** (زي الـ Host، سواء في header أو في absolute URL بالـ request line) — أي تطبيق أو middleware لازم **يتعامل مع كل المصادر دي بشكل متسق تمامًا**، ويستخدم **نفس المصدر بالظبط** للفحص الأمني وللـ routing الفعلي، مش مصدرين مختلفين.
- **أي نقطة "غير قياسية" أو "أقل استخدامًا" في بروتوكول HTTP** (زي absolute URL في الـ request line) تستحق دايمًا **الاختبار المنفصل** وقت تحليل ثغرات SSRF أو Host header injection، لأنها غالبًا **مش متغطية بنفس مستوى الفحص** اللي بيتطبق على الحالات الشائعة.
- ده مثال إضافي على فئة **Parser/Validation Discrepancy**، واللي بنشوفها بتتكرر في سياقات مختلفة جدًا (إيميل، HTTP requests، cache handling) — المبدأ العام واحد: **لو فيه أكتر من "نسخة" أو "تفسير" لنفس البيانات جوه النظام، وكل جزء من النظام بيثق في نسخة مختلفة، فده سطح هجوم محتمل**.
- **دايمًا جرب الحالات الحدّية في بروتوكول HTTP نفسه** (absolute URLs, duplicate headers, unusual encodings) مش بس البيانات اللي بيقبلها التطبيق ظاهريًا — كتير من الثغرات المتقدمة بتيجي من فهم عميق للبروتوكول نفسه مش بس منطق التطبيق.
