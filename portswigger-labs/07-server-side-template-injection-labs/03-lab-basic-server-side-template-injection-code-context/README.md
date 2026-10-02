# Lab : Basic Server-Side Template Injection ( Code Context )

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic-code-context

- الفكره من ال Lab ده انك تكتشف مكان الي الparameter فيه مصاب و تستغله علشان تنفز فيه الاستغلال  وتمشي علي context code علشان يرن ذي الاستغلال الخير مشت علي context علشان يمشي شبه بعضه

```bash
1. user.name}}{{7*7

2. user.__class__

3. user.__init__.__globals__

4. user.__init__.__globals__['__builtins__'].__import__('os').popen('rm morale.txt').read()

5. {% import os %} {{os.system('rm /home/carlos/morale.txt')
```

### الفرق الأساسي: Plaintext Context مقابل Code Context

اللابات بتاعة SSTI في PortSwigger بتتقسم حسب **مكان حقن الـ input** جوه التمبلت، وده بيحدد شكل الـ detection والـ exploitation:

#### Plaintext Context (زي اللاب اللي فات)

مدخلاتك بتتحط **جوه نص عادي (static text)** في الصفحة، يعني علشان تخلي الكود بتاعك يتنفذ، لازم **إنت شخصيًا تكتب كل الـ syntax بتاع الـ engine** من الأول للآخر:

```
${7*7}
```

يعني إنت اللي بتفتح وتقفل التعبير بالكامل.

#### Code Context (اللاب الحالي)

هنا الموضوع مختلف — الـ input بتاعك بيتحط **جوه تعبير (expression) موجود بالفعل وشغال جوه كود التمبلت نفسه**. يعني الـ engine أصلاً بيعالج المكان ده كـ "كود" مش "نص عرض". فمش محتاج تكتب `${ }` أو `{{ }}` كاملة، إنما محتاج **تكسر (break out) من الـ context اللي إنت فيه** — زي string literal — وتحقن كودك، وبعدين تصلّح الـ syntax عشان الباقي يفضل شغال من غير errors.

**مثال مبسّط للفكرة:**

لو الكود الأصلي في التمبلت شكله كده:

```
{{ "Hello " + name }}
```

وإنت بتتحكم في `name`، فبدل ما تحط `{{7*7}}` (لأنك أصلاً جوه كود context مش plaintext)، إنت محتاج تقفل الـ string وتضيف حاجتك:

```
" + (7*7) + "
```

عشان يبقى الناتج النهائي:

```
{{ "Hello " + (7*7) + "" }}
```

### ليه الفرق ده مهم؟

1. **الـ detection مختلف** — لو جربت payload بتاع plaintext context (زي `${7*7}`) في مكان هو أصلاً code context، ممكن يرجعلك **error** بدل الناتج المتوقع، لأنك بتضيف syntax فوق syntax موجود أصلاً — فممكن تفتكر إن مفيش ثغرة أصلاً وهي موجودة.
2. **الحمل بتاعك يبقى أخف** — إنت مش محتاج تفتح tag كامل، بس محتاج تفهم **إيه اللي حواليك بالظبط** (string؟ variable؟ داخل quotes؟) عشان تكسره صح.
3. **بيوريك إن السطح المعرّض للثغرة أوسع مما يبدو** — مطورين كتير بيفتكروا إنهم "آمنين" لأنهم مش بيحطوا user input جوه `{{ }}` كامل، بس لو حطوه جوه تعبير موجود أصلاً (زي concatenation)، برضو بيفضل عرضة للاستغلال.

### طريقة الحل عمليًا

1. جرب حاجة بسيطة زي quote واحدة `"` أو `'` أو قوس وشوف هل بترجع **syntax error** — ده مؤشر قوي إنك جوه code context.
2. من شكل الـ error message، حدد نوع الـ template engine (اللاب ده بالذات بيستخدم غالبًا **Tornado** بتاع بايثون).
3. راجع الـ documentation بتاع الـ engine عشان تعرف إزاي تنفذ كود Python/system commands منه.
4. اكتب payload يقفل الـ context اللي إنت فيه صح، وينفذ الأمر المطلوب (زي `subprocess` لحذف `morale.txt`).
