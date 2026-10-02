# Lab : Multi - step process with no access control on one step

https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step

- الفكره من الLab ده ان التحقق من الrole بيبقا اكتر من step فا هتضاف parameter confirmed و المرادي مفيش تامين علي ,method ال POST

```bash
1. POST /admin-roles?username=wiener HTTP/2
Host: 0aa100f903f57afe82aa03b1003e0035.web-security-academy.net
Cookie: session=p0wY63vMdBcXnxNxqxAx6FDRUKyVoaXa
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0aa100f903f57afe82aa03b1003e0035.web-security-academy.net/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

action=upgrade&confirmed=true&username=wiener
```

اللاب ده كمان من فئة **Broken Access Control**، بس المشكلة هنا مختلفة عن اللابات اللي فاتت: العملية (ترقية مستخدم لـ admin) مش بتحصل في **request واحد**، لكن في **عملية متعددة الخطوات (multi-step process)** — يعني فيها أكتر من request متتابعة عشان تكتمل العملية.

المشكلة إن المطور طبّق فحص الصلاحيات (access control check) على **الخطوة الأولى بس** من العملية، وافترض إن لو الخطوة الأولى محمية، يبقى العملية كلها آمنة تلقائيًا. لكن الخطوات اللي بعدها (زي خطوة التأكيد النهائية) **مفيهاش أي فحص صلاحيات مستقل**.

### سيناريو العملية (Multi-step) في اللاب ده

العملية غالبًا بتكون شكلها كده:

1. **Step 1**: الأدمن بيدوس على "Promote to admin" بجانب اسم مستخدم → الطلب ده بيتحقق من صلاحية الأدمن.
2. **Step 2 (Confirmation)**: بيظهر صفحة تأكيد ("Are you sure you want to promote X?") مع زرار "Confirm" → الريكوست بتاع الخطوة دي **مفيهوش فحص صلاحيات** خالص، لأن المطور افتكر إن "بما إنه وصل للخطوة دي أصلاً يبقى هو أدمن أكيد".

### خطوات الحل بالتفصيل

#### 1. سجل دخول كأدمن وافهم العملية

```
administrator:admin
```

روح لـ **Admin panel**، ودوس على **Promote** بجانب `carlos`. هتلاقي إن الموقع بيوريك **صفحة تأكيد** (confirmation step) قبل ما العملية تتم فعليًا — يعني فيه خطوتين مش خطوة واحدة.

#### 2. اعترض خطوة التأكيد

اضغط على زرار **Confirm** في صفحة التأكيد، وابعت الريكوست ده (بتاع التأكيد نفسه، مش أول ريكوست) على **Burp Repeater**. هيكون شكله تقريبًا:

```
POST /admin/roles/confirm HTTP/1.1
Cookie: session=<admin-session>

csrf=...&username=carlos&action=upgrade
```

#### 3. سجل دخول بحساب عادي في نافذة تانية

افتح **Incognito window** وسجل دخول بـ:

```
wiener:peter
```

انسخ الـ **session cookie** بتاع `wiener`.

#### 4. استبدل الـ session وغيّر اليوزرنيم

في الريكوست اللي في Repeater (بتاع خطوة **التأكيد** بس، مش الخطوة الأولى):

- استبدل الـ `Cookie: session=` بالـ session بتاع `wiener`.
- غيّر قيمة `username=carlos` لـ `username=wiener` (أو اسمك انت).

يبقى الريكوست النهائي:

```
POST /admin/roles/confirm HTTP/1.1
Cookie: session=<wiener-session>

csrf=...&username=wiener&action=upgrade
```

#### 5. ابعت الريكوست

لو الثغرة موجودة، هيتنفذ من غير أي مشكلة، لأن **خطوة التأكيد دي معندهاش فحص صلاحيات مستقل** — الفحص كان بس على أول خطوة (اللي هي عرض صفحة التأكيد نفسها، مش تنفيذ العملية).

#### 6. تأكيد الحل

راجع حسابك، لو بقيت admin، اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون مقسّم شبه كده:

python

```python
@app.route("/admin/roles", methods=["GET"])def show_confirmation():    if not current_user.is_admin:        return "Unauthorized", 401    return render_confirmation_page()   # ← فحص الصلاحية هنا بس@app.route("/admin/roles/confirm", methods=["POST"])def confirm_promotion():    # ❌ مفيش أي فحص لـ current_user.is_admin هنا!    username = request.form.get("username")    promote_user(username)    return "User promoted successfully"
```

المشكلة واضحة: المطور حط الفحص في نقطة **عرض** صفحة التأكيد بس، وافترض غلط إن أي حد وصل لخطوة الـ **تنفيذ الفعلي** (`/confirm`) لازم يكون عدى الفحص الأول بالضرورة. لكن بما إن الخطوتين دول **endpoints منفصلتين تمامًا**، أي حد يقدر ينادي على `/admin/roles/confirm` مباشرة من غير ما يمر بالـ endpoint الأول خالص.

### الدرس المستفاد

- في أي **عملية متعددة الخطوات (multi-step workflow)**، لازم **كل خطوة** (مش الخطوة الأولى بس) يكون فيها **فحص صلاحيات مستقل بذاته**، لأن كل خطوة هي فعليًا endpoint منفصل يمكن الوصول له مباشرة بغض النظر عن الترتيب المفترض في الـ UI.
- الافتراض إن "المستخدم لازم يكون مر بالخطوات اللي قبل كده" هو افتراض خاطئ خطير، لأن الـ UI بس اللي بيفرض الترتيب ده، مش الـ backend — وأي حد يقدر يتخطى الـ UI ويبعت الريكوست مباشرة (زي إحنا فعلنا بـ Burp).
- **مبدأ عام في الأمان**: افحص الصلاحيات على مستوى **كل endpoint بيغيّر حالة (state-changing operation)**، مش بس على نقطة الدخول للعملية.
- ده نفس فكرة "**Defense in depth**" اللي اتكررت في اللابات اللي فاتت — كل نقطة تنفيذ فعلي محتاجة حراستها الخاصة، مش الاعتماد على إن نقطة سابقة في نفس الـ flow كانت محمية.
