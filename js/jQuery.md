* text()，不包含 html 标记
* html()，包含标记
* val()，返回表单字段的值，只能返回有效的value字段
* attr()，随便什么属性都能返回，即使是自定义的 123=“asd”;



* $("p").remove(".italic"); //删除类为 .italic的



```javascript
$.ajax({
    url:"这里一定要填完整的路径啊！参天呐！加 http://前缀啊",
    type:"post",
    data:params,
    dataType:"json",
    contentType:"application/json",
    success:function (result) {
 });
```





### JSON

* key 必须是字符串，value 可以是合法的 JSON 数据类型（字符串, 数字, 对象, 数组, 布尔值或 null）。
* 