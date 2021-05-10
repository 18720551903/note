# 1. 获取数据类型

## typeof和instance和Object.prototype.toString

1. **typeof **返回值：number,boolean,string,function,object,undefined

   > null, object 和 array均识别为object
   >
   > 用法:  typeof a = "undefined"

2. **instanceof**:  用来检测 constructor.prototype 是否存在于参数 object 的原型链上。

   > object instanceof constructor
   >
   > 用法:
   >
   > function Car(make, model, year) {
   > this.make = make;
   > this.model = model;
   > this.year = year;
   > }
   > const auto = new Car('Honda', 'Accord', 1998);
   >
   > console.log(auto instanceof Car);
   > // expected output: true

   - 可以用来判断对象类型

   > function a () {}            a instanceof Function   // true
   >
   > *const* b = {};                 b instanceof Object;    // true

3. **Object.prototype.toString**

   对于 `Object.prototype.toString()` 方法，会返回一个形如 `"[object type]"` 的字符串。

   如果对象的 `toString()` 方法未被重写，就会返回如上面形式的字符串。

   ```js
   ({}).toString();     // => "[object Object]"
   Math.toString();     // => "[object Math]"
   ```

   但是，大多数对象，`toString()` 方法都是重写了的，这时，需要用 `call()` 或 `Reflect.apply()` 等方法来调用。

   ```js
   var x = {
     toString() {
       return "X";
     },
   };
   x.toString();                                     // => "X"
   Object.prototype.toString.call(x);                // => "[object Object]"
   ```

- 获取数据类型的方法封装

  ```js
  function typeOf(target) {
    const toString = Object.prototype.toString;
    const map = {
      '[object Boolean]': 'boolean',
      '[object Number]': 'number',
      '[object String]': 'string',
      '[object Function]': 'function',
      '[object Array]': 'array',
      '[object Date]': 'date',
      '[object RegExp]': 'regExp',
      '[object Undefined]': 'undefined',
      '[object Null]': 'null',
      '[object Object]': 'object'
    };
    return map[toString.call(target)];
  }
  
  // 深拷贝
  function deepCopy(data) {
    const t = typeOf(data);
    let o;
  
    if (t === 'array') {
      o = [];
    } else if (t === 'object') {
      o = {};
    } else {
      return data;
    }
  
    if (t === 'array') {
      for (let i = 0; i < data.length; i++) {
        o.push(deepCopy(data[i]));
      }
    } else if (t === 'object') {
      for (const i in data) {
        o[i] = deepCopy(data[i]);
      }
    }
    return o;
  }
  
  ```



# 2.js隐式转换

- js中有7种数据类型，可以分为两类：原始类型、对象类型：

  基础类型(原始值)：

  ```
  Undefined、 Null、 String、 Number、 Boolean、 Symbol (es6新出的，本文不讨论这种类型)
  ```
  
复杂类型(对象值)：
  
```
  object
  ```
  
- 隐式转换最多的两个运算符 + 和 ==。

- 隐式转换中主要涉及到三种转换：

  1、将值(复杂类型)转为原始值，ToPrimitive()。

  2、将值转为数字，ToNumber()。

  3、将值转为字符串，ToString()。

### 1.1、通过ToPrimitive将对象转换为原始值

- ToPrimitive(input, PreferredType?)

  - input是要转换的值

  - PreferredType是可选参数，可以是Number或String类型。

  #### 1.1.1、如果PreferredType被标记为Number，则会进行下面的操作流程来转换输入的值。

  ```
  1、如果输入的值已经是一个原始值，则直接返回它
  2、否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
  如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
  3、否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
  4、否则，抛出TypeError异常。
  ```

  ##### 1.1.2、如果PreferredType被标记为String，则会进行下面的操作流程来转换输入的值。

  ```
  1、如果输入的值已经是一个原始值，则直接返回它
  2、否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
  3、否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
     如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
  4、否则，抛出TypeError异常。
  ```

  #### 1.1.3、如果省略preferedType

  ```
  1、该对象为Date类型，则PreferredType被设置为String
  2、否则，PreferredType被设置为Number
  ```

  #### 1.1.4、valueOf方法和toString方法解析

  - Object.prototype是所有对象原型链顶层原型，所有对象都会继承该原型的方法，故任何对象都会有valueOf和toString方法。

    ##### 1. valueOf

    1、Number、Boolean、String这三种构造函数生成的基础值的对象形式，通过valueOf转换后会变成相应的原始值。如：

    ```
    var num = new Number('123');
    num.valueOf(); // 123
    
    var str = new String('12df');
    str.valueOf(); // '12df'
    
    var bool = new Boolean('fd');
    bool.valueOf(); // true
    ```

    2、Date这种特殊的对象，其原型Date.prototype上内置的valueOf函数将日期转换为日期的毫秒的形式的数值。

    ```
    var a = new Date();
    a.valueOf(); // 1515143895500
    ```

    3、除此之外返回的都为this，即对象本身：

    ```
    var a = new Array();
    a.valueOf() === a; // true
    
    var b = new Object({});
    b.valueOf() === b; // true
    ```

    ##### 2. toString

    Number、Boolean、String、Array、Date、RegExp、Function这几种构造函数生成的对象，通过toString转换后会变成相应的字符串的形式，因为这些构造函数上封装了自己的toString方法。如：

    ```
    var num = new Number('123sd');
    num.toString(); // 'NaN'
    
    var str = new String('12df');
    str.toString(); // '12df'
    
    var bool = new Boolean('fd');
    bool.toString(); // 'true'
    
    var arr = new Array(1,2);
    arr.toString(); // '1,2'
    
    var d = new Date();
    d.toString(); // "Wed Oct 11 2017 08:00:00 GMT+0800 (中国标准时间)"
    
    var func = function () {}
    func.toString(); // "function () {}"
    ```

    除这些对象及其实例化对象之外，其他对象返回的都是该对象的类型，都是继承的Object.prototype.toString方法。

    ```
    var obj = new Object({});
    obj.toString(); // "[object Object]"
    
    Math.toString(); // "[object Math]"
    ```


### 1.2、通过ToNumber将值转换为数字

根据参数类型进行下面转换：

| 参数      | 结果                                                         |
| --------- | ------------------------------------------------------------ |
| undefined | NaN                                                          |
| null      | +0                                                           |
| 布尔值    | true转换1，false转换为+0                                     |
| 数字      | 无须转换                                                     |
| 字符串    | 有字符串解析为数字，例如：‘324’转换为324，‘qwer’转换为NaN, ''转化为数字 |
| 对象(obj) | 先进行 ToPrimitive(obj, Number)转换得到原始值，在进行ToNumber转换为数字 |

```
2 * {} = ?
1、首先*运算符只能对number类型进行运算，故第一步就是对{}进行ToNumber类型转换。
2、由于{}是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
3、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
4、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
5、转换为原始值后再进行ToNumber运算，"[object Object]"就转换为NaN。
故最终的结果为 2 * NaN = NaN
```

### 1.3、通过ToString将值转换为字符串

根据参数类型进行下面转换：

| 参数      | 结果                                                         |
| --------- | ------------------------------------------------------------ |
| undefined | 'undefined'                                                  |
| null      | 'null'                                                       |
| 布尔值    | 转换为'true' 或 'false'                                      |
| 数字      | 数字转换字符串，比如：1.765转为'1.765'                       |
| 字符串    | 无须转换                                                     |
| 对象(obj) | 先进行 ToPrimitive(obj, String)转换得到原始值，在进行ToString转换为字符串 |

### 1.4隐式转换规则

- 字符串和数字比较, 会把字符串转换成数字
- 有布尔值的, 把布尔值转换成数字
- 对象和标准基本类型之间的相等比较, 调用对象的toPrimitive
- 在 == 中 undifined 与null 互相相等
- NaN不等于任何值包括NaN

**1. 对象和布尔值比较**

> 对象和布尔值进行比较时，对象先ToPrimitive(input, Number)运算，然后再转换为数字，布尔值直接转换为数字

```
{} == true;
1、由于{}是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
2、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
3、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
4、转换为原始值后再进行ToNumber运算，"[object Object]"就转换为NaN。
5、true转换为数字1。
故最终的结果为 NaN === 1
```

```
[] == false; 
1、由于[]是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
2、所以会执行valueOf方法，([]).valueOf(),返回的还是[]对象，不是原始值。
3、继续执行toString方法，([]).toString(),返回""，是原始值。			// 数组中的toString()方法会被改写
4、转换为原始值后再进行ToNumber运算，""就转换为0。
5、false转换为数字1。
故最终的结果为 0 === 0
```

**2.  对象和字符串比较**

> 对象和字符串进行比较时，对象先ToPrimitive(input, Number)运算，然后再转换为字符串，然后两者进行比较。

```
[1,2,3] == '1,2,3' 
1、由于[]是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
2、所以会执行valueOf方法，([]).valueOf(),返回的还是[]对象，不是原始值。
3、继续执行toString方法，([]).toString(),返回"1,2,3"，是原始值。			// 数组中的toString()方法会被改写
4、转换为原始值后再进行ToString运算，"1,2,3"就转换为1,2,3"。
故最终的结果为 '1,2,3' === '1,2,3' 
```

**3.字符串和数字比较**

> 字符串和数字进行比较时，字符串转换成数字，二者再比较。

```
'1' == 1 // true
```

**4.字符串和布尔值比较**

> 字符串和布尔值进行比较时，二者全部转换成数值再比较。

```
'1' == true; // true
```

**5.布尔值和数字比较**

> 布尔值和数字进行比较时，布尔转换为数字，二者比较。

```
true == 1 // true
```

# 3.JavaScript执行栈

- 所有的 JS 代码在运行时都是在执行上下文中进行的。执行上下文是一个抽象的概念，JS 中有三种执行上下文：

  - 全局执行上下文，默认的，在浏览器中是 window 对象，并且 this 在非严格模式下指向它。
  - 函数执行上下文，JS 的函数每当被调用时会创建一个上下文。
  - Eval 执行上下文，eval 函数会产生自己的上下文，这里不讨论。

- 栈，是一种数据结构，具有先进后出的原则。JS 中的执行栈就具有这样的结构，当引擎第一次遇到 JS 代码时，会产生一个全局执行上下文并压入执行栈，每遇到一个函数调用，就会往栈中压入一个新的上下文。引擎执行栈顶的函数，执行完毕，弹出当前执行上下文, 后进先出。

  **例子**

  ```js
  var count = 0;
  function foo(count) {   //相当于 let count = 0
    count += 1;			//修改该函数执行上下文中count的值
    console.log(count);
  }
  foo(count); // 1
  foo(count); // 1
  
  函数每次被调用都会产生新的执行上下文，并被压入执行栈，执行完毕后当前上下文就会被弹出执行栈。所以第一次调用应该返回 1，第二次调用也应该返回 1，第 n 次调用都应该返回 1。
  ```

  ```js
  var count = 0;
  function foo() {
    count += 1;
    console.log(count);	//修改全局执行上下文中count的值(非严格模式)
  }
  foo(count); // 1
  foo(count); // 2
  ```

# 4. addEventListener

| *vent*       | 必须。字符串，指定事件名。  **注意:** 不要使用 "on" 前缀。 例如，使用 "click" ,而不是使用 "onclick"。  **提示：** 所有 HTML DOM 事件，可以查看我们完整的 [HTML DOM Event 对象参考手册](https://www.runoob.com/jsref/dom-obj-event.html)。 |
| ------------ | ------------------------------------------------------------ |
| *function*   | 必须。指定要事件触发时执行的函数。  当事件对象会作为第一个参数传入函数。 事件对象的类型取决于特定的事件。例如， "click" 事件属于 MouseEvent(鼠标事件) 对象。 |
| *useCapture* | 可选。 布尔值\| 对象                                         |

- *useCapture*
  - 布尔值   
    -  true : 事件在捕获阶段执行; 
    - false(默认)事件在冒泡阶段执行;
  - 对象
    - `capture`:  Boolean; 
      - true事件在捕获阶段执行; 
      - true事件在冒泡阶段执行; 
    - once: Boolean
      - true : 表示 `listener 在添加之后最多只调用一次。如果是` `true，` `listener` 会在其被调用之后自动移除。
    - passive: Boolean
      - 表示 `listener` 永远不会调用 `preventDefault()`;如果 listener 仍然调用了这个函数，客户端将会忽略它并抛出一个控制台警告。

# 5.模拟定时器和延时器的实现

- 定时器

  ```js
  function mySetinterval (callback, intervel) {
        let startTime = Date.now();
        let current;
        return function loop() {
          current = Date.now();
          window.requestAnimationFrame(loop);				// 相当于放置了一个定时器,每隔16.7秒执行一次函数, 递归
          if (current - startTime >=  intervel) {
            startTime = current = Date.now();
            callback();
          } 
        }
      }
      
      mySetinterval(() => {
        console.log('两秒钟到了');
      }, 2000)();
  ```

- 延时器

  ```js
       function mySetinterval (callback, intervel) {
        let startTime = Date.now();
        let current;
        return function loop() {
          current = Date.now();
          const timer = requestAnimationFrame(loop);  //用来取消动画帧
          if (current - startTime >=  intervel) { 
            startTime = current = Date.now();
            callback(timer);
          } 
        }
      }
      
      mySetinterval((timer) => {
        console.log('两秒钟到了');
        cancelAnimationFrame(timer)	//取消动画
      }, 2000)();
  ```

  

# 6. call, bind, apply的实现原理

```js
const obj = {
  sing(num, num2){
    console.log('我喜欢唱歌', num, num2);
    return num
  }
}
const obj2 = {};
 //call
 Function.prototype.call = function (target, ...arg) {
  target._method = this;				// this即调用者obj.sing
  const res = target._method(...arg);	//obj.sing中的this变成 target
  delete target._method;				//删掉该方法
  return res;							//将原函数返回值返回
  }
//apply
Function.prototype.apply = function (target, arg) {
  target._method = this;
  const res = target._method(...arg);
  delete target._method;
  return res;
  }
 // bind
Function.prototype.bind = function (target, ...args) {
  target._method = this;
  return function () {
    const res = target._method(...args);
    delete target._method;
    return res;
  };
}
obj.sing.call(obj2, 1, 2);
obj.sing.apply(obj2, [1, 2]);
obj.sing.bind(obj2, 1, 2)();
```

# 6. constructor的实现原理

```js
function instanceOf (target, structure) {
    const prototype = target.__proto__;
   	if (!target.__proto__ || !structure) {			//原型为null, 到原型链顶端了
        return false;
    } else if ( target.__proto__.constructor === structure) {
        return true;
    } else {
        return instanceOf(target.__proto__, structure)
    }
}
const obj = {};
const flag = instanceOf(obj, Function); //false
const flag1 = instanceOf(obj, Object); //true
```

# 7.Object方法

## 1.Object.create

> 语法: Object.create(proto，[propertiesObject])
>
> propertiesObject: 添加到新创建对象的可枚举属性（即其自身的属性，而不是原型链上的枚举属性)

```js
var o;

// 创建一个原型为null的空对象
o = Object.create(null);

o = {};
// 以字面量方式创建的空对象就相当于:
o = Object.create(Object.prototype);
```

