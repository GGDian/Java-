### Http

* 对于GET方式的请求，**浏览器会把http header和data一并发送出去**，服务器响应200（返回数据）；

* 而对于POST，**浏览器先发送header**，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。