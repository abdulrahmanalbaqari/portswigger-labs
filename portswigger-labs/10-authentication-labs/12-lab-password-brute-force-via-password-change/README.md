# Lab : Password brute - force via password change

https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change

- الفكره من الLab ده في function ال new password في خانه ال current password  قابله للbrout force فا ممكن استنتج منها password الضحيه بس لو انا عملت مره غلط بيخرجني من الحساب فا الbypass ان اخلي الnew password و الconfirm new password مش متشابهين علشان مياكدش ان الcurrent password غلط

```bash
1. 
```

اللاب ده من فئة **Broken brute-force protection**، لكن هنا الاستغلال بيحصل من خلال endpoint غير متوقع: **صفحة تغيير الباسورد (Change password)**، مش صفحة اللوجن العادية.

### طبيعة الثغرة (الـ Logic Flaw)

فورم تغيير الباسورد فيها 3 حقول:

```
current-password  (الباسورد الحالي)
new-password-1     (الباسورد الجديد)
new-password-2     (تأكيد الباسورد الجديد)
```

بالإضافة لحقل **`username`** مخفي (hidden input) — وده أول نقطة ضعف: **اليوزرنيم بيتبعت كـ parameter تقدر تغيّره**، مش بياخده السيرفر من الـ session بتاعتك.

المشكلة الحقيقية في **رسائل الخطأ المختلفة** حسب الحالة:

| الحالة | current-password | new-password-1 vs new-password-2 | الرسالة |
| --- | --- | --- | --- |
| 1 | غلط | متطابقين | الحساب بيتقفل (lockout) |
| 2 | غلط | مختلفين | `Current password is incorrect` |
| 3 | **صح** | مختلفين | `New passwords do not match` |

لاحظ الفرق بين الحالة 2 والحالة 3: **الرسالة بتختلف بناءً على هل الـ current-password صح ولا لأ** — وده **side channel** بيسرّب معلومة: "هل الباسورد الحالي اللي جربته صح؟" حتى لو العملية نفسها (تغيير الباسورد) فشلت لسبب تاني (عدم تطابق الباسورد الجديد).

يعني تقدر تستخدم حقل **`current-password`** كوسيلة **Oracle** (أداة تكشيف) لتخمين **باسورد أي مستخدم تاني**، من غير ما تتعرض لأي قفل حساب — لأن القفل بيحصل بس لما الباسوردين الجدد يتطابقوا، وإحنا هنخليهم دايمًا مختلفين عشان نتفادي القفل.

### خطوات الحل بالتفصيل

#### 1. افهم سلوك الفورم

سجل دخول بـ `wiener:peter`، وجرب فورم تغيير الباسورد بحالات مختلفة عشان تلاحظ الفرق في الرسائل زي الجدول اللي فوق.

#### 2. جهّز الريكوست الأساسي

ادخل **باسوردك الحالي الصح** (`peter`)، وحط باسوردين جديدين **مختلفين عن بعض** (عشان تتجنب القفل):

```
POST /my-account/change-password HTTP/1.1

username=wiener&current-password=peter&new-password-1=123&new-password-2=abc
```

هتلاقي الرد: `New passwords do not match` — ده تأكيد إن السيرفر وصل لمرحلة فحص تطابق الباسوردين، يعني الباسورد الحالي كان صح.

ابعت الريكوست ده على **Burp Intruder**.

#### 3. جهّز هجوم الـ Intruder

- غيّر قيمة `username` لـ `carlos`.
- حط **payload position** على `current-password` بس.
- خلّي `new-password-1` و `new-password-2` **قيمتين ثابتتين ومختلفتين عن بعض** (زي `123` و `abc`)، عشان تضمن إنك مش هتقفل الحساب أبدًا مهما جربت.

الريكوست في Intruder هيبقى شكله:

```
username=carlos&current-password=§incorrect-password§&new-password-1=123&new-password-2=abc
```

#### 4. حط قائمة الباسوردات كـ Payload

في **Payloads panel**، حط **قائمة candidate passwords** كاملة كـ payload set على الـ position الوحيد.

#### 5. اعمل Grep - Match على الرسالة المميزة

في **Settings panel**، ضيف **Grep - Match rule** يدور على النص:

```
New passwords do not match
```

#### 6. شغّل الهجوم

لما يخلص، فلتر على النتائج اللي فيها الـ match بتاع الرسالة دي. هتلاقي **response واحد بس** فيه الرسالة دي — القيمة المقابلة له في عمود الـ payload هي **الباسورد الصحيح بتاع carlos**.

#### 7. سجل دخول بحساب carlos

```
username: carlos
password: <الباسورد اللي لقيته>
```

#### 8. افتح My Account

اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
@app.route("/my-account/change-password", methods=["POST"])def change_password():    username = request.form.get("username")   # ❌ بيتاخد من الفورم مش من الـ session    current_password = request.form.get("current-password")    new1 = request.form.get("new-password-1")    new2 = request.form.get("new-password-2")    if not check_password(username, current_password):        return "Current password is incorrect"   # الباسورد الحالي غلط    if new1 != new2:        return "New passwords do not match"   # ✅ الباسورد الحالي كان صح!    update_password(username, new1)    return "Password changed successfully"
```

المشكلة مركّبة من نقطتين:

1. **`username` بيُقبل كـ parameter حر** بدل ما يتحدد من الـ session — يعني تقدر "تستهدف" أي حساب من غير ما تكون مسجل دخول بيه.
2. **الرسائل المختلفة بتسرّب نتيجة فحص داخلي** (هل current-password صح) حتى لو العملية الكاملة فشلت — وده بيخلي الـ endpoint ده **oracle كامل لتخمين الباسورد** بدون أي خطر من الـ lockout، لأن القفل مرتبط بشرط تاني تمامًا (تطابق الباسوردين الجدد) مش بعدد المحاولات الفاشلة على current-password.

### الدرس المستفاد

- **أي endpoint بيتحقق من باسورد (حتى لو مش صفحة اللوجن الأساسية)** لازم يتعامل بنفس الحذر من ناحية:
    - رسائل خطأ موحدة ما بتفرقش بين أسباب الفشل المختلفة.
    - آلية rate-limiting/lockout حقيقية مرتبطة **بعدد محاولات current-password الغلط**، مش بشرط منفصل زي تطابق باسوردين جدد.
- **الحقول المخفية (hidden inputs) زي `username` مش وسيلة حماية** — أي حد يقدر يعدلها بسهولة عن طريق Burp أو حتى DevTools في المتصفح.
- **كل مسار ممكن يتحقق فيه من باسورد مستخدم (login, password change, 2FA, إلخ) هو سطح هجوم منفصل** لازم يتفحص لوحده، مش بس صفحة اللوجن الرئيسية — المهاجمين بيدوروا على "الطريق الجانبي" الأسهل زي ده بالظبط.
- التصميم الصحيح: يتأكد الـ `username` من الـ session الحالية بس (مش من input)، والرسائل توحّد لحالة "Current password is incorrect" **أو** رسالة عامة واحدة في كل حالات الفشل بغض النظر عن السبب الداخلي.
