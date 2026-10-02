# What is O Auth

## شرح OAuth 2.0 (مع إضافات توضيحية)

### الفكرة الأساسية

OAuth هو إطار عمل للـ **authorization** (تفويض الوصول) مش authentication أصلاً — الفرق ده مهم جداً وهنرجعله تحت. الهدف الأساسي: تخلي موقع/تطبيق يوصل لبيانات محدودة من حسابك في تطبيق تاني، من غير ما تدي التطبيق الأول الباسورد بتاعك.

**مثال حقيقي:** لما تربط حساب Trello بـ Google Calendar، Trello مش بياخد باسورد Gmail بتاعك، ده بياخد **token** محدود الصلاحية يسمحله بس يشوف/يعدل الكالندر.

### الأطراف التلاتة

| الطرف | الدور |
| --- | --- |
| **Client Application** | التطبيق اللي عايز يوصل لبياناتك (مثلاً موقع بيقولك "Login with Google") |
| **Resource Owner** | إنت، صاحب البيانات |
| **OAuth Service Provider** | جوجل/فيسبوك/جيت هب... اللي عنده الـ Authorization Server والـ Resource Server |

**إضافة:** الـ Authorization Server والـ Resource Server ممكن يكونوا نفس السيرفر فعلياً، لكن منطقياً بيتقسموا لوظيفتين: واحد بيدي التوكنز، التاني بيرجع البيانات لما تستخدم التوكن.

### خطوات الـ Flow

1. الـ Client بيطلب وصول لجزء من بياناتك (بيحدد الـ grant type والـ scope)
2. إنت بتسجل دخول للـ OAuth provider وبتوافق على الطلب
3. الـ Client بياخد **access token**
4. الـ Client بيستخدم التوكن ده عشان يـ call الـ API ويجيب البيانات

### أهم Grant Types

#### Authorization Code Flow

ده الأكتر أماناً — بيتستخدم للتطبيقات اللي عندها backend server. بترجع **code** الأول عبر المتصفح، وبعدين السيرفر (مش المتصفح) هو اللي بيبدل الـ code بـ access token في اتصال server-to-server مباشر — ده بيحمي الـ token من انه يتسرب عبر المتصفح.

#### Implicit Flow

مصمم أصلاً للـ single-page apps اللي مفيهاش backend يقدر يخزن secret. المشكلة: الـ **access token** نفسه بيترجع مباشرة في الـ URL fragment عبر المتصفح — يعني أي حد شايف الـ history أو فيه XSS ممكن ياخده. عشان كده معظم الأنظمة الحديثة بقت تتجنبه لصالح Authorization Code + PKCE.

**إضافة مهمة:** الـ PKCE (Proof Key for Code Exchange) مش موجود في النص الأصلي بس مهم إنك تعرفه — ده extension اتعمل أصلاً للموبايل apps لكن دلوقتي بقى best practice حتى مع web apps عادية، لأنه بيمنع سرقة الـ authorization code حتى لو اتسرب، عن طريق إن الـ client يبعت "code_verifier" وقت التبديل لازم يطابق "code_challenge" اللي بعتها الأول.

### OAuth كـ Authentication ("Login with X")

هنا بقى بيحصل اللبس الأساسي: OAuth **مصمم أصلاً للـ authorization مش authentication**. لكن الناس بقت تستخدمه كبديل لـ SSO كده:

1. التطبيق بيطلب scope بسيط زي `email` أو `profile`
2. بياخد access token، وبيستخدمه يـ call endpoint زي `/userinfo`
3. بيستخدم الـ email اللي رجعله كـ "username" ويسجلك دخول

**المشكلة الجوهرية هنا (ده أهم حاجة تفهمها):** access token في الأصل معناه "عندك صلاحية توصل لداتا معينة" — مش معناه "أنا فعلاً اللي طلبت الحاجة دي دلوقتي". لما تستخدمه كـ substitute لعملية login، إنت بتفترض ضمنياً إن امتلاك التوكن = هوية المستخدم، وده افتراض ممكن ينكسر بسهولة زي ما هنشوف.

**ده السبب اللي خلى OpenID Connect (OIDC) يتعمل** فوق OAuth — عشان يضيف حاجة اسمها **ID Token** (JWT موقّع) مخصص بالكامل للـ authentication، وفيه حقل `aud` (audience) و `nonce` بيربط التوكن بالـ client المحدد ده، بخلاف access token اللي أصلاً معمول عشان الـ API access بس.

### أشهر الثغرات

#### 1. Improper Implicit Flow Implementation

لو التطبيق بيبعت user ID + access token للسيرفر في POST request عشان يعمل session، والسيرفر مش بيتحقق إن الاتنين فعلاً متطابقين (يعني التوكن ده فعلاً بتاع اليوزر ده) — المهاجم يقدر يغير الـ user ID بس ويسجل دخول كأي حد.

#### 2. Flawed CSRF Protection (غياب state parameter)

الـ `state` المفروض يكون قيمة عشوائية غير متوقعة مرتبطة بالسيشن، بترجع تاني في الـ callback عشان تتأكد إن الطلب ده فعلاً بدأ من نفس المتصفح. لو مش موجود، مهاجم يقدر يبدأ OAuth flow بحسابه هو، وبعدين "يخدع" ضحية إنه يكمل الـ flow ده — النتيجة: حساب المهاجم يترتبط بحساب الضحية على التطبيق (Forced Profile Linking).

#### 3. تسريب Authorization Codes / Access Tokens عبر redirect_uri

لو الـ OAuth server مش بيتحقق بدقة من `redirect_uri`، مهاجم يقدر يعمل طلب OAuth بس يحط `redirect_uri` بتاعه هو، يخدع الضحية تعمل الـ flow، فالـ code/token يوصل للمهاجم مباشرة.

**تقنيات bypass شائعة لـ redirect_uri validation:**

- إضافة subdirectories أو query params لو الفاليديشن بس بتتأكد إن الرابط "بيبدأ بـ" domain معين
- استغلال اختلافات الـ parsing بين مكونات السيرفر المختلفة (زي `https://default-host.com&@evil.net`)
- Parameter pollution (بعت `redirect_uri` مرتين)
- استغلال معاملة خاصة لـ `localhost` (تسجيل domain زي `localhost.evil.net`)
- تغيير `response_mode` من query لـ fragment ممكن يغير الـ parsing بالكامل

#### 4. Stealing via Proxy Page

لو مش قادر تحط domain خارجي في `redirect_uri`، جرب تلاقي paths تانية داخل نفس الـ domain المسموح بيها (زي directory traversal)، وبعدين دور على ثغرة فيها زي **open redirect** أو **XSS** أو حتى **HTML injection** (استغلال الـ Referer header) عشان تسرب الـ code/token لدومين بتاعك.

#### 5. Flawed Scope Validation ("Scope Upgrade")

لو سجلت client application بتاعتك عند الـ OAuth provider وطلبت الأول scope بسيط (زي `email`)، ممكن بعدين تضيف scope إضافي (زي `profile`) وقت تبديل الـ code بـ token — لو السيرفر مش بيتحقق إن الـ scope الجديد نفس اللي المستخدم وافق عليه، هتاخد صلاحيات أكتر من غير موافقة تانية.

#### 6. Unverified User Registration

لو الـ OAuth provider بيسمح تسجل حساب بإيميل من غير ما تتحقق منه (verify)، مهاجم يقدر يسجل حساب بنفس إيميل الضحية، وبعدين يستخدمه يـ login كأنه هو على أي client application بتثق في الـ provider ده.

### إزاي تكتشف إن الموقع بيستخدم OAuth (recon)

- دور على زرار "Login with X"
- افتح Burp/proxy وشوف أول request بيروح على `/authorization` endpoint، وشوف الـ params: `client_id`, `redirect_uri`, `response_type`
- جرب endpoints زي:

دول بيرجعوا JSON فيه تفاصيل عن الـ features المدعومة، ممكن تكشف attack surface إضافي مش موثق.
    - `/.well-known/oauth-authorization-server`
    - `/.well-known/openid-configuration`
