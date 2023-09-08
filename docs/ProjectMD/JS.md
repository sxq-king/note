## JS简介

浏览器分为两部分：渲染引擎和JS引擎。

- 渲染引擎：用来解释HTML和CSS，俗称内核，比如chrome的blink，老版本的webkit。

- JS引擎：也成为JS解释器。用来读取网页中的JavaScript代码，对其处理后运行，比如chrome浏览器的V8。

## JS组成

1. ECMAScript

   > ECMAScript是由ECMA国际（原欧洲计算机制造商协会）进行标准化的一门编程语言，这种语言在万维网上应用广泛，它往往被称为JavaScript或 JScript，但实际上后两者是ECMAScript语言的实现和扩展。**CMAScript规定了JS的编程语法和基础核心知识，是所有浏览器厂商共同遵守的一套JS语法工业标准。**

2. DOM--文档对象模型

>  文档对象模型（Document Object Model，简称DOM），是W3C组织推荐的处理可扩展标记语言的标准编程接口。
> 通过 DOM 提供的接口可以对页面上的各种元素进行操作（大小、位置、颜色等）。

3. BOM--浏览器对象模型

> BOM (Browser Object Model，简称BOM) 是指浏览器对象模型，它提供了独立于内容的、可以与浏览器窗口进行
> 互动的对象结构。通过BOM可以操作浏览器窗口，比如弹出框、控制浏览器跳转、获取分辨率等。

  ```
  //浏览器弹出警告
  
  ​    alert("HelloWorld")
  
  ​    //让计算机在页面中输出一个内容
  
  ​    document.write('看我出现没')
  
  ​    //控制台输出内容
  
  ​    console.log('控制台输出')
  ```

## 数据类型

1. String 字符串

2. Number 数值 

   > 字面量：Number.MAX_VALUE 最大值
   >
   > 字面量：Number.MIN_VALUE 最小值
   >
   > 字面量：Inifinity 正无穷 -Inifinity  负无穷
   >
   > NaN ： 不是一个数字 但是使用typeOf会是Number类型

3. Boolean 布尔值 

   > true 真 ;
   >
   > false 假;

4. NUll 空值 

   > 只表示值null
   >
   > null专门用来标识一个空的对象
   >
   > 使用typeOf(null)检查一个null值时会返回object

5. Undefined 未定义 

   > 当声明一个变量，但是并不给变量赋值是，它的值就是undifined

6. Object 对象

> 对象

7. Symbol (ES6新增数据类型)

> 用于规避ES5对象属性名的冲突，保证了此类的属性名的独特性

### 强制类型转换

- 将其他类型转为string

1. 调用被转换数据的toString()方法(注意null和unidifined没有toString())。
2. 调用String()函数。强制类型转换。

``` js
var a = 123; // 123
var b = a.toString(); // "123"
var c = String(a); // "123"
```

- 将其他数据转换为Number

1.使用Number（）函数 :如果是纯数字的字符串可以直接转成数字，如果含有非数字的内容，则转换为NaN

2.通过parseInt(a，进制)，可以将字符串中的有效整数取出来。

```js
var a = "123";
var b = Number(a) // 123

var c = "abc";
var d = Number(b) // NaN

var e = true;
var f = Number(e); //1  true-->1,false-->0

var g = Null;
var h = Number(g); // 0

var i = undifined;
var j = Number(i); //NaN
```

- 将其他数据类型转为Boolean

调用Boolean();函数

```js
```



