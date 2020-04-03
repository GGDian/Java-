## RestTemplate

```java
MultiValueMap<String, String> map = new HttpHeaders();
map.add("serial",payment.getSerial());
return restTemplate.postForObject(PAYMENT_URL+"/payment/create", map, CommonResult.class);
```

* 如果第二个参数是对象，即会用 JSON 的消息转换器把对象放在 body 中调用服务方，被调用方要用 @RequestBody 来接
* 如果第二个参数是一个 MultivalueMap，那么这个Map中的值将会被FormHttpMessageConverter以“application/xwww-form-urlencoded”的格式写到请求体中，被调用方不需要用 Requestbody