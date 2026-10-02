# Lab : Referer - based access control

https://portswigger.net/web-security/access-control/lab-referer-based-access-control

- الفكره من الLab ده ان الموقع بيهتم ب referer header انه جاي من صفحه الadmin و مبيتحققش من الصلاحيه اصلا و لاحتي من الي بيتبعت

```bash
1. Referer: https://0a76005103815900803f17b8003d0096.web-security-academy.net/admin

2. GET /admin-roles?username=wiener&action=upgrade HTTP/2
```

اللاب ده كمان من فئة **Broken Access Control**، وتحديدًا مشكلة اسمها **Referer-based access control bypass**.

الفكرة إن التطبيق بيتحقق من صلاحيات الأدمن مش من خلال الـ **session** بس، لكن كمان بيعتمد على هيدر **`Referer`** — وهو هيدر بيُرسل تلقائيًا من المتصفح بيقول "الطلب ده جه من أنهي صفحة". المنطق اللي المطور بناه هو:

> "لو الريكوست جاي من صفحة الأدمن بانل نفسها (يعني `Referer` بيساوي `/admin`)، يبقى غالبًا اليوزر ده أدمن فعلاً، فأسمحله."
> 

المشكلة إن **الهيدر `Referer` بيتحكم فيه العميل بالكامل** — أي حد يقدر يبعته بأي قيمة يحبها، فمش وسيلة موثوقة للتحقق من الهوية أو الصلاحية أبدًا.

### خطوات الحل بالتفصيل

#### 1. سجل دخول كأدمن وافهم العملية

```
administrator:admin
```

روح لـ **Admin panel**، ودوس على **Promote** بجانب `carlos`. الريكوست هيكون شكله تقريبًا:

```
GET /admin-roles?username=carlos&action=upgrade HTTP/1.1
Referer: https://<lab-id>.web-security-academy.net/admin
Cookie: session=<admin-session>
```

ابعت الريكوست ده على **Burp Repeater**.

#### 2. سجل دخول بحساب عادي في نافذة تانية

افتح **Incognito window** وسجل دخول بـ:

```
wiener:peter
```

#### 3. جرب تنادي على الـ endpoint مباشرة من غير Referer

لو جربت تفتح الرابط ده مباشرة في المتصفح (بالكتابة اليدوية في شريط العنوان، مش بالضغط من رابط داخل الصفحة):

```
/admin-roles?username=carlos&action=upgrade
```

هتلاقي إن الطلب **مرفوض (Unauthorized)** — لأن المتصفح لما تكتب رابط يدويًا في الـ address bar، **مبيبعتش هيدر `Referer`** أصلاً. ده بيأكد إن الفحص فعلاً معتمد على وجود/قيمة الـ `Referer` header، مش على صلاحيات حقيقية.

#### 4. استبدل الـ session وابعت من Repeater

ارجع للريكوست اللي في Repeater (اللي فيه الـ Referer الصحيح `/admin` محفوظ فيه بالفعل من وقت ما بعتناه كأدمن). دلوقتي:

- انسخ الـ **session cookie** بتاع `wiener` من الـ incognito window.
- استبدل بيه قيمة الـ `Cookie` في الريكوست.
- غيّر `username=carlos` لـ `username=wiener`.

يبقى الريكوست النهائي:

```
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Referer: https://<lab-id>.web-security-academy.net/admin   ← لسه موجود من أول ريكوست
Cookie: session=<wiener-session>
```

#### 5. ابعت الريكوست

بما إن الـ `Referer` header لسه موجود وقيمته صح (`/admin`)، الفحص هيعدي، وهيتم ترقية `wiener` لـ admin رغم إنه مش أدمن أصلاً.

#### 6. تأكيد الحل

راجع حسابك، لو بقيت admin، اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
@app.route("/admin-roles")def upgrade_role():    referer = request.headers.get("Referer", "")    if "/admin" in referer:   # ❌ فحص ضعيف جدًا ومعتمد بالكامل على هيدر يتحكم فيه العميل        promote_user(request.args.get("username"))        return "Success"    else:        return "Unauthorized", 401
```

المشكلة إن الفحص هنا **مبنيش على هوية المستخدم الحالي (session) وصلاحياته الحقيقية**، لكن على قيمة هيدر HTTP بيتحكم فيها الـ client بالكامل ويقدر يزوّرها بسهولة (سواء بإعادة استخدام ريكوست قديم فيه الـ Referer الصح، زي إحنا عملنا، أو حتى بتعديل الهيدر يدويًا في Burp من الأول).

### الدرس المستفاد

- **هيدر `Referer` مالوش أي قيمة أمنية موثوقة** — زي أي هيدر تاني بيتحكم فيه المتصفح/العميل (زي `X-Original-URL`, `User-Agent`, `X-Forwarded-For`)، ممكن يتغير أو يتشال أو يتزوّر بسهولة تامة.
- استخدام `Referer` في قرارات access control بيبقى **دايمًا bypassable**، لأن أي أداة زي Burp Repeater بتخليك تبعت أي هيدر بأي قيمة تحبها، أو حتى تشيله تمامًا.
- **الحل الصحيح**: الصلاحيات لازم تتحدد بس من خلال **الـ session/token الموثّق بيه المستخدم على السيرفر**، والتحقق من دوره (role) المخزن في قاعدة البيانات، مش من أي هيدر بيبعته المتصفح.
- ده نفس النمط المتكرر في كل لابات الـ access control اللي شرحناها: أي اعتماد على **بيانات جاية من العميل** (هيدر، cookie قيمته سهلة التوقع، أو حتى ترتيب الخطوات في الـ UI) بدل **فحص حقيقي من جانب السيرفر** هيكون دايمًا نقطة ضعف قابلة للاستغلال.
