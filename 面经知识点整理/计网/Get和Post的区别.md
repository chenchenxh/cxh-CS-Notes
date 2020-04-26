## 浏览器的get和post

“GET请求没有body，只有url，请求数据放在url的querystring中；POST请求的数据在body中“

- GET 用于获取信息，是无副作用的，是幂等的，且可缓存
- POST 用于修改服务器上的数据，有副作用，非幂等，不可缓存





---

- get相对不安全，但是最好还是用https

- HTTP 协议没有 Body 和 URL 的长度限制，对 URL 限制的大多是浏览器和服务器的原因。

- 某些浏览器post会发送两次请求（header和body），时间消耗大