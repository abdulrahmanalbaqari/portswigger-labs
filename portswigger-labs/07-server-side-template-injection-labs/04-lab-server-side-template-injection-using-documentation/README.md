# Lab : Server - Side template injection using documentation

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-using-documentation

- الفكره من ال Lab ده  انك تعرف مكان الحقن و تعرف نوع الtemplate وتعمل exploit ليه عن طريق انك تعرفه من الdocumentation

```bash
1. ${foobar}

2. <#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

### فكرة الـ Lab

اللاب اسمها **"SSTI using documentation to find an exploit"**. الفكرة مش بس اكتشاف SSTI، لكن كمان إنك تتعلم إزاي تستخدم **الـ documentation الرسمي** بتاع الـ template engine عشان توصل للـ payload المناسب - لأن مفيش payload واحد يشتغل مع كل الـ engines.

### إيه هي SSTI

لما الـ application ياخد input ويحطه جوه template بدون تعقيم، الـ template engine بيتعامل معاه كـ **كود قابل للتنفيذ** مش نص عادي، وده ممكن يوصل لـ RCE كامل.

### مكان الحقن في اللاب ده تحديدًا

مفيش حقول متعددة زي الاسم أو الإيميل زي في لابات تانية. هنا الموضوع مختلف:

**1. سجّل الدخول بحساب:**

```
content-manager : C0nt3ntM4n4g3r
```

**2. روح لصفحة تعديل "product description template"**

فيه منتج (product) وليه **template** بيتحكم في شكل وصفه في الصفحة، وانت كـ content-manager عندك صلاحية تعدله مباشرة.

**3. مكان الحقن = جوه نص الـ Template نفسه**

جوه الـ template هتلاقي تعبيرات موجودة بالفعل بالـ syntax:

```
${someExpression}
```

يعني مش "حقل واحد تكتب فيه اسمك"، لكن **الـ textarea الكامل بتاع الـ template** هو مكان الحقن، وتقدر:

- تعدّل أي `${expression}` موجود بالفعل
- أو تضيف تعبير جديد من عندك في أي مكان في النص

**الخلاصة: مكان واحد بس للحقن - محتوى الـ template نفسه.**

### الحل خطوة بخطوة

**الخطوة 1 - تأكيد وجود SSTI وتحديد نوع الـ Engine:**

غيّر أي تعبير موجود، أو حط تعبير غير موجود زي:

```
${foobar}
```

واحفظ. هيظهرلك **error message** بيوضح إن الـ engine المستخدم هو **Freemarker**.

**الخطوة 2 - الدخول على الـ Documentation:**

دلوقتي وانت عارف الاسم، تدور في docs بتاع Freemarker على حاجة بتنفذ system commands. هتلاقي:

```
freemarker.template.utility.Execute
```

**الخطوة 3 - بناء الـ Exploit:**

امسح أي syntax غلط دخلته، وحط بدل منه:

```
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

**الخطوة 4 - الحفظ والتحقق:**

احفظ الـ template وافتح صفحة المنتج - الأمر هيتنفذ فورًا، وملف `morale.txt` هيتمسح من `/home/carlos/`، وده معناه إن اللاب اتحلت بنجاح. ✅

### شرح الـ Payload سطر بسطر

```
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

الـ payload ده مقسوم لجزئين أساسيين، كل جزء بيعمل حاجة مختلفة:

#### الجزء الأول: `<#assign ex="freemarker.template.utility.Execute"?new()>`

ده **FreeMarker directive** (أمر خاص بلغة الـ template بتاعت Freemarker)، ومكوّن من عناصر:

| الجزء | الوظيفة |
| --- | --- |
| `<#assign ... >` | ده أمر بيعمل "تعريف متغيّر" جديد جوه الـ template، زي `var` في JavaScript أو `let x =` |
| `ex` | اسم المتغيّر اللي إحنا عاملينه - ممكن تسميه أي اسم |
| `"freemarker.template.utility.Execute"` | اسم الـ Java class بالكامل (Fully Qualified Class Name). الكلاس ده جزء من مكتبة Freemarker نفسها، ووظيفته الأساسية إنه **بيسمح بتنفيذ أوامر على نظام التشغيل** (زي `Runtime.exec()` في Java) |
| `?new()` | ده الجزء الخطير - ده **built-in function** بتاعت Freemarker بتاخد اسم الكلاس (كـ نص/String) وتعمل منه **object جديد فعليًا** (instantiate). يعني بالظبط زي ما تكتب في Java: `new Execute()` |

**بمعنى أبسط:** السطر ده بيقول "اعمل نسخة (object) من كلاس Execute، وسمّيها `ex`".

#### الجزء الثاني: `${ ex("rm /home/carlos/morale.txt") }`

| الجزء | الوظيفة |
| --- | --- |
| `${ ... }` | ده الـ syntax الأساسي في Freemarker لتنفيذ وطباعة نتيجة أي تعبير (expression) في الصفحة |
| `ex("rm /home/carlos/morale.txt")` | هنا بننادي (call) على الـ object اللي عملناه (`ex`) وكأنه دالة، وبنبعتله الأمر اللي عايزين ننفذه كـ argument |

كلاس `Execute` مصمم أساسًا (من مطوّري Freemarker نفسهم) عشان أي string تتبعتله كـ argument، بياخدها ويشغّلها كـ **shell command** على نظام التشغيل مباشرة عن طريق `Runtime.getRuntime().exec()`.

يعني هنا بالظبط بنقوله: نفّذ الأمر:

bash

```bash
rm /home/carlos/morale.txt
```

### ليه اللاب استخدم الكلاس ده بالذات؟

لأن الفكرة الأساسية للاب هي إنك تدور في **الـ documentation الرسمي** بتاع Freemarker، ولو دخلت docs الـ engine ده، هتلاقي كلاس `Execute` متوثّق رسميًا كـ "utility class" - ومكتوب فيه صراحة إنه بينفذ أوامر على النظام. يعني مش استغلال ثغرة في الكود، لكن **استخدام feature موجودة فعليًا وموثقة**، بس في سياق غير آمن (لأن اليوزر قادر يتحكم في محتوى الـ template).
