# Lab : Bypass rate limits via conditions

https://portswigger.net/web-security/race-conditions/lab-race-conditions-bypassing-rate-limits

- الفكره من الLab ده

```bash
1. def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )
    
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard
    
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')

def handleResponse(req, interesting):
    table.add(req)
```

### الفكرة العامة (نوع الثغرة)

اللاب ده من فئة **Race Conditions**، تحديدًا استغلال **فجوة زمنية بين "تسجيل محاولة الدخول" و"زيادة عداد المحاولات الفاشلة"** في نظام الـ rate limiting.

### طبيعة الثغرة

النظام بيمنعك من تسجيل الدخول بعد **3 محاولات فاشلة** على نفس اليوزرنيم. لكن العداد ده **بيتسجل على السيرفر بعد** كل محاولة، مش **أثناء** معالجتها. يعني فيه **فجوة زمنية دقيقة** بين لحظة استلام الطلب ولحظة تحديث العداد.

لو بعتّ **عدد كبير من الطلبات في نفس اللحظة بالظبط** (مش بالتتابع العادي)، كل الطلبات دي **هتوصل للسيرفر قبل ما العداد يتحدّث من أي طلب سابق منها** — يعني السيرفر هيعالج **كل الطلبات دي على أساس إن العداد لسه صفر**، وبالتالي **هتعدي الحد الأقصى (3 محاولات) بسهولة**.

### خطوات الحل بالتفصيل (باستخدام Repeater Groups قدر الإمكان + شرح ليه محتاجين Turbo Intruder في الآخر)

#### المرحلة 1: اكتشاف السلوك الأساسي (Predict a potential collision)

1. جرب تدخل بباسورد غلط لحسابك (`wiener`) أكتر من 3 مرات — هتلاحظ إنك بتتحظر مؤقتًا.
2. جرب يوزر تاني عشوائي — هتاخد رسالة "Invalid username or password" عادية، مش رسالة حظر. ده معناه إن **الحظر مرتبط باليوزرنيم نفسه (per-username)، مش بالـ session أو الـ IP**.
3. الاستنتاج: عداد المحاولات الفاشلة **بيتخزن على السيرفر لكل يوزرنيم لوحده**، وفيه احتمال **فجوة زمنية (race window)** بين إرسال الطلب وتحديث العداد.

#### المرحلة 2: قياس السلوك (Benchmark) باستخدام Repeater Groups

1. خد طلب `POST /login` (بباسورد غلط لحسابك)، وابعته على **Burp Repeater**.
2. ضيف التاب دي لمجموعة (Group).
3. كرر التاب (**Duplicate tab**) لحد ما يبقى عندك **20 تاب** في المجموعة.
4. ابعت المجموعة **بالتتابع (in sequence) باستخدام اتصالات منفصلة** — عشان تقلل احتمالية التداخل، وتشوف السلوك "الطبيعي".
5. لاحظ إنك بعد **محاولتين إضافيتين بس (يعني المحاولة التالتة)** بتتحظر — يعني بالطريقة العادية، النظام بيشتغل صح.

#### المرحلة 3: البحث عن دليل (Probe for clues) — هنا الفرق يبان

1. ابعت **نفس المجموعة العشرين**، لكن المرة دي **بالتوازي (in parallel)** بدل التتابع.
2. ادرس الردود: هتلاحظ إنه رغم إن الحظر اتفعّل، **أكتر من 3 طلبات** رجعوا رد "Invalid username or password" **العادي** (مش رسالة حظر) — يعني عدّت محاولات أكتر من الحد المسموح قبل ما الحظر "يلحق" يتفعل.
3. الاستنتاج: **لو بعتّ الطلبات بسرعة كافية ومتزامنة، تقدر تعدّي أكتر من 3 محاولات قبل ما العداد يتحدث.**

#### المرحلة 4: إثبات المفهوم والاستغلال الفعلي (هنا لازم Turbo Intruder)

للأسف، هنا **الطريقة اليدوية بـ Repeater مش كافية للاستغلال الفعلي**، والسبب:

- إحنا محتاجين **نبعت كل الباسوردات المرشحة (30 باسورد) دفعة واحدة بالظبط في نفس اللحظة**، مش بس نكرر نفس الطلب. ده معناه إننا محتاجين **payload مختلف لكل طلب** (باسورد مختلف)، **لكن كلهم يوصلوا في نفس اللحظة الدقيقة**.
- **Repeater Groups بتدّيك تحكم في التوقيت، لكن مش بتدّيك QUEUE ديناميكي بمنطق برمجي** (زي "جرب كل باسورد من القائمة دي، ولكن كلهم في نفس اللحظة"). ده محتاج **سكريبت** — وهنا بالظبط دور **Turbo Intruder**، خصوصًا مع تقنية **"Single-packet attack"** اللي بتستغل HTTP/2 عشان تضمن وصول كل الطلبات في **حزمة شبكة واحدة بالظبط (single TCP packet)** — وده دقة **مستحيل تحققها يدويًا حتى بأفضل استخدام لـ Repeater Groups**، لأن Repeater بيبعت كل طلب كـ HTTP request منفصل فعليًا، مش بيتحكم في مستوى الـ packets الخام.

#### لماذا نحتاج بالتحديد Turbo Intruder هنا:

الـ **Single-packet attack** technique (المستخدمة في `race-single-packet-attack.py`) بتشتغل عن طريق:

1. تجهيز **كل الطلبات (بباسوردات مختلفة) بالكامل** مسبقًا.
2. حجز **آخر جزء بايت واحد بس** من كل طلب (الـ "gate").
3. إرسال **كل الطلبات تقريبًا كاملة** على الشبكة، ومنع وصولها الكامل للسيرفر.
4. إرسال **آخر بايت من كل الطلبات في نفس اللحظة بالضبط**، فيوصلوا للسيرفر **في نفس اللحظة تمامًا** (مش حتى مللي ثانية فرق).

ده مستوى من التزامن **مش ممكن تحقيقه من واجهة Repeater العادية**، حتى مع "Send group in parallel"، لأن الأخيرة بتبعت الطلبات **كاملة** بالتوازي، لكن **بدون** التحكم في مستوى الـ TCP packet نفسه.

#### الخطوات الفعلية (Turbo Intruder):

1. في Repeater، حدد قيمة الـ `password` parameter في الطلب، كليك يمين → **Extensions > Turbo Intruder > Send to turbo intruder**.
2. غيّر الـ `username` لـ `carlos`.
3. اختار القالب الجاهز:

```
    examples/race-single-packet-attack.py
```

1. عدّل السكريبت عشان يبعت طلب واحد لكل باسورد من الحافظة (clipboard):

python

```python
def queueRequests(target, wordlists):    engine = RequestEngine(endpoint=target.endpoint,                           concurrentConnections=1,                           engine=Engine.BURP2)  # HTTP/2 مطلوب للـ single-packet attack    passwords = wordlists.clipboard    for password in passwords:        engine.queue(target.req, password, gate='1')    engine.openGate('1')  # بعت كل الطلبات في نفس اللحظةdef handleResponse(req, interesting):    table.add(req)
```

1. **انسخ قائمة الباسوردات المرشحة** للحافظة (clipboard) قبل ما تشغّل الهجوم.
2. دوس **Launch attack**.
3. افحص النتائج:
    - لو مفيش `302` (نجاح)، استنى انتهاء فترة الحظر وكرر المحاولة (يمكن تشيل الباسوردات المؤكد فشلها من القائمة).
    - لو لقيت رد **`302`**، ده معناه تسجيل دخول ناجح — سجّل الباسورد المقابل من عمود الـ Payload.
4. استنى انتهاء فترة الحظر، وسجل دخول فعليًا بـ `carlos` والباسورد المكتشف.
5. روح للأدمن بانل واحذف `carlos`. اللاب هيتحل تلقائيًا ✅.

### ليه Repeater Groups عمومًا مفيدة هنا لكن مش كافية للاستغلال الكامل

| الاستخدام | Repeater Groups | Turbo Intruder (Single-packet) |
| --- | --- | --- |
| اكتشاف وجود الـ race condition (المرحلة 2-3) | ✅ كافية تمامًا | غير ضروري |
| إثبات المفهوم بتجربة عدة محاولات متزامنة تقريبًا | ✅ ممكن تستخدمها بنجاح جزئي | أدق بكتير |
| استغلال فعلي يحتاج تجربة 30 باسورد **في نفس اللحظة بالظبط** لتفادي إهدار محاولاتك المحدودة | ❌ صعب/غير عملي (هتحتاج تجهز 30 تاب يدويًا بباسوردات مختلفة) | ✅ الأداة المصممة لده بالظبط |

### ليه ده بيحصل تقنيًا

python

```python
failed_attempts = {}  # per usernamedef login(username, password):    if failed_attempts.get(username, 0) >= 3:        return "Account locked", 429    if not check_password(username, password):        failed_attempts[username] = failed_attempts.get(username, 0) + 1   # ❌ تحديث غير atomic        return "Invalid username or password", 401    return "Success", 302
```

المشكلة الجوهرية: **العملية "افحص العداد → افحص الباسورد → حدّث العداد" مش atomic** (يعني مش عملية واحدة غير قابلة للتقسيم). لو **عدة threads/requests وصلوا في نفس اللحظة**، كلهم بيقرأوا **نفس قيمة العداد القديمة** (قبل ما أي واحد فيهم يحدّثها)، فكلهم بيعدّوا "المحاولة رقم كذا" بينما فعليًا في المجموع تجاوزوا الحد المسموح بكتير.

### الدرس المستفاد

- **أي rate-limiting أو counter مبني على "اقرأ ثم حدّث (read-then-write)" عرضة لـ race conditions** لو العملية مش **atomic** بالكامل. الحل الصحيح: استخدام **database-level locking** أو **atomic increment operations** (زي `INCR` في Redis) تضمن إن القراءة والتحديث يحصلوا كوحدة واحدة غير قابلة للمقاطعة.
- **HTTP/2 خلق فرصة جديدة لهجمات race condition أدق (single-packet attacks)** — لأن بروتوكول HTTP/2 بيسمح بإرسال أجزاء من طلبات متعددة عبر نفس الاتصال وبتحكم دقيق جدًا في التوقيت، عكس HTTP/1.1 اللي فيه latency طبيعي بين الطلبات بيصعّب هجمات التزامن الدقيقة.
- **Burp Repeater Groups ممتازة للاكتشاف والتشخيص** (زي ما استخدمناها في مراحل الـ benchmark والـ probing)، لكن **Turbo Intruder ضروري للاستغلال الدقيق** اللي محتاج تزامن على مستوى الـ packet، خصوصًا لما يكون عندك **قائمة طويلة من الـ payloads المختلفة** ومحتاج تبعتهم كلهم بتزامن شبه مثالي.
- **Note مهمة من اللاب نفسه**: بما إن كل محاولة لازم تستنى انتهاء فترة الحظر لو فشلت، **الوقت محدود (15 دقيقة)** — فالكفاءة في الاستغلال (single-packet attack بتجرب كل الباسوردات دفعة واحدة) مهمة جدًا عمليًا، مش بس نظريًا.
