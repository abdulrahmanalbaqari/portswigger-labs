# What is GraphQL API Vulnerabilities

## GraphQL — ما هو وكيف يُستغل أمنيًا

### إيه هو GraphQL؟

**GraphQL** هي **لغة استعلام لـ APIs (Query Language)** صممتها Facebook عام 2015، هدفها تسهيل التواصل بين الـ **client** (المتصفح/التطبيق) والـ **server**. الفكرة الأساسية: بدل ما السيرفر يفرض عليك شكل رد ثابت، **انت بتحدد بالظبط إيه البيانات اللي عايزها**، والسيرفر بيرجعلك **نفس الشكل اللي طلبته بالظبط**.

#### الفرق الجوهري عن REST API

|  | **REST API** | **GraphQL** |
| --- | --- | --- |
| **عدد الـ endpoints** | endpoint منفصل لكل مورد (`/users`, `/products/1`, إلخ) | **endpoint واحد فقط** لكل العمليات (غالبًا `/graphql`) |
| **الـ HTTP methods** | متعددة (GET, POST, PUT, DELETE) حسب العملية | غالبًا **POST بس** (نوع العملية بيتحدد من **محتوى الطلب نفسه**، مش من الـ method أو الـ URL) |
| **حجم الاستجابة** | ثابت — بترجع كل حقول المورد حتى لو مش محتاجها | **مرن** — انت بتحدد الحقول اللي عايزها بس |
| **عدد الطلبات لجلب بيانات مترابطة** | غالبًا أكتر من طلب (over-fetching/under-fetching) | طلب واحد ممكن يجيب بيانات من أكتر من مصدر مترابط |

#### 💡 إضافة مني: ليه ده مهم أمنيًا من البداية؟

كونه **endpoint واحد بيستقبل كل العمليات** (queries, mutations, subscriptions) معناه إن **كل منطق التحكم في الصلاحيات (access control) لازم يتم على مستوى "الحقل" أو "العملية" نفسها جوه الطلب**، مش على مستوى الـ URL زي REST. ده بيخلق سطح هجوم مختلف تمامًا عن كل لابات الـ Access Control اللي شرحناها بالأول (زي Method-based access control, URL-based access control) — هناك كان الفحص على مستوى الـ endpoint/method، هنا الفحص المفروض يكون **داخل resolver function لكل حقل على حدة**.

### المكونات الأساسية

#### 1. Schema (المخطط)

بيعرّف **هيكل البيانات المتاحة** — الـ types، الحقول، والعلاقات بينهم. بيُعتبر **عقد (contract)** بين الـ frontend والـ backend:

graphql

```graphql
type Product {    id: ID!    name: String!    description: String!    price: Int}
```

الـ `!` بتعني **الحقل إجباري (non-nullable)**.

#### 2. Operations (العمليات الثلاثة)

| النوع | الوظيفة | مكافئها في REST |
| --- | --- | --- |
| **Query** | جلب بيانات | `GET` |
| **Mutation** | إضافة/تعديل/حذف بيانات | `POST` / `PUT` / `DELETE` |
| **Subscription** | اتصال دائم (عادة عبر WebSockets) يسمح للسيرفر يبعت تحديثات فورية | مفيش مكافئ مباشر في REST التقليدي |

#### مثال Query

graphql

```graphql
query myGetProductQuery {    getProduct(id: 123) {        name        description    }}
```

لاحظ إن `Product` ممكن يكون فيه حقول أكتر (زي `price`)، لكن احنا طلبنا بس `name` و`description`.

#### مثال Mutation

graphql

```graphql
mutation {    createProduct(name: "Flamin' Cocktail Glasses", listed: "yes") {        id        name        listed    }}
```

#### 💡 إضافة مني: ليه الـ Mutations أخطر أمنيًا من الـ Queries؟

بما إن الـ Mutations بتغيّر حالة فعلية (state-changing)، أي ثغرة فيها (زي IDOR أو bypass للـ access control) بتكون **أخطر بكتير** من نفس الثغرة في Query — تمامًا زي الفرق اللي شرحناه قبل كده بين قراءة بيانات حد تاني (IDOR عادي) وحذف/تعديل بيانات حد تاني (زي لابات Access Control اللي كنا بنحذف فيها `carlos`). لازم تدّي اهتمام إضافي لأي mutation بتقدر توصلها كمختبر.

### مكونات تانية مهمة للاستغلال الأمني

#### Arguments (المعاملات) — ⚠️ نقطة الضعف الأشهر

graphql

```graphql
query myGetEmployeeQuery {    getEmployees(id:1) {        name { firstname lastname }    }}
```

المستند بيحذرنا صراحة: **لو الـ API بتستخدم arguments للوصول المباشر لـ objects، ممكن تكون عرضة لثغرات access control زي IDOR**.

#### 💡 إضافة مني: ده بالظبط نفس نمط كل لابات الـ IDOR اللي شرحناها!

فاكر لاب **“User ID controlled by request parameter”**؟ المبدأ **مطابق تمامًا** هنا، بس بدل `?id=wiener` في الـ URL، بقى `getEmployees(id: 1)` جوه الـ GraphQL query. **الدرس العام**: مهما تغيّر شكل التقنية (REST, GraphQL, RPC)، **أي argument بيوصل resource مباشرة من غير تحقق صلاحية = IDOR محتملة**. المستند نفسه بيوضح مثال رهيب على كده: منتج “delisted” (ID=3 ناقص من القائمة العامة) بس لسه تقدر توصله مباشرة بالـ query، تمامًا زي لاب “Insecure direct object references” اللي شرحناه (transcripts بأرقام متسلسلة).

#### Aliases — سلاح ذو حدين

graphql

```graphql
query getProductDetails {    product1: getProduct(id: "1") { id name }    product2: getProduct(id: "2") { id name }}
```

الاستخدام الشرعي: طلب أكتر من instance من نفس الـ type في طلب واحد (GraphQL أصلاً بيرفض تكرار نفس اسم الحقل، فالـ alias بيحل المشكلة دي).

#### ⚠️ الاستخدام الخبيث: Bypassing Rate Limiting

graphql

```graphql
query isValidDiscount($code: Int) {    isValidDiscount(code:$code){ valid }    isValidDiscount2:isValidDiscount(code:$code){ valid }    isValidDiscount3:isValidDiscount(code:$code){ valid }}
```

بما إن **rate limiters كتير بتعدّ "عدد الـ HTTP requests"**، مش "عدد العمليات المنفذة"، تقدر تحط **مئات الاستعلامات في طلب HTTP واحد بس** باستخدام aliases مختلفة، وتتجاوز الحماية بالكامل.

#### 💡 إضافة مني: ده نسخة أنيقة جدًا من مشكلة شرحناها قبل كده!

فاكر لاب **“Broken brute-force protection, multiple credentials per request”**؟ كانت الفكرة هناك إنك تحط array من الباسوردات في JSON body واحد بدل string واحد، عشان العداد يتحسب "محاولة واحدة" بس. **نفس المبدأ بالظبط هنا**، بس GraphQL بيوفرلك آلية **رسمية ومدعومة (aliases)** لعمل ده، وده بيخليها أسهل وأشيع في استغلال APIs الحقيقية المبنية على GraphQL.

#### Fragments (أجزاء قابلة لإعادة الاستخدام)

graphql

```graphql
fragment productInfo on Product {    id    name    listed}
```

مفيدة للتنظيم، وممكن كمان تُستخدم في هجمات **query complexity / DoS** (استعلامات متداخلة جدًا بتستهلك موارد السيرفر).

### Introspection — أهم نقطة ضعف في GraphQL أمنيًا

#### إيه هي؟

ميزة مدمجة بتسمحلك **تستعلم السيرفر عن الـ schema نفسه** — يعني "اديني كل الـ types والـ queries والـ mutations المتاحة عندك".

graphql

```graphql
{    "query": "{__schema{queryType{name}}}"}
```

#### ليه دي خطيرة؟

لو **introspection شغالة في production**، أي مهاجم يقدر **يرسم خريطة كاملة للـ API** من غير ما يحتاج يخمن حاجة — كل الـ queries، الـ mutations، الحقول الحساسة، حتى **وصف الحقول (field descriptions)** اللي ممكن تسرّب تفاصيل داخلية.

#### 💡 إضافة مني: دي أقرب حاجة لـ "دليل استخدام كامل للـ API" مجانًا للمهاجم

قارن ده بكل لابات الـ **Information Disclosure** اللي شرحناها (debug pages, `.git` history) — الفكرة الجوهرية واحدة: **كشف معلومات بنيوية عن النظام بيسهّل أي هجوم لاحق**. بس introspection أخطر شوية لأنها **ميزة رسمية مُفعّلة افتراضيًا** في أغلب تطبيقات GraphQL، مش "غلطة نشر" عرضية زي `.git/` المكشوفة. المطور لازم **يتعمد** تعطيلها.

#### تجاوز حماية Introspection (Bypass Techniques)

لو الـ `__schema` ممنوعة، المستند بيقترح تقنيات ذكية:

**1. حقن حرف خاص بعد `__schema`** (استغلال regex ضعيف):

graphql

```graphql
query{__schema{queryType{name}}}
```

لو المطور عمل regex بيستثني `__schema{` **بالظبط**، إضافة **سطر جديد أو مسافة** بعد `__schema` بتكسر الـ pattern matching، لكن GraphQL نفسه بيتجاهل الـ whitespace فمبيأثرش على صحة الاستعلام.

#### 💡 إضافة مني: ده نفس مبدأ الـ “....//” في path traversal!

فاكر لاب **“Traversal sequences stripped non-recursively”**؟ المبدأ **مطابق تمامًا**: فلتر ساذج بيدور على نص **حرفي محدد (`__schema{`)**، وأي **تغيير طفيف في الشكل** (مسافة، سطر جديد، فاصلة) بيخلي الفلتر "يعمى" عن النمط رغم إن **المعنى النهائي للكود ثابت**. المبدأ العام اللي اتكرر معانا في المحادثة دي كلها: **الفلترة بالـ string matching السطحي دايمًا قابلة للتخطي**، والحل الصحيح هو فحص **بعد الـ parsing الكامل**.

**2. جرّب HTTP methods مختلفة**:

```
GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```

لو introspection معطلة بس على POST، جرب GET أو POST بـ `x-www-form-urlencoded`.

#### 💡 إضافة مني: ده نفس مبدأ لاب “Method-based access control can be circumvented”!

بالظبط نفس الفكرة: **فحص أمني مربوط بـ method واحد بس بدل كل الطرق الممكنة لاستدعاء نفس الوظيفة**. المطور عطّل introspection على الـ endpoint الرئيسي، لكن نسي إن نفس الـ logic قابل للاستدعاء بطرق تانية.

#### Suggestions — تسريب الـ Schema حتى مع Introspection مقفولة

في **Apollo GraphQL**، لو عملت استعلام فيه خطأ إملائي بسيط، السيرفر ممكن "يقترح" التصحيح:

```
There is no entry for 'productInfo'. Did you mean 'productInformation' instead?
```

#### 💡 إضافة مني: أداة Clairvoyance — مفهوم مهم للتطبيق العملي

المستند بيذكر أداة **Clairvoyance** بتستغل الـ suggestions دي **آليًا وبشكل متكرر** عشان "تبني" الـ schema بالكامل من الصفر، query بعد query، من غير أي احتياج لـ introspection مفعّلة. ده مثال عملي على كيف **"تسريب معلومات جزئي ومتكرر"** (information disclosure صغير في كل رسالة خطأ) ممكن **يتراكم** عبر أتمتة ليصل لنفس نتيجة introspection كاملة — نفس فكرة "استخدم Burp Scanner لأتمتة التكرار الممل" اللي شرحناها في قسم Essential Skills.

### GraphQL CSRF

#### متى بتحصل؟

- GraphQL endpoint **مش بيتحقق من الـ Content-Type**.
- **مفيش CSRF token** مُطبّق.

#### الفكرة

`POST` بـ `Content-Type: application/json` **آمنة بطبيعتها من CSRF**، لأن المتصفح **مش بيقدر يبعت JSON body تلقائي عبر فورم HTML عادي** (الفورمات بترسل `application/x-www-form-urlencoded` أو `multipart/form-data` بس). لكن لو الـ endpoint **بيقبل** `x-www-form-urlencoded` أو حتى `GET`، بقى **قابل للاستغلال عبر CSRF تقليدي**.

#### 💡 إضافة مني: ربط مباشر بلاب “Forced OAuth profile linking”

المبدأ هنا **نفس** فكرة أي CSRF شرحناها قبل كده — المتصفح بيبعت الكوكيز تلقائيًا، فلو الـ endpoint مش بيتحقق من مصدر الطلب (CSRF token) ولا نوعه (Content-Type)، أي صفحة خبيثة تقدر "تخلي" متصفح الضحية يبعت mutation GraphQL كاملة بالنيابة عنه — تخيل مثلاً `mutation { changeEmail(email: "attacker@evil.com") }` بتتبعت تلقائيًا بمجرد ما الضحية يفتح صفحة عادية!

### استراتيجية اختبار GraphQL API (من المستند + منهجيتي)

#### 1. اكتشاف الـ Endpoint

```
query{__typename}
```

جرب المسارات الشائعة: `/graphql`, `/api`, `/api/graphql`, `/graphql/api`, `/graphql/graphql`، وجرب إضافة `/v1`.

#### 💡 إضافة مني: استخدم نفس منهجية Content Discovery اللي شرحناها

فاكر أداة **“Discover content”** في Burp اللي استخدمناها في لاب **“Inconsistent handling of exceptional input”** لاكتشاف `/admin`؟ **نفس الأداة بالظبط** مفيدة هنا لاكتشاف GraphQL endpoints المخفية، خصوصًا لو لاب **“Finding a hidden GraphQL endpoint”** محتاج استكشاف أعمق من مجرد تجربة المسارات الشائعة.

#### 2. فحص الـ Request Methods

GraphQL آمن لما يقبل **POST + JSON بس**. أي قبول لـ GET أو form-urlencoded = **جرس إنذار** لثغرات CSRF محتملة.

#### 3. فحص الـ Arguments

دوّر على أي argument بيوصل object مباشرة (زي `id`) — **جرب IDs مش معروضة** (زي ID=3 الناقص من القائمة).

#### 4. فحص Introspection

```
{"query": "{__schema{queryType{name}}}"}
```

لو شغالة = سهّلت عليك كل حاجة. لو لأ، جرب تقنيات التخطي (whitespace injection, method switching).

#### 5. فحص Rate Limiting باستخدام Aliases

جرب تبعت نفس الـ mutation/query **عدة مرات بأسماء aliases مختلفة** في طلب واحد، وشوف هل بتتخطى أي rate limit موجود.

#### 6. فحص CSRF Protection

جرب تبعت نفس الـ mutation بـ `Content-Type: x-www-form-urlencoded` أو كـ `GET` — لو اتقبلت، الـ endpoint عرضة لـ CSRF.

### الحماية من هجمات GraphQL

| الإجراء | الهدف |
| --- | --- |
| تعطيل introspection في production (لو الـ API مش عام) | تقليل معلومات متاحة للمهاجم |
| تعطيل suggestions (`hideSchemaDetailsFromClientErrors` في Apollo v4+) | منع Clairvoyance وأدوات مشابهة |
| مراجعة الـ schema للتأكد من عدم كشف حقول خاصة (إيميلات، IDs) | منع تسريب بيانات حساسة بالتصميم |
| تحديد **query depth** (عمق التداخل) | منع DoS عبر queries متداخلة جدًا |
| تحديد **operation limits** (عدد الحقول/aliases المسموح بها) | منع bypass الـ rate limiting بالـ aliases |
| تحديد الحجم الأقصى بالبايت للاستعلام | منع queries ضخمة مُرهقة |
| **Cost analysis** — تقييم التكلفة الحسابية قبل التنفيذ | رفض queries مكلفة جدًا تلقائيًا |
| قبول **POST + application/json بس** + التحقق من مطابقة الـ Content-Type الفعلي | منع CSRF |
| آلية CSRF token حقيقية | طبقة حماية إضافية ضد CSRF |

#### 💡 إضافة مني: خلاصة الربط الكبير

GraphQL مش "عالم جديد منفصل" من الثغرات — هو **نفس المبادئ الأساسية اللي شرحناها في كل المحادثة دي**، لكن بمصطلحات وأدوات جديدة:

| المبدأ العام من اللابات السابقة | تطبيقه في GraphQL |
| --- | --- |
| IDOR (أرقام متسلسلة بدون تحقق صلاحية) | `getProduct(id: 3)` لمنتج مش معروض |
| Blacklist/regex ضعيفة قابلة للتخطي | تخطي حظر introspection بـ whitespace |
| فحص مربوط بـ method واحد بس | introspection معطلة على POST بس، شغالة على GET |
| عدّ الطلبات بدل العمليات الفعلية (rate limiting) | Aliases بتحط عمليات متعددة في HTTP request واحد |
| غياب CSRF token + قبول content-types غير آمنة | GraphQL CSRF |
| Information disclosure تراكمي | Suggestions + Clairvoyance |

الفايدة الحقيقية من فهم GraphQL مش "تعلم تقنية جديدة من الصفر"، لكن **التعرف على نفس الأنماط اللي اتعلمتها في سياق مختلف** — وده بالظبط جوهر قسم **“Essential Skills”** اللي شرحناه قبل كده.
