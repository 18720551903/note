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

# 5.原理

## 1.模拟定时器和延时器的实现

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

  

## 2. call, bind, apply的实现原理

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

## 3. constructor的实现原理

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

# 6.浏览器存储

- Cookie 到 Web Storage、IndexedDB

|      特性      |                       周期                        |   大小   |          是否同源          |
| :------------: | :-----------------------------------------------: | :------: | :------------------------: |
|     cookie     |             过期时间前后端都可以设置              |    4k    |             是             |
|  localstorage  | 除非清理掉,否则永久在, 可以在多个同源标签页中共享 |  5m左右  |             是             |
| sessionStorage |       页面关闭就不存在, 仅在当前标签页使用        |  5m左右  |             是             |
|   indexedDB    |               除非清理掉,否则永久在               | 250m以上 | 是(同一源能创建多个数据库) |

## 1.Cookie

```js
// 设置cookie
function setCookie(name, value) {
  const Days = 1;
  const exp = new Date();
  exp.setTime(exp.getTime() + Days * 24 * 60 * 60 * 1000);
  document.cookie =
    name + '=' + escape(value) + ';expires=' + exp.toGMTString();
}

// 读取cookie
function getCookie(name) {
  const reg = new RegExp('(^| )' + name + '=([^;]*)(;|$)');
  const arr = document.cookie.match(reg);

  if (arr) {
    return unescape(arr[2]);
  } else {
    return null;
  }
}

// 删除cookie
function delCookie(name) {
  const exp = new Date();
  exp.setTime(exp.getTime() - 1);
  const cval = getCookie(name);
  if (cval != null) {
    document.cookie = name + '=' + cval + ';expires=' + exp.toGMTString();
  }
}
```

## 2.localStorage

```js
// 设置属性
    // 自身方法
    localStorage.setItem("name","bonly");
    // []方法
    localStorage["name"]="bonly";
    // .方法
    localStorage.name="bonly";

// 获取属性
	// 自身方法
    localStorage.getItem("name");
    // []方法
    localStorage["name"];
    // .方法
    localStorage.name;

//移除某个属性
    // 自身方法
    localStorage.removeItem("name");
    // []方法
    delete localStorage["name"];
    // .方法
    delete localStorage.name

//删除所有的值
	localStorage.clear()
```

## 3.sessionStorage

```js
// 设置属性
    // 自身方法
    sessionStorage.setItem("name","bonly");
    // []方法
    sessionStorage["name"]="bonly";
    // .方法
    sessionStorage.name="bonly";

// 获取属性
	// 自身方法
    sessionStorage.getItem("name");
    // []方法
    sessionStorage["name"];
    // .方法
    sessionStorage.name;

//移除某个属性
    // 自身方法
    sessionStorage.removeItem("name");
    // []方法
    delete sessionStorage["name"];
    // .方法
    delete sessionStorage.name

//删除所有的值
	sessionStorage.clear()
```

## 4.indexDB

**1）获得indexedDB对象**

```
if (!window.indexedDB) {
    window.indexedDB = window.mozIndexedDB || window.webkitIndexedDB || window.msIndexDB;     //兼容浏览器
}
```

**2）打开数据库**

```js
var request = indexedDB.open("person",3);        //第二个参数为版本

// request对象为
{
    error: null
    onblocked: null
    onerror: null
    onsuccess: null
    onupgradeneeded: null
    readyState: "done"
    result: { 	
        name: "person"
        objectStoreNames: DOMStringList {0: "person", length: 1}
        onabort: null
        onclose: null
        onerror: null
        onversionchange: null
        version: 3
        __proto__: IDBDatabase
    }
    source: null
    transaction: null
}
```

- 注: 由于事件执行的顺序问题，打开数据库open方法一定是放在window.onload = function(){} 或都其它事件函数之外

- 这个方法接受两个参数，第一个参数是字符串，表示数据库的名字。如果指定的数据库不存在，就会新建数据库。第二个参数是整数，表示数据库的版本。如果省略，打开已有数据库时，默认为当前版本；新建数据库时，默认为`1`。

- indexedDB.open()`方法返回一个 IDBRequest 对象。这个对象通过三种事件`error`、`success`、`upgradeneeded

  **（1）error 事件**

  `error`事件表示打开数据库失败。

  > ```javascript
  > request.onerror = function (event) {
  > console.log('数据库打开报错');
  > };
  > ```

  **（2）success 事件**

  `success`事件表示成功打开数据库。

  > ```javascript
  > var db;
  > request.onsuccess = function (event) {
  > db = request.result;
  > console.log('数据库打开成功');
  > };
  > ```

  这时，通过`request`对象的`result`属性拿到数据库对象。

  **（3）upgradeneeded 事件**

  如果指定的版本号，大于数据库的实际版本号，就会发生数据库升级事件`upgradeneeded`。

  > ```javascript
  > var db;
  > request.onupgradeneeded = function (event) {
  > db = event.target.result;
  > }
  > ```

  这时通过事件对象的`target.result`属性，拿到数据库实例。

**3）新建数据库**

- 新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在`upgradeneeded`事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。

  - 新建数据库以后, 先判断一下某张表格是否存在，如果不存在再新建。

    ```javascript
    request.onupgradeneeded = function(event) {
        db = event.target.result;
        var objectStore;
        if (!db.objectStoreNames.contains('person')) {
            objectStore = db.createObjectStore('person', { keyPath: 'id' });
        }
    }
    ```

    上面代码中，数据库新建成功以后，新增一张叫做`person`的表格，主键是`id`。

    > **主键**（key）是默认建立索引的属性。比如，数据记录是`{ id: 1, name: '张三' }`，那么`id`属性可以作为主键。主键也可以指定为下一层对象的属性，比如`{ foo: { bar: 'baz' } }`的`foo.bar`也可以指定为主键。

  - 新建对象仓库以后，下一步可以新建索引。

    ```
    request.onupgradeneeded = function(event) {
      db = event.target.result;
      var objectStore = db.createObjectStore('person', { keyPath: 'id' });
      objectStore.createIndex('name', 'name', { unique: false });
      objectStore.createIndex('email', 'email', { unique: true });
    }
    ```

    > `IDBObject.createIndex()`的三个参数分别为索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）。

**4）操作数据**

- 事务: transaction

  ```
  var transaction = db.transaction(["customers"], "readwrite");
  ```

  - 参数
    - 第一个参数是事务希望跨越的对象存储空间的列表。如果你希望事务能够跨越所有的对象存储空间你可以传入db.objectStoreNames, 传空数组会报错;
    - 可选, 如果你没有为第二个参数指定任何内容，你得到的是只读事务`readonly`。如果你想写入数据，你需要传入 `"readwrite"` 标识

- 新增数据

  - 新增数据指的是向对象仓库写入数据记录。这需要通过事务完成。

    ```
    function add() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' });
    
      request.onsuccess = function (event) {
        console.log('数据写入成功');
      };
    
      request.onerror = function (event) {
        console.log('数据写入失败');
      }
    }
    
    add();
    ```

    > 上面代码中，写入数据需要新建一个事务。新建时必须指定表格名称和操作模式（"只读"或"读写"）。新建事务以后，通过`IDBTransaction.objectStore(name)`方法，拿到 IDBObjectStore 对象，再通过表格对象的`add()`方法，向表格写入一条记录。
    >
    > 写入操作是一个异步操作，通过监听连接对象的`success`事件和`error`事件，了解是否写入成功。

- 更新数据(增加或修改数据)

  - 更新数据要使用`IDBObject.put()`方法。

  - 如果该数据的主键在数据库中已经有相同主键的时候，则会修改数据库中对应主键的对象

    ```
    function update() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });
    
      request.onsuccess = function (event) {
        console.log('数据更新成功');
      };
    
      request.onerror = function (event) {
        console.log('数据更新失败');
      }
    }
    
    update();
    ```

- 读取数据

  - 读取数据也是通过事务完成。

    ```
    function read() {
       var objectStore = db.transaction(['person']).objectStore('person');
       var request = objectStore.get(1);       //方法用于读取数据，参数是主键的值。
    
       request.onerror = function(event) {
         console.log('事务失败');
       };
    
       request.onsuccess = function( event) {
          if (request.result) {
            console.log('Name: ' + request.result.name);
            console.log('Age: ' + request.result.age);
            console.log('Email: ' + request.result.email);
          } else {
            console.log('未获得数据记录');
          }
       };
    }
    
    read();
    ```

- 删除数据

  - `IDBObjectStore.delete()`方法用于删除记录。

    ```
    function remove() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .delete(1);
    
      request.onsuccess = function (event) {
        console.log('数据删除成功');
      };
    }
    remove();
    ```

- 使用索引

  - 假定新建表格的时候，对`name`字段建立了索引。

    ```
    objectStore.createIndex('name', 'name', { unique: false });
    ```

  - 现在，就可以从`name`找到对应的数据记录了。

    ```
    var transaction = db.transaction(['person'], 'readonly');
    var store = transaction.objectStore('person');
    var index = store.index('name');
    var request = index.get('李四');
    
    request.onsuccess = function (e) {
      var result = e.target.result;
      if (result) {
        // ...
      } else {
        // ...
      }
    }
    ```

**5）删除数据库**

```
var deleteRequest = indexedDB.deleteDatabase('test');
deleteRequest.onsuccess = function(){
    alert("删除成功");
}
db.deleteObjectStore('books');        //删除数据表
```

**6）用例**

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
	<button class="add">添加</button>
	<button class="del">删除</button>
	<button class="clear">清空数据</button>
	<button class="put">修改</button>
	<button class="review">查看</button>
	<button class="del_database">删除数据库</button>
	<button class="index">使用索引</button>

	<script>
		//添加按钮
		function add () {
			console.log('添加数据');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				//单个事务
					// var customerObjectStore = db.transaction("customers", "readwrite").objectStore("customers");
					// customerObjectStore.add({ ssn: "1001", name: "Donna", age: 32, email: "donna@home.org" });
					// customerObjectStore.add({ ssn: "1002", name: "Ada", age: 32, email: "Ada@home.org" }); 
					// addRequest.onsuccess =  () => {
					// 	console.log('添加成功');
					// }
				//多个事务
				const trans = db.transaction(["customers",'supplyer'], "readwrite"); //开启多个事务
				trans.objectStore("supplyer")
					.add({ ssn: "1001", name: "Donna", age: 32, email: "donna@home.org" })
				var addRequest = trans.objectStore("customers")
					.add({ ssn: "1001", name: "Donna", age: 32, email: "donna@home.org" })
				
			}
		} 

		//删除按钮
		function del () {
			console.log('删除按钮');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				var delRequest = db.transaction("customers", "readwrite").objectStore("customers")
					.delete('1001');
				// delRequest.onsuccess =  () => {
				// 	console.log('删除成功');
				// }
			}
		} 

		//修改按钮
		function put () {
			console.log('修改按钮');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				var customerObjectStore = db.transaction("customers", "readwrite").objectStore("customers");
				var putRequest =  customerObjectStore.put({ ssn: "1001", name: "Ada", age: 32, email: "donna@home.org" });
				// putRequest.onsuccess =  () => {
				// 	console.log('修改成功');
				// }
			}
		} 

		//查看按钮
		function review () {
			console.log('查看按钮');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				var customerObjectStore = db.transaction("customers").objectStore("customers");
				const resultRequest = customerObjectStore.get('1001');
				resultRequest.onsuccess = target => {
					console.log(resultRequest.result); 	//查看是异步的直接调用会报错
				}
			}
		} 
		//清空对象仓库
		function clear () {
			console.log('清空对象仓库');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				var customerObjectStore = db.transaction("customers",'readwrite').objectStore("customers");
				const resultRequest = customerObjectStore.clear();
				resultRequest.onsuccess = () => {
					console.log('清空对象仓库成功'); 	//查看是异步的直接调用会报错
				}
			}
		}
		// //删除数据库    待研究
		// function delDatabase () {
		// 	window.indexedDB.databases().then(res => {
		// 		console.log('删除数据库', res);
		// 	})
		// 	indexedDB.open("person", 2); 
    //   var DBDeleteReq = 	indexedDB.deleteDatabase("person"); 
		// 	DBDeleteReq.onsuccess = function(event) { 
		// 		console.log("Database deleted successfully"); 
		// 		window.indexedDB.databases().then(res => {
		// 		console.log('删除数据库', res);
		// 	})
		// 	} 
		// } 

		// //删除对象仓库    待研究
		// function delObjectStore () {
		// 	var request = indexedDB.open("person", 3);        //第二个参数为版本
		// 	request.onupgradeneeded = function (event) {
		// 		const db = event.target.result;
		// 		var customerObjectStore = db.transaction("customers",'readwrite').objectStore("customers");
		// 		if(db.objectStoreNames.contains('customers')){
		// 			db.deleteObjectStore('customers');
		// 			console.log('删除数据表成功');
    // 		}
    // 	}
    // }

		//使用索引单表|连表查询
		function index () {
			console.log('使用索引');
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
				var customerObjectStore = db.transaction("customers",'readonly').objectStore("customers");
				//提前添加索引及数据
				var index = customerObjectStore.index('name');
				var request = index.get('Donna');
				request.onsuccess = (e) => {
					var result = e.target.result;
					console.log(result);  //{ssn: "1001", name: "Donna", age: 32, email: "donna@home.org"}
				}
			}
		}

		//遍历数据表格(对象仓库)的所有记录
		function readAll() {
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onsuccess = function (event) {
				const db = event.target.result;
					var objectStore = db.transaction('customers').objectStore('customers');
					objectStore.openCursor().onsuccess = function (event) {
						var cursor = event.target.result;
		
						if (cursor) {
							console.log('主键' , cursor.key);			 //主键 1001
							console.log('value' ,cursor.value);    //value {ssn: "1001", name: "Donna", age: 32, email: "donna@home.org"}
							cursor.continue();
						} else {
							console.log('没有更多数据了！');
						}
					};
			}
		}


		window.onload = function () {
			const addBtn = document.querySelector('.add');
			const delBtn = document.querySelector('.del');
			const reviewBtn = document.querySelector('.review');
			const putBtn = document.querySelector('.put');
			const clearBtn = document.querySelector('.clear');
			const databaseBtn = document.querySelector('.del_database');
			const indexBtn = document.querySelector('.index');
			addBtn.onclick = add;
			delBtn.onclick = del;
			reviewBtn.onclick = review;
			putBtn.onclick = put;
			clearBtn.onclick = clear;
			databaseBtn.onclick = delObjectStore; //delDatabase delObjectStore
			// indexBtn.onclick = index;
			indexBtn.onclick = readAll;
			if (!window.indexedDB) {
					window.indexedDB = window.mozIndexedDB || window.webkitIndexedDB || window.msIndexDB;     //兼容浏览器
			}
			var request = indexedDB.open("person", 2);        //第二个参数为版本
			request.onerror = function(event) {
				// 错误处理
			};
			request.onupgradeneeded = function(event) {
				const db = event.target.result;
				// 建立一个对象仓库来存储我们客户的相关信息，我们选择 ssn 作为主键（key path）
  			var objectStore = db.createObjectStore("customers", { keyPath: 'ssn' ,autoIncrement: true});   //为该数据库创建一个对象仓库

				//创建第二个对象仓库
				db.createObjectStore("supplyer", { keyPath: 'ssn' ,autoIncrement: true});

				// 建立一个索引来通过姓名来搜索客户。名字可能会重复，所以我们不能使用 unique 索引
				objectStore.createIndex("name", "name", { unique: false });

				// 使用邮箱建立索引，我们向确保客户的邮箱不会重复，所以我们使用 unique 索引
				objectStore.createIndex("email", "email", { unique: true });
				};
		}
	</script>
</body>
</html>
```



# 7.Object方法

## 可枚举属性和不可枚举属性

+ 定义

  在JavaScript中，对象的属性分为可枚举和不可枚举之分，它们是由属性的enumerable值决定的。可枚举性决定了这个属性能否被for…in查找遍历到。 

+ 属性的枚举性会影响以下三个函数的结果：

  + for…in 	此方法只能遍历对象本身的可枚举属性，

  + Object.keys(obj)	  

    + 一个表示给定对象的所有可枚举属性的字符串数组。

      ```js
      var obj = { a: '1', b: '2', c: '3' };
      console.log(Object.keys(obj)); // console: ['a', 'b', 'c']
      ```

  + JSON.stringify 

    +  此方法也只能读取对象本身的可枚举属性，并序列化为JSON对象。 

## Object.prototype.hasOwnProperty()

- 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性（也就是，是否有指定的键）。

- 语法:`obj.hasOwnProperty(prop)`

  ```
  o = new Object();
  o.hasOwnProperty('prop'); // 返回 false
  o.prop = 'exists';
  o.hasOwnProperty('prop'); // 返回 true
  delete o.prop;
  o.hasOwnProperty('prop'); // 返回 false
  ```

## Object.prototype.toString.call()

1. 每一个继承Object的对象都有toString 方法,如果没有重新toString的话,会返回[Object type],其中type是对象的类型

2. 除了对象外,其他的会返回都是内容的字符串

   ```js
   const arr = [1,2,3]
   Object.prototype.toString.call(arr)//'[Object Array]''
   ```

## Object.defineProperty

```css
数据描述符
Object.defineProperty(obj, prop,{
    	value:属性的值
        writable:布尔值,是否可以修改
        ennumerable:是否可枚举
        configurable:是否可以配置描述符,和删除属性
})
存取描述符
Object.defineProperty(obj, prop,{
    ennumerable:是否可枚举 		/可省
   	configurable:是否可以配置		/可省
    get(){
        return 	 //给属性赋值
    }
    set(value){
        		// 修改成最新的值
    }
})


configurable: false 时，不能删除当前属性，且不能重新配置当前属性的描述符
configurable: true时，可以删除当前属性，可以配置当前属性所有描述符。

但是在writable: true的情况下，可以改变value的值
```

## Object.is()

相等运算符的缺点: 会自动转换数据类型

全等运算符的缺点 : NaN不等于自身, +0 等于 -0  (相等运算符中nan也不等于自身)

Object.is	(可以实现:只要两个值相等就相等)

```js
Object.is(+0,0) false
Object.is(NaN,NaN) true
```

## Object.assign方法

```js
const target = { b: 1, c: 9 }
const a = { c: 1, e: 9 }
const b = { k: 1, c: 10 }

** 合并对象**
//方法一
// Object.assign方法用于对象的合并，将其他对象的所有属性，复制到目标对象（target)
 Object.assign(target, a, b)
 console.log(target); //{ b: 1, c: 10, e: 9, k: 1 }

// 方法二
const newObj = { ...target,...a,...b }
console.log(newObj) //{ b: 1, c: 10, e: 9, k: 1 }
```

## Object.create()方法

+ Object.create()方法创建一个新对象, 使用现有的对象来提供新创建的对象的__proto__。 
+ 类似于new 的方法创建一个对象,new 后面是一个构造函数,而Object.create()里面是一个原型
+ 返回值 : 一个新对象，带着指定的原型对象和属性。 
+ 参数

> ```
> Object.create(proto[, propertiesObject]) 
> 1. proto 新创建对象的原型对象。
> 2. propertiesObject可选。如果没有指定为 undefined，则是要添加到新创建对象的自身定义的属性，而不是其原型链上的枚举属性,
> 如果propertiesObject参数是 null 或非原始包装对象，则抛出一个 TypeError 异常。
> ```

```js
const person = {
  isHuman: false,
  printIntroduction: function () {
    console.log(`My name is ${this.name}. Am I human? ${this.isHuman}`);
  }
    //person 对象里面的属性和方法都在me的原型上
};
const me = Object.create(person);
me.printIntroduction();
//"My name is Matthew. Am I human? true"
```

+ 用途

  1. 原型继承

  ```js
  MyClass.prototype = Object.create(SuperClass.prototype); 
  或者
  MyClass.prototype = new SuperClass
  ```

## Object.seal()

> ```
> Object.seal(obj)
> ```

+  `**Object.seal()**`方法封闭一个对象，不可以添加新属性,和删除当前属性. 但是当前属性的值可以重新赋值.
+  现有属性被标记为不可配置
+  不可以delete obj.key

## Object.freeze()

 **`Object.freeze()`** 方法可以**冻结**一个对象。

+ 不能向这个对象添加新的属性，
+ 不能删除已有属性，
+ 不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值
+ 冻结一个对象后该对象的原型也不能被修改。`freeze()` 返回和传入的参数相同的对象。 

------

## Object.entries(obj)

+ 以数组方式返回keys和values,

  返回值  给定对象自身可枚举属性的键值对数组。 

  ```js
  const obj = { foo: 'bar', baz: 42 };
  console.log(Object.entries(obj)); // [ ['foo', 'bar'], ['baz', 42] ]
  ```

## Object.values(obj)

```js
var obj = {a:1,b:'gy'}
 Object.values(obj) //[1,'gy']
```

## Object.keys(obj) 

```js
let person = {name:"张三",age:25,address:"深圳",getName:function(){}}

Object.keys(person) // ["name", "age", "address","getName"]
```

# 8.Array的方法

## 判断数组的方法

+ Object.prototype.toString.call()

>  const arr = [1,2,3]
>  Object.prototype.toString.call(arr)//'[Object Array]''

+ instanceof

  + instanceof是通过判断对象的原型链上是不是能找到类型的prototype

  + > [] instanceof Array //true
    >
    > [] instanceof Object //true

+ Array.isArray()     ES5新增 //推荐

## isArray

**Array.isArray(测试对象)**

```
Array.isArray('你') //false 
相当于 Object.prototype.toString.call(Array,'你') === "[ Object Array]"
```

## 伪数组转真数组

- 什么是伪数组
  1. 可以使用索引对数据进行操作；
  2. 具有length（长度）属性；
  3. 但是**不能使用数组的方法**，如push，pop等。

1. Array.from

   ```
   const originArr = {0:'a',1:'b','length':2};
   const arr = Array.from(originArr);
   console.log(arr);
   ```

2. 创建一个空数组，循环遍历伪数组，将遍历出的数据逐一放在空数组中		

   ```
   var ali = document.getElementsByTagName('li');
   console.log(ali);       // [li, li, li, li]
   ali.push("hello");      // TypeError: ali.push is not a function
   
   var arr = [];           // 先创建空数组
   for(var i=0;i<ali.length;i++){  // 循环遍历伪数组
       arr.push(ali[i]);;    // 取出伪数组的数据，逐个放在真数组中
   }
   arr.push("hello");
   console.log(arr);  
   ```

3. arr.push.apply(arr,伪数组)

   ```
   var ali = document.getElementsByTagName('li');
   console.log(ali);       // [li, li, li, li]
   var arr=[];
   Array.prototype.push.apply(arr,ali);
   //不需要修改push的this，只是利用apply的传参特点
   console.log(arr);
   ```

4. slice

   ```
   var ali = document.getElementsByTagName('li');
   console.log(ali);       // [li, li, li, li]
   var arr = Array.prototype.slice.apply(ali);
   arr.push("hello");
   console.log(arr);       // [li, li, li, li, "hello"]
   ```

5. 扩展符

   ```
   var ali = document.getElementsByTagName('li');
   console.log(ali);       // [li, li, li, li]
   var arr = [...ali]
   arr.push("hello");
   console.log(arr);       // [li, li, li, li, "hello"]
   ```

## flat 数组拍平

```js
const newArr = [1,[1,2,[5,33]]].flat()  
newArr => [1,1,2,[5,33]]
将二维数组转成一维数组,返回一个新数组

将多维数组扁平化
[1, 2, [3, [4, 5]]].flat()
// [1, 2, 3, [4, 5]]
[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]
[1, 2, [3, [4, [5,6]]].flat(Infinity)
// [1, 2, 3, 4, 5,6]
```

## flatMap

相当于map + flat

- flatMap()只能展开一层数组。

+ 对原数组的每个成员执行一个函数,相当于map,然后对返回值组成的数组执行flat方法,返回一个新数组不改变原来的数组

  ```
  const arr = [1,2,3,4,5,6];
  const res = arr.flatMap((val) => {
    return [val * 2]    
    //或者val * 2,会将[val * 2,...]进行扁平化处理并return出去;
  })
  console.log(res);    //[2,4,6,8,10,12]
  ```

## Reduce方法

#### 基本参数

array.reduce(function(prev, currentValue, currentIndex, arr){}, initialValue) 

1. `prev` : initialValue的值,如果没有设置initialValue,则表示`array`中的第一个元素或者上一次执行方法所返回的值

2. `currentValue`:当前循环的`array`的元素

3. `currentIndex`:当前循环的`array`的元素值所对应的下标

4. `arr`:表示当前所遍历的函数

5. `initialValue`:为`prev`指定一个初始值，可不写，则取`array`的第一个元素

#### 例子

+ 计算数组的和 

  ```js
  const a = [1,2,3,3,4]
  let res = a.reduce( (pre,cur) => {return pre + cur})
  ```

+ 数组去重 

  ```js
  方法1
   Array.from(new Set([...a, ...b]))或者[...new Set(数组)]
  方法2
      var arr = [1,2,3,3,4,4,2,1,5]
      var newArr = arr.reduce((prev, cur,index,curarr) => {
          prev.indexOf(cur) === -1 && prev.push(cur);
          return prev;
      },[]);
      console.log(newArr);//[1,2,3,4,5]
  方法3 Array.filter() + indexOf
  ```

+ 计算数组最大值 

  ```js
  方法1
  var max = arr.reduce(function (prev, cur) {
      return Math.max(prev,cur);
  });
  方法2
  Math.max(...数组)
  ```


## join

> 作用：**将数组中的每一项拼接成字符串**

```javascript
// 语法：arr.join(分隔符)
var arr = ['张三','李四','王五','赵六'];
var str = arr.join();  // 不传参数，默认每一项之间以 逗号 进行拼接
var str = arr.join("-");//按 - 进行拼接
var str = arr.join("");//分隔符为空串，中间就没有分隔符
```

## push、pop、unshift、shift 

- 数组的增删操作

  ```
  // --------------------在数组的最后，添加一个或多个项，返回添加后数组的length
  array.push();
  
  // -------------------在数组的最后，删除一项，返回删除的项
  array.pop();
  
  // --------------------在数组的最前面，添加一个或多个项，返回添加后数组的length
  array.unshift();
  
  // ---------------------在数组的最前面，删除一项，返回删除的项
  array.shift();
  ```

## concat、slice

> 数组的合并

```
//-----------------合并数组，不会改变原数组，会返回一个新的拼接好的数组
var newArr = arr.concat(arr2);
```

> 数组的截取

```
var arr = ["赵云","马超","刘备","关羽","张飞"];
//--------------------------数组的截取，从数组中截取一部分，不会改变原数组，返回截取的新数组
var newArr = arr.slice();// 不传参——》从开始截取到最后，截取整个数组——》相当于复制一份
var newArr = arr.slice(begin);// 从begin（下标）开始，截取到最后，包括begin！！
var newArr = arr.slice(begin,end);// 从begin开始，截取到end，包括begin，不包括end！！！
```

## splice

- 数组的删除、添加、替换

```
//------------------splice 方法可以在数组的任意位置，添加或者删除任一项（会改变原数组）
arr.splice(从哪开始删除，删除几个，添加的项1，添加的项2，......)
arr.splice(begin,deleteCount,item1,item2,...)

var arr = ["赵云","马超","刘备","关羽","张飞"];           
           
//删除--------------------从下标为1开始删除，删除两项
arr.splice(1,2);// 删除

//添加--------------------把第一项、第二项添加到下标2的位置
arr.splice(2,0,'第一项','第二项');// 添加

//替换--------------------把下标2这一项替换成新项（先删除，再添加）
  arr.splice(2,1,'新项');// 替换
```

## indexOf、lastIndexOf

```
//------------indexOf()——》查找数组中元素第一次出现的下标——》如果找不到，返回-1
var arr = [1,2,3,4,5,4,3,2,1];
console.log(arr.indexOf(2));// 查找2在数组中第一次出现的下标
console.log(arr.indexOf(100));// 数组中不存在的值，返回-1

// 需求: 判断 arr 中是否有 赵六
var arr = ['张三', '田七', '李四', '王五'];
var index = arr.indexOf('赵六');
if (index === -1) {
    console.log('没有');
}
else {
    console.log('有赵六, 下标是' + index);
}

//-------------lastIndexOf()——》查找数组中元素最后一次出现的下标——》如果找不到，返回-1
var arr = [1,2,3,4,5,4,3,2,1];
console.log(arr.lastIndexOf(2));// 查找2在数组中最后一次出现的下标
console.log(arr.lastIndexOf(100));// 数组中不存在的值，返回-1
```



## reverse、sort 

- 数组的翻转与排序

  ```
  //------------------让当前数组反转
  arr.reverse();
  
  //-------------------让当前数组排序，默认按照首字符排序
  arr.sort();
  //--------------sort方法可以传递一个函数作为参数，设置是升序还是降序排序
  arr.sort(function(a, b){
    // a表示前一项，b表示后一项
    // 如果返回值 >0,则交换位置
    // --------------从小到大升序排列
    return a - b;
    //---------------从大到小降序排列
    return b - a;
  });
  ```

# 9.Function

## Arguments 对象

**`arguments`** 是一个对应于传递给函数的参数的类数组对象。

 `arguments`对象是所有（非箭头）函数中都可用的**局部变量**。你可以使用`arguments`对象在函数中引用函数的参数。此对象包含传递给函数的每个参数，第一个参数在索引0处。例如，如果一个函数传递了三个参数，你可以以如下方式引用他们： 

```js
arguments[0]
arguments[1]
arguments[2]
```

参数也可以被设置：

```js
arguments[1] = 'new value';
```

`arguments`对象不是一个 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array) 。它类似于`Array`，但除了length属性和索引元素之外没有任何`Array`属性。例如，它没有 [pop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/pop) 方法。但是它可以被转换为一个真正的`Array`：

```js
var args = Array.prototype.slice.call(arguments);
var args = [].slice.call(arguments);

// ES2015
const args = Array.from(arguments);
const args = [...arguments];
```

# 10.基本包装类型

**基本包装类型的步骤：**

1. 在js中为了操作方便，如果是简单数据类型要获取方法时——》默认转换成复杂数据类型
2. 变成复杂数据类型之后——》调用其方法，得出结果
3. 结束时，在还原成简单数据类型

**定义: 把基本类型包装成复杂类型 **

- 简单数据类型是没有任何属性和方法的。

- 但是为了方便操作基本数据类型，js中还提供了三个特殊的复杂类型：String、Number、Boolean对象。可以使用其中的方法：
  - Number：    var num = new Number(123);
  - String：    var str = new String('abc');
  - Boolean：   var flag = new Boolean(true);

### Number对象

> Number对象是数字的包装类型，数字可以直接使用这些方法

```javascript
var num = 11.111111;
//------------------保留几位小数
console.log(num.toFixed(2));
//--------------------转成字符串
console.log(num.toString(2));
//--------------------取整
console.log(parseInt(string/number, radix //进制);
 //--------------------取浮点数
console.log(parseFloat(string/number);
```

### Boolean对象

> Boolean对象是布尔类型的包装类型。

```javascript
var flag = true;
//-------------------转成字符串
console.log(flag.toString();)//底层先转成基本包装类型——》使用方法得到字符串——》还原成简单数据类型
```

> undefined和null没有包装类型！！！所以没有方法！！

---

### String对象

> 字符串可以类似于看做是一个数组（不是真的数组——》伪数组）

- **字符串可以遍历——》字符串不是数组，不是真的数组**

  ```js
  var str = 'abcdefg';
  // 底层会默认转换成 String对象，var str = new String('abcdefg');
  //----------------------打印字符串中下标为0的字符
  console.log(str[0]);
  //-----------------------字符串的遍历（类似于数组）
  for (var i = 0; i < str.length; i++) {
      console.log(str[i]);
  }
  ```

- **查找指定字符的位置：indexOf、lastIndexOf**

  ```javascript
  //------------indexOf()——》查找字符第一次出现的下标——》如果找不到，返回-1
  var str = "abdedba";
  console.log(str.indexOf(a));// 查找a在str中第一次出现的下标
  
  //-------------lastIndexOf()——》查找字符最后一次出现的下标——》如果找不到，返回-1
  var str = "abdedba";
  console.log(str.lastIndexOf(a));// 查找a在str中最后一次出现的下标
  ```

- **去除字符串首尾的空格：trim**

  ```javascript
  var str = '      hello world      ';
  //----------------------------去除字符串首尾的空格，中间的不管
  str = str.trim();// 返回去除首尾空格之后的字符串，重新赋值给str
  console.log(str);
  ```

- **字母大小写转换：toUpperCase、toLowerCase**

  ```javascript
  var myName = 'ZhangSan';
  //-----------------------------------每个英文字母转换成大写
  console.log(myName.toUpperCase());
  //-----------------------------------每个英文字母转换成小写
  console.log(myName.toLowerCase());
  ```

---

- **字符串拼接与截取：concat、slice、substring、substr**

  > 拼接——》+用的最多

  ```js
  var str1 = 'abc';
  var str2 = 'def';
  //----------------------拼接+用的最多
  console.log(str1 + str2);
  //------------------------拼接字符串（不用）会返回一个新字符串
  var newStr = str1.concat(str2);
  console.log(newStr);
  ```

  > 字符串的截取

  ```javascript
  var str = 'abcdefg';
  //-----------------------slice(begin,end)——》从begin开始，截取到end（有始无终）
  console.log(str.slice(1, 3));     // bc
  //-------------------------subString(begin,end)——》从begin开始，截取到end（有始无终）
  console.log(str.substring(1, 3));  // bc
  
  //------------------------subStr(begin,length)——》从begin开始，截取length个，包括begin
  console.log(str.substr(1, 3));  // bcd
  ```

---

- **将字符串分割成一个数组：split**

  > 和arr.join（）正好相反

  ```javascript
  // join 将数组的值拼接成一个字符串
  // split('分割符') 将字符串分割成一个数组, 返回值, 就是分割后得到的数组
  
  var str = 'a|b|c|d';
  //-----------------split('分割符'): 将字符串通过分隔符分割成一个数组, 返回分割后得到的数组
  var arr = str.split('|');
  console.log(arr);  // ["a", "b", "c", "d"]
  ```

---

- **字符串替换：replace**

  > 可以把字符串中特定字符替换掉

  ```javascript
  var words = '大菜鸡, 真坑啊!!! 大菜鸡, 大菜鸡';
  //-----------------------str.replace('aa','bb'):将str中的第一个aa替换成bb——》返回替换后的结果
  words = words.replace('菜鸡', '***');
  console.log(words);
  
  //--------------------------------------（拓展）替换所有的需要使用后面讲的正则——》g：全局
  words = words.replace(/你妹的/g, '***')
  console.log(words);
  ```

---

#10.Date对象

> js提供了一个**Date构造函数**，通过Date构造函数可以创建不同的日期对象
>
> ——》因为日期都是不同的！！！
>
> Date对象用来处理日期和时间

- **创建一个日期对象**

  ```javascript
  var now = new Date();//不传参，默认是一个当前时间的对象
  var date = new Date("2019-05-20 12:00:00");//(格式固定)指定具体的时间对象，后面的时分秒可以省略
  console.log(now);
  console.log(date);
  //-----------------------------不常用的方式
  var date = new Date(2019,4,20,12,0,0);// 可以把每一个项分别传入，但是注意月份从0开始的，0~11
  var date = new Date(1558324800000);// 直接传入时间戳也行
  ```

- **日期格式化（了解，不用）**

  > Date对象中有默认的方法可以进行日期格式化，但是不好看，一般不用。

  ```javascript
  var now = new Date()；
  console.log(now);// 默认直接打印now对象，会默认调用toString方法，打印结果是一个字符串
  console.log(now.toString());// 转成标准的字符串日期数据输出（默认）
  console.log(now.toLocaleString()); // 输出本地格式日期
  console.log(now.toLocaleDateString());  // 本地格式日期，只输出日期部分
  console.log(now.toLocaleTimeString());  // 本地格式日期，只输出时间部分
  ```

- **获取日期的指定部分**

  > 之前默认的日期格式化格式很丑，一般不用——》通过获取日期的指定部分，可以自定义格式化日期

  ```javascript
  var now = new Date(); // 当前时间
  
  // 获取年份
  var year = now.getFullYear();
  
  // 获取月份——》月份从0开始，范围是0~11，一般会+1
  var month = now.getMonth() + 1;
  
  // 获取日——》一个月的几号——》getDay表示获取星期几（从0开始，0表示周日，1表示周一）
  var day = now.getDate();
  
  // 获取时
  var hours = now.getHours();
  
  // 获取分
  var minutes = now.getMinutes();
  
  // 获取秒
  var seconds = now.getSeconds();
  
  var str = year + '年' + month + '月' + day + '日, ' + hours + '时' + minutes + '分' + seconds + '秒';
  document.write(str);
  ```

---

- **时间戳**

  > 一般日期打印出来，是字符串的形式
  >
  > **时间戳则是日期的数字形式**，可以运算

  **时间戳：**表示距离1970年01月01日00时00分00秒起，过去的总毫秒数

  **作用：** 用来计算时间差

  **代码：** `var date = +new Date();`

  - 可以统计代码执行的时间

    ```js
    // ------------------------获取开始的时间
    var begin = +new Date(); 
    var sum = 0;
    for (var i = 1; i <= 100000000; i++) {
        sum += i;
    }
    console.log(sum);
    // -------------------------------获取结束的时间
    var end = +new Date();  
    console.log(end - begin);  // 计算时间差，可以得出代码的执行时间毫秒数
    ```

  - 倒计时（距离下课的时间）

    ```js
    // -----------------------------------当前时间
    var now = new Date();
    // ----------------------------------将来需要倒计时的时间
    var future = new Date('2019-5-20 12:00:00'); 
    
    // ------------------------------得到时间差——》转换成秒数（小数后忽略）
    var time = parseInt((future - now) / 1000);  
    
    // --------------------------------秒数中获取时——》1小时=3600秒
    var hours = parseInt(time / 3600);
    
    // --------秒数中获取分——》1分钟=60秒, 对所有的分钟数, 对60求余数即可(超过60的进位到小时中了）
    var minutes = parseInt(time / 60) % 60;
    
    // ---------获取秒数，对秒数求60的余数（超过60的部分进位到分钟去了)
    var seconds = parseInt(minutes % 60);
    
    var str = "距离下课还有: " + hours + '小时' + minutes + '分钟' + seconds + '秒';
    document.write(str);
    ```

# 11.数据结构

## Set

+ es6提供的一种新的数据结构

```js
const a = new Set ([1,2,3,4,5])
    a.size 可以求出它的长度
    
    数组去重
    const a = new Set ([1,2,3,4,5])
    var newArr = [...a]
    或者
    Array.from(new Set(数组))

    添加
    a.add(b) 直接修改原数据结构
    删除
    a.delete(b) 返回true或者false 表示删除成功或者失败
    判断参数是否是set中的成员
    a.has(b) 返回布尔值
    a.clear(b) 直接清空数据结构中的值
```

## Map

+ es6 提供了Map数据结构,它类似于对象,也是键值对的集合,但是键的类型不限于字符串,各种类型包括对象也可以做键

  ```js
  const a = new Map()
      a.size 可以求出它的长度
     设置
      a.set(b,'hh') 直接修改原数据结构
      获取
      a.get(b) 直接修改原数据结构
      删除
      a.delete(b) 返回true或者false 表示删除成功或者失败
      判断参数是否是set中的成员
      a.has(b) 返回布尔值
  ```

  