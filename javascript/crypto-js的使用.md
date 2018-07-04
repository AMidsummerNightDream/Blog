# [crypto-js](https://github.com/brix/crypto-js)

## 使用方法
1. 单独使用需要加密的js
2. 引入`crypto-js.js`这个文件，相当于引入了所有加密方式

## 适用场景
密码可逆，有一定的安全性，`DES`、`AES`

**AES**:高级加密标准（英语：`Advanced Encryption Standard`，缩写：`AES`），在密码学中又称`Rijndael`加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院（NIST）于2001年11月26日发布于FIPS PUB 197，并在2002年5月26日成为有效的标准。2006年，高级加密标准已然成为对称密钥加密中最流行的算法之一。

`AES`有好几种模式、补码方式

我们是用的`CBC`模式，`AES-128bit`, `Pkcs7`补码方式（后台有可能是`PKCS5Padding`，是一样的）
CryptoJS默认就是CBC模式和Pkcs补码

AES方法是支持AES-128、AES-192和AES-256的，加密过程中使用哪种加密方式取决于传入key的类型，否则就会按照AES-256的方式加密。

由于Java就是按照128bit给的，但是由于是一个字符串，需要先在前端将其转为128bit的才行。

使用`CryptoJS.enc.Utf8.parse`方法才可以将key转为128bit

由于后端使用的是PKCS5Padding，但是在使用CryptoJS的时候发现根本没有这个偏移，查询后发现PKCS5Padding和PKCS7Padding是一样的东东，使用时默认就是按照PKCS7Padding进行偏移的。

mode开头的是模式，pad开头的是补码方式。

由于CryptoJS生成的密文是一个对象，如果直接将其转为字符串是一个Base64编码过的，在encryptedData.ciphertext上的属性转为字符串才是后端需要的格式。

由于加密后的密文为128位的字符串，那么解密时，需要将其转为Base64编码的格式。
那么就需要先使用方法`CryptoJS.enc.Hex.parse`转为十六进制，再使用`CryptoJS.enc.Base64.stringify`将其变为Base64编码的字符串，此时才可以传入`CryptoJS.AES.decrypt`方法中对其进行解密。

``` js
  // 字符串类型的key用之前需要用uft8先parse一下才能用
  var key = CryptoJS.enc.Utf8.parse(keyStr); 

  // 加密
  var encryptedData = CryptoJS.AES.encrypt(plaintText, key, {  
    mode: CryptoJS.mode.ECB,
    padding: CryptoJS.pad.Pkcs7
  });
  var encryptedBase64Str = encryptedData.toString();
  // 输出：'RJcecVhTqCHHnlibzTypzuDvG8kjWC+ot8JuxWVdLgY='

  // 需要读取encryptedData上的ciphertext.toString()才能拿到跟Java一样的密文
  var encryptedStr = encryptedData.ciphertext.toString();  
  // 输出：'44971e715853a821c79e589bcd3ca9cee0ef1bc923582fa8b7c26ec5655d2e06'
 
　// 拿到字符串类型的密文需要先将其用Hex方法parse一下
  var encryptedHexStr = CryptoJS.enc.Hex.parse(encryptedStr);
  // 将密文转为Base64的字符串
  // 只有Base64类型的字符串密文才能对其进行解密
  var encryptedBase64Str = CryptoJS.enc.Base64.stringify(encryptedHexStr);  
  // 解密
  var decryptedData = CryptoJS.AES.decrypt(encryptedBase64Str, key, {  
      mode: CryptoJS.mode.ECB,
      padding: CryptoJS.pad.Pkcs7
  });

  // 解密后，需要按照Utf8的方式将明文转位字符串
  var decryptedStr = decryptedData.toString(CryptoJS.enc.Utf8);  
  console.log(decryptedStr); // 'aaaaaaaaaaaaaaaa'

```

## 例子
``` js
  function getAesString(data, key, iv) {
    var key = CryptoJS.enc.UTF8.parse(key);
    var iv = CryptoJS.enc.Utf8.parse(iv);
    var encrypted = CryptoJS.AES.encrypt(data, key,
      {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
      });
    return encrypted.toString(); //返回的是base64格式的密文
  }

  function getDAesString(encrypted,key,iv){//解密
    var key  = CryptoJS.enc.Utf8.parse(key);
    var iv   = CryptoJS.enc.Utf8.parse(iv);
    var decrypted =CryptoJS.AES.decrypt(encrypted,key,
      {
        iv:iv,
        mode:CryptoJS.mode.CBC,
        padding:CryptoJS.pad.Pkcs7
      });
    return decrypted.toString(CryptoJS.enc.Utf8);     
  } 

  function getAES(data){ //加密
    var key = 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA';  //密钥
    var iv = '1234567812345678';
    var encrypted = getAesString(data,key,iv); //密文
    var encrypted1 = CryptoJS.enc.Utf8.parse(encrypted);
    return encrypted;
  }

  function getDAes(data){//解密
    var key = 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA';  //密钥
    var iv = '1234567812345678';
    var decryptedStr = getDAesString(data,key,iv);
    return decryptedStr;
  }

  // md5
  function md5encode(word) {
    return CryptoJS.MD5(word).toString();
  }

  // base64
  var str = CryptoJS.enc.Utf8.parse("张");
  var base64 = CryptoJS.enc.Base64.stringify(str);
  // base64 = 5byg
  var words = CryptoJS.enc.Base64.parse("5byg");
  var parseStr = words.toString(CryptoJS.enc.Utf8);
  // parseStr = 张
```
`key`和`iv`可以配置， 加密和解密时`key`和`iv`一致