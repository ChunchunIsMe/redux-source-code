# redux-source-code
redux v4.0.0源码的阅读 从redux源码的api整理js基础知识

先从utils开始读，因为这里是基础工具库，代码比较简单方便我们开始，而且很多地方都有用到。
## utils(自定义工具库)
### actionTypes.js
actionTypes.js 这段代码很好理解，其实就是向外暴露两个action类型，没有什么难点。我们来看看他用到哪些Api
1. Math.random() 这个没什么好说的它返回一个范围是[0, 1)的小数。不接收任何参数
2. Number.prototype.toString()

这个函数可能很多人都用过，但是大家都不知道只有 Number.prototype的toString 可以传参数，它的参数radix代表了数字的基数，也就是我们说的2进制，10进制，16进制等。radix的取值范围是2<= radix <=36 因为最小进制是2进制。0-9（10个数字）+ a-z（26个英文字母）总共36个。

3. String.prototype.substring()

String.prototype.substring
这个函数就是一个截取字符串的方法，返回包含指定部分的新字符串,不会影响原字符串，它接收两个参数，indexStart , indexEnd(可选) 分别代表了开始下标和结束下标，如果这两个值有负值它会把它当做0来看待，返回的字符串则是 [indexStart, indexEnd) 的字符串。如果indexEnd不填则返回 indexStart到最后一个字符

与他类似的是 String.prototype.slice() 他们唯一不同的地方就是如果slice的两个参数是负数，则会转换成 length+indexStart(indexEnd)

4. String.prototype.split()

这个函数使用指定的分隔符字符串将一个String对象分割成子字符串数组，以一个指定的分割字串来决定每个拆分的位置。 

他有两个参数 split(separator ,(limit)) 第一个参数指定分割的字符，第二个参数为限定返回的分割片段数量，如
```
const a = '1.2.3.4.5';
a.split('.') // ['1','2','3','4','5']
a.split('.',2)  // ['1','2']
```
5. Array.prototype.join() 这个没什么好说的只允许一个参数用来分割数组的每个元素。并将其生成字符串返回。

### isPlainObject.js
这段代码也很好理解，其实就是用来判断对象是否是简单对象，简单对象就是只用new Object()或者字面量方式创建出来的对象，也就是原型是Object.prototype。

先判断obj是不是对象类型，然后 proto 到最后一定会变成 Object.prototype 然后如果 `obj.__proto__ === Object.prototype`那么则返回true。

总结一下用到的方法
1. typeof

typeof 是一个操作符，运算符后接操作数返回其类型

规则

|类型|结果|
|:---|:--:|
|Undefined|"undefined"|
|Null|"object"|
|Boolean|"boolean"|
|Number|"number"|
|BigInt|"bigint"|
|String|"string"|
|Symbol|"object"|
|宿主对象|取决于具体实现|
|Function对象|"function"|
|其他任何对象|"object"|


附加信息

null
```
// JavaScript 诞生以来便如此
typeof null === 'object'
```
在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是0.由于null代表的是空值指针(大多数平台下值为0x00)，因此，null的类型标签是0，typeof null 也因此返回'object'.

使用new
```
// 除了 Function 外的所有构造函数的类型都是 'object'
var str = new String('String');
var str1 = String(12345);
var func = new Function();

typeof str; // 返回'object'
typeof str1; // 返回'string'
typeof func; // 返回'function'
```
语法中的括号
```
var num = 1;

typeof num + ' hello'; // 'number hellow'
typeof (num + ' hello'); // 'string'
```
正则表达式
```
typeof /s/ // 'object' ECMA标准
```
错误
```
// 当 typeof 的操作符是未定义 的值会返回 undefined 但是操作符是在typeof后使用 let 或者 const 定义的变量就会报错，因为暂时性死区
typeof undeclar; // undefined;
typeof str; // Uncaught ReferenceError
const str = 'str';
```
例外
```
typeof document.all // undefined

// 当前浏览器都是这么实现的但是这并不是ECMA标准
```
2. BigInt
由于typeof中看到了 BigInt 所以带出

原始数据类型

原始数据类型 BigInt 他是通过在整数末尾附加 n 或通过构造函数来创建的

通过使用常量`Number.MAX_SAFE_INTEGER`,可以获得数字递增的最安全的值，通过引入BigInt，你可以操作超过`Number.MAX_SAFE_INTEGER`的数字。

```
// Number.MAX_SAFE_INTEGER === 9007199254740991
const x = 2n ** 53n;
// 9007199254740992n
const y = x + 1n;
// 9007199254740993n
```
其他的行为皆和number一致,但是BigInt不能和数字互换操作，否则会抛出TypeofError

BigInt 是一种内置对象,不能 new。

创建BigInt的方法有 2 种
```
var num = 1n
var num1 = BigInt(1);
```
Number 和 BigInt 不是严格相等的，是宽松相等的
```
var num = 1;
var num1 =  1n;

num == num1 // true
num === num1 // false
```
静态方法
```
BigInt.asIntN()
// 将一个BigInt值转换成有符号整数
BigInt.asUintN()
// 将一个BIgInt值转换成无符号整数

// 这两个函数都有两个必填参数
width
  可储存整数的位数
bigint
  要储存在指定位数上的整数
```
实例方法有 toLocaleString()/toString()/valueOf() 和number的一致

3. Number.MAX_SAFE_INTEGER

上面出现了 Number.MAX_SAFE_INTEGER 所以我们来带出为什么 Number.MAX_SAFE_INTEGER 是 2 ** 52n -1

详见[JavaScript-Number](https://github.com/ChunchunIsMe/redux-source-code/blob/master/Number.md 'JavaScript-Number')

4. Object.getPrototypeOf()

这个是Object的静态方法，只有一个参数，它会返回指定对象的原型

在 ES5中 如果该方法不是一个对象类型则会报错，但是 ES6 中,参数会被强制转换为一个 Object
### warning.js
这里的代码也很简单，就是判断了一下console的兼容性，然后打印了一下错误，并且抛出了一个错误

这里使用的API
1. console.error 打印一个错误 console 在 IE8 之前不可用
2. throw 语句

用来抛出一个用户自定义的异常。throw后的语句将不会执行，并且控制将被传递到调用堆栈中的第一个catch块。如果调用者没有catch块，程序将会终止。

如：
```
  try {
    throw 'error';
  } catch(e) {
    console.log(e); // expected output: "error"
  }
```

如果抛出一个对象那么可以在语句中将它展开

3. Error

语法：
```
new Error([message],[filename],[lineNumber]);
```
拥有三个可选参数

message: 可选。人类可阅读的错误描述信息。

fileName: 可选。被创建的Error对象的fileName属性值。默认是构造器代码所在的文件名

lineNumber: 可选。被创建的Error对象的lineNumber属性值。默认是调用Error构造器代码所在的文件的行号。

Error可以当做函数使用如 `Error('error')` 他将返回一个Error对象。和new构造的输出相同。

Error类型

|类型|原因|
|:---|:--:|
|EvalError|与`eval()`有关|
|InternalError|JavaScript引擎内部错误，如：‘递归太多’|
|RangeError|数值变量或参数超出其有效范围|
|ReferenceError|无效引用|
|SyntaxError|`eval()`解析代码过程中发生的语法错误|
|TypeError|变量或参数不属于有效类型|
|URIError|给`encodeURI()`或`decodeURI()`传递的参数无效|

## 逻辑代码
### index.js
index.js是整个redux的入口文件，尾部的export出来的方法就是redux方法了。这里的代码也是比较简单的了。

但是还是有两个需要注意的地方

1. `_DO_NOT_USE_ActionTypes`

这个导出的东西还是比较陌生的，如果不是看源码可能都不知道这个是什么，然后我们可以看到这里引入的就是之前util/actionTypes

2. `isCrushed`函数

这里定义了一个空的函数，然后判断环境变量是否是`development`,判断函数名是否是`string`类型,判断函数名是否是`isCrushed`
```
function isCrushed() {}

if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
    "You are currently using minified code outside of NODE_ENV === 'production'. " +
      'This means that you are running a slower development build of Redux. ' +
      'You can use loose-envify (https://github.com/zertosh/loose-envify) for browserify ' +
      'or DefinePlugin for webpack (http://stackoverflow.com/questions/30030031) ' +
      'to ensure you have the correct code for your production build.'
  )
}
```
这个有什么用呢,其实就是用来判断在开发环境是否进行代码压缩，如果进行了代码压缩就进行警告，因为进行代码压缩之后就函数名字就不会是`isCrushed`了