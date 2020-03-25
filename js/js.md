### 表单

```html
<!--border控制边框，cellpadding单元格边框,cellspaing 单元格间距-->
<!--bgcolor颜色-->
<table border="1" cellpadding="10" cellspacing="10">
    <caption>位于表格最上方</caption>
    <tr>
        <th>我是表头</th>
        <th>我是第二列</th>
    </tr>
    <tr>
        <td>我是一列</td>
        <td>
            <ul>
                <li>表格内的内嵌</li>
                <li>表格内的内嵌</li>
            </ul>
        </td>
    </tr>
</table>
```

### 列表

![image-20200320145525918](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320145525918.png)

list-style-type：none 可以使小点点去掉

```html
<ul type="circle">
    <li>第一</li>
    <ul>
       <li>嵌套列表</li>
    </ul>
    <li>第二</li>
</ul>
<ol type="i" start="10">
    <li>第一</li>
    <li>第二</li>
</ol>
//自定义标识
<dl>
    <dt>helloWorld</dt>
        <dd>自定义标识</dd>
</dl>

```

### HTML块

![image-20200320150659729](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320150659729.png)

```html
<link rel="stylesheet" type="text/css" href="htmlcss.css">
 <div id="divid">
        我在div里面
 </div>

//Css中的样式
 #divid{
     color: blue;
 }
```

![image-20200320153913643](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320153913643.png)



### 框架

iFrame

### 背景

![image-20200320160638767](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320160638767.png)

![image-20200320165722031](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320165722031.png)



### 字体

![image-20200320170529448](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320170529448.png)

* text-align：对齐方式
* text-indent：进行缩进
* text-transform：可以变换字母大小写
* text-shadow：加阴影
* font-size:大小 
* font-family：字体类型

### 定位

![image-20200320171802155](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320171802155.png)

![image-20200320172308981](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320172308981.png)

### 浮动

![image-20200320172407759](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320172407759.png)

### 盒子模型

![image-20200320175250630](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320175250630.png)

![image-20200320175342171](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320175342171.png)

![image-20200320175508713](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320175508713.png)



#### 自动居中

```
margin-left:auto
margin-right:auto
margin:100px auto
```



![image-20200320180931070](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320180931070.png)





![image-20200320181124959](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320181124959.png)

cursor：变鼠标形状

display：inline 可以使列表 li 变成一行，横向



### JS

```javascript
var arr  = [1,2,34,4];
var arr = new Array("1","2","4");

var arr = new Array();
arr[0] = 10;
arr[1] = 11;
```

添加事件句柄

![image-20200320201634386](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320201634386.png)

![image-20200320202606217](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320202606217.png)

### 自定义对象

![image-20200320204749648](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320204749648.png)

### String对象

![image-20200320205350556](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320205350556.png)

### 数组方法

![image-20200320210420437](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320210420437.png)

![image-20200320210706249](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320210706249.png)

###  DOM对象

![image-20200320211021332](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320211021332.png)

* getElementsByName() 返回的可以是数组

![image-20200320211801038](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320211801038.png)

* 添加节点

![image-20200320211849367](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320211849367.png)

![image-20200320212106226](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320212106226.png)

* 删除子节点

![image-20200320212525895](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320212525895.png)





![image-20200320214117701](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320214117701.png)

### 面向对象

![image-20200320214802599](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320214802599.png)





![image-20200320215903115](C:\Users\垫\AppData\Roaming\Typora\typora-user-images\image-20200320215903115.png)