## get和post的区别

“GET请求没有body，只有url，请求数据放在url的querystring中；POST请求的数据在body中“

- GET 用于获取信息，是无副作用的，是幂等的，且可缓存
- POST 用于修改服务器上的数据，有副作用，非幂等，不可缓存



- get相对不安全，但是最好还是用https

- HTTP 协议没有 Body 和 URL 的长度限制，对 URL 限制的大多是浏览器和服务器的原因。

- 某些浏览器post会发送两次请求（header和body），时间消耗大





## POST的四种常见方式

1. application/x-www-form-urlencoded

   最常见的提交方式，和get的url的查询一样，形如

   ```
   POST http://www.example.com HTTP/1.1
   Content-Type: application/x-www-form-urlencoded;charset=utf-8
   
   title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
   ```

2. multipart/form-data

   和第一种是当前原生的<form>表单支持的两种。首先**生成了一个 boundary 用于分割不同的字段**，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。

   ```
   POST http://www.example.com HTTP/1.1
   Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
   
   ------WebKitFormBoundaryrGKCBY7qhFd3TrwA
   Content-Disposition: form-data; name="text"
   
   title
   ------WebKitFormBoundaryrGKCBY7qhFd3TrwA
   Content-Disposition: form-data; name="file"; filename="chrome.png"
   Content-Type: image/png
   
   PNG ... content of chrome.png ...
   ------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
   ```

3. application/json

   通过json格式传输数据，json结构简单清晰，目前很多语言和浏览器都支持。

   ```
   POST http://www.example.com HTTP/1.1 
   Content-Type: application/json;charset=utf-8
   
   {"title":"test","sub":[1,2,3]}
   ```

4. text/xml

   xml协议简单易用，但是结构较为复杂多余。

   ```
   POST http://www.example.com HTTP/1.1 
   Content-Type: text/xml
   
   <?xml version="1.0"?>
   <methodCall>
       <methodName>examples.getStateName</methodName>
       <params>
           <param>
               <value><i4>41</i4></value>
           </param>
       </params>
   </methodCall>
   ```

   