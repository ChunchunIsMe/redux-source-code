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

| 类型         |      结果      |
|:-------------|:--------------:|
| Undefined    |  "undefined"   |
| Null         |    "object"    |
| Boolean      |   "boolean"    |
| Number       |    "number"    |
| BigInt       |    "bigint"    |
| String       |    "string"    |
| Symbol       |    "object"    |
| 宿主对象     | 取决于具体实现 |
| Function对象 |   "function"   |
| 其他任何对象 |    "object"    |


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

| 类型           |                     原因                     |
|:---------------|:--------------------------------------------:|
| EvalError      |                与`eval()`有关                |
| InternalError  |    JavaScript引擎内部错误，如：‘递归太多’    |
| RangeError     |         数值变量或参数超出其有效范围         |
| ReferenceError |                   无效引用                   |
| SyntaxError    |     `eval()`解析代码过程中发生的语法错误     |
| TypeError      |           变量或参数不属于有效类型           |
| URIError       | 给`encodeURI()`或`decodeURI()`传递的参数无效 |

## 逻辑代码
这段的讲解我们先将逻辑讲明白，在所有文件讲解的最后再整理Api
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

### createStore.js
这里导出了一个函数，函数传入了三个参数，reducer、preloadedState、enhancer

第一个和第二个参数我们经常使用就是 reducer 函数和 preloadedState (state 初始值)

第三个值是用来改造 dispatch 增加中间件的。

我们来开始阅读源码

首先
```
if (typeof perloadedState === 'function' && typeof enhancer === 'undefined') {
  enhancer = preloadedState;
  preloadedState = undefined
}

if (typeof enhancer !== 'undefined') {
  if (typeof enhancer ! == 'function') {
    throw Error('Expected the enhancer to be a function.')
  }

  return enhancer(createStore)(reducer, preloadedState)
}

if (typeof reducer !=== 'function') {
  throw new Error('Expected the reducer to be a function.')
}
```
这里其实就是对参数进行了判断处理，先是判断如果第二个参数是函数则将它当做第三个参数 enhancer 来看待，然后是判断 enhancer 如果 enhancer存在且是函数那么 返回这个函数调用两次的结果，如果存在且不是函数则报错。然后再判断了reducer是不是函数，如果不是则报错。我们可以从`enhancer(createStore)(reducer, preloadedState)`这句看出 enhancer 的结构应该是
```
function enhancer (createStroe) {
  // ...
  return function (reducer, preloadedState) {
    // ...
    return {
      dispatch,
      subscribe,
      getState,
      replaceReducer,
    }
  }
}
```
我们想想也可以知道肯定在其中改写了一些方法。

然后再定义了一些变量
```
let currentReducer = reducer;  // 获取reducer
let currentState = preloadedState; // 获取初始store
let currentListeners = [];  // 储存subscribe注册的函数
let nextListeners = currentListeners; // 储存dispatch时subscribe注册的函数，因为如果只使用一个数组存储在dispatch执行中又注册了函数则会调用新注册的函数。
let isDispatching = false;  // 判断是否在dispatch，以截断某些在dispatch时不能进行的操作。
```
#### ensureCanMutateNextListeners
这个函数很简单，其实就是拷贝了一下 currentListeners 将此时此刻的监听者函数进行记录
```
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice();
  }
}
```
#### getState
这个函数也很简单，它先判断一下是否正在dispatch，如果正在那么报错，这样是为了保持数据的一致性，不然你获取了state之后，你没有进行任何操作state变了，这会让人困惑，然后直接就把currentState返回给你了
```
function getState() {
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
    );
  }
  return currentState;
}
```
#### subscribe
这个函数也是比较简单

首先做了两个判断，判断传入的参数是否是function，不是则报错，再判断是否正在dispatch如果在则报错不让加入新的监听，这里也是为了做数据一致性

然后他注册了一个状态 isSubscribed 这个有什么用我们往下看

接下来调用 ensureCanMutateNextListeners 给 nextListeners 拷贝赋值再将新的监听加入 nextListeners

然后他返回了一个函数 unsubscribe 这个函数就是用来将监听器移除监听列表的，首先判断 isSubscribed 是否是true 因为如果是false 则已经移除过改函数可以直接返回，因为下面将isSubscribed设置为false了。然后判断是否在dispatch，在则报错，因为这是为了保持数据的一致性，然后调用ensureCanMutateNextListeners，最后找到这个监听并进行删除
```
functiong subscribe(listener) {
  if (typeof listener !== 'function') {
    throw new Error('Expected the listener to be a function');
  }

  if (isDispatching) {
    throw new Error(
      'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
    );
  }

  let isSubscribed = true;

  ensureCanMutateNextListeners();
  nextListeners.push(listener);

  return functiong unsubscribe() {
    if (!isSubscribed) {
      return;
    }

    if (isDispatching) {
      throw new Error(
        'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      );
    }

    isSubscribed = false;

    ensureCanMutateNextListeners();
    const index = nextListeners.indexof(listener);
    nextListeners.splice(index, 1);
  }  
}
```
#### dispatch
一开始函数就进行了三次判断
- 判断是否是简单对象
- 判断action是否存在
- 判断当前是否在进行其他dispatch操作

如果有则报错

接下来
1. 设置dispatch状态为true
2. 调用传入的reducer并赋值给state
3. 最后调用完reducer将dispatch状态设置为false

这里代码放在try里的原因是因为害怕传入的reducer报错

然后 将 currentListeners 赋值成 nextListeners 再赋给 listeners

再进行循环调用监听数组中的函数
```
function dispatch(action) {
  if (!isPlainObject(action)) {
    throw new Error(
      'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
    );
  }

  if (typeof action.type === 'undefined') {
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
    )
  }

  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.');
  }

  try {
    isDispatching = true;
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false;
  }

  const listeners = (currentListeners = nextListeners);
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i];
    listener();
  }

  return action;
}
```
可能你会有疑问，在dispatch中(currentListeners = nextListeners)，然后又在subscribe对nextListeners进行拷贝操作。这其实就是保持监听的一致性。
#### replaceReducer
这个函数其实就是对reducer进行了替换，然后将state进行了重新的初始化
```
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.');
  }

  currentReducer = nextReducer;
  dispath({ type: ActionTypes.REPLACE });
}
```
函数步骤
1. 先判断是否是函数，不是则报错
2. 直接替换reducer
3. 发一个 action 这个type是在util工具库中随机生成的
4. dispatch之后将state设置为切换后的reducer的default返回的值，即重置state
#### observable
这个函数文档中都没说，导出的时候也不知道是导出的时候使用了一个Symbol好像我们好像拿不到，不懂。

#### 初始化
在最后调用了一下dispatch
```
dispatch({ type: ActionTypes.INIT })
```
这个 type 也是 util 随机生成的，目的就是将state设置为default返回的值(所以reducer真的一定要设置default啊。。不然就算你设置了初始值也是undefined)
### combineReducers.js
我们先不看他定义的函数，因为这些函数都是一些错误处理的函数，我们先直接跳到导出的函数 combineReducers 看到 combineReducers 使用外部定义函数的时候可以先跳过
#### combineReducers
我们来看这个函数做了什么
1. 首先获取传入的reducers对象的所有key，创建 finalReducers 储存最终的 reducer
2. 对传入reducer的key进行循环，如果是undefined则使用warn报出警告，如果是函数则拷贝到 finalReducers 中
3. 获取 finalReducers 的所有key，创建一个值，如果是开发环境则给他复制为 {}
4. 创建一个错误储存值 shapeAssertionError，调用 assertReducerShape 函数并将 finalReducers 传入，如果中途报错则将错误赋值给 shapeAssertionError
5. 然后返回一个函数，这个函数就是我们最终需要的生成的合成reducer

然后我们来看这个返回的 combination 
1. 首先标配 reducer 传入的 state 和 action
2. 如果刚刚检测到的错误存在则抛出这个错误
3. 如果是开发环境则调用外部函数检测警告并使用util工具库打印出来
4. 定义一个值来判断state是否改变，并定义一个新的对象表示下个state
5. 循环finalReducerKeys，在循环中获取循环到的key，并获取这个key下的reducer和state
6. 调用循环到的reducer，并赋值给 nextStateForKey
7. 如果返回的是 undefined 则调用外部函数并抛出一个错误
8. 将结果赋值给nextState的相同key下
9. 判断state是否进行了改变，循环中如果改变了则 hasChanged 会变成 true
10. 如果state进行了改变则返回 nextState 如果没有则返回 state
```
function combineReducers(reducers) {
  const reducerKyes = Object.keys(reducers);
  const finalReducers = {};
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i];
    
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key];
    }
  }
  const finalReducerKeys = Object.keys(finalReducers);

  let unexpectedKeyCache;
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {};
  }

  let shapeAssertionError;
  try {
    assertReducerShape(fianlReducers);
  } cache (e) {
    shapeAssertionError = e;
  }

  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError;
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      );

      if (warningMessage) {
        warning(warningMessage);
      }
    }

    let hasChanged = false;
    const nextState = {};
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i];
      const reducer = finalReducers[key];
      const previousStateForKey = state[key];
      const nextStateForKey = reducer(previouStateForKey, action);
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action);
        throw new Error(errorMessage);
      }
      nextState[key] = nextStateForKey;
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
    }
    return hasChanged ? nextState : state;
  }
}
```
#### assertReducerShape
在代码中第一个运行到的函数就是这个，在源码的143行，它将生成的 finalReducers 传了进去，让我们看看它做了什么

1. 首先获取传入的 reducers 的所有 key 进行循环
2. 在获取 key 下的 reducer 
3. 给第一个参数 state 传入 undefined action 传入一个随机生成type
4. 如果返回值是 undefined 说明 reducer 没有设置 default 抛出错误
5. 接下来他又随机生成了一个 type 并判断是否是 undefined 如果是则报错
6. 这样做的原因是再次判断是否设置了 default 因为如果通过了上面的判断没有通过下面的判断的话，是因为传入的 reducer 函数设置了 ActionTypes.INIT 却没有设置 default
```
function assertReducerShape(reducers) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key];
    const initialState = reducer(undefined, { type: ActionTypes.INIT });

    if (typeof intialState === 'undefined') {
      throw new Error(
         `Reducer "${key}" returned undefined during initialization. ` +
          `If the state passed to the reducer is undefined, you must ` +
          `explicitly return the initial state. The initial state may ` +
          `not be undefined. If you don't want to set a value for this reducer, ` +
          `you can use null instead of undefined.`
      )
    }

    const type = 
      '@@redux/PROBE_UNKNOWN_ACTION_' +
      Math.random()
        .toString(36)
        .substring(7)
        .split('')
        .join('.');
    if (typeof reducer(undefined, { type }) === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
          `Don't try to handle ${
            ActionTypes.INIT
          } or other actions in "redux/*" ` +
          `namespace. They are considered private. Instead, you must return the ` +
          `current state for any unknown actions, unless it is undefined, ` +
          `in which case you must return the initial state, regardless of the ` +
          `action type. The initial state may not be undefined, but can be null.`
      )
    }
  })
}
```
#### getUnexpectedStateShapeWarningMessage
第二次执行检测错误的函数是在返回的整合后的reducer中调用的，传入的参数有state,finalReducers,action,unexpectedKeyCache，然后我们来看这个函数
1. 获取了传入 reducers 对象下的 key 储存到 reducerKeys
2. 定义一个提示字符串
3. 如果 reducerKeys 长度为 0 返回一段提示字符串
4. 如果 state 不是简单对象则返回一段提示字符串
5. 循环传入state的所有 key 并进行过滤，如果 reducers 对象中没有这个属性且unexpectedKeyCache 中没有这个属性则过滤出来，这里是为了避免多余的state属性。
6. 再将过滤出来的state keys 数组进行循环，将unexpectedKeyCache下的属性(unexpectedKeys的所有key)设置为true
7. 如果 action 存在并且 type 是 ActionTypes.REPLACE 那么直接返回
8. 如果过滤出了多余的 state 属性则将返回一段提示文字提示有多余的 stage 属性
9. 然后外部如果拿到了字符串则会打印出警告
```
function getUnexpectedStateShapeWarningMessage(
  inputState,
  reducers,
  action,
  unexpectedKeyCache
) {
  const reducerKeys = Object.keys(reducers)
  const argumentName =
    action && action.type === ActionTypes.INIT
      ? 'preloadedState argument passed to createStore'
      : 'previous state received by the reducer'

  if (reducerKeys.length === 0) {
    return (
      'Store does not have a valid reducer. Make sure the argument passed ' +
      'to combineReducers is an object whose values are reducers.'
    )
  }

  if (!isPlainObject(inputState)) {
    return (
      `The ${argumentName} has unexpected type of "` +
      {}.toString.call(inputState).match(/\s([a-z|A-Z]+)/)[1] +
      `". Expected argument to be an object with the following ` +
      `keys: "${reducerKeys.join('", "')}"`
    )
  }

  const unexpectedKeys = Object.keys(inputState).filter(
    key => !reducers.hasOwnProperty(key) && !unexpectedKeyCache[key]
  )

  unexpectedKeys.forEach(key => {
    unexpectedKeyCache[key] = true
  })

  if (action && action.type === ActionTypes.REPLACE) return

  if (unexpectedKeys.length > 0) {
    return (
      `Unexpected ${unexpectedKeys.length > 1 ? 'keys' : 'key'} ` +
      `"${unexpectedKeys.join('", "')}" found in ${argumentName}. ` +
      `Expected to find one of the known reducer keys instead: ` +
      `"${reducerKeys.join('", "')}". Unexpected keys will be ignored.`
    )
  }
}
```
#### getUndefinedStateErrorMessage
最后一个警告函数是在调用整合后的 reducer 中调用的函数中使用的，整合后的 finalReducers 下的方法即是一个 reducer 如果该 reducer 调用时返回的是 undefined 那么将会调用这个函数，并将 reducer 在 finalReducers 中的 key 和 action 传入，接下来让我们看这个函数做了什么
1. 如果 action 存在则给 actionType 赋值为 action.type,不存在则赋值 action
2. 然后如果 actionType 存在则赋值 `action "${String(actionType)}"` 不存在则赋值 'an action'
3. 然后返回一段字符传，在外部进行抛出了这段错误文字
```
function getUndefinedStateErrorMessage(key, action) {
  const actionType = action && action.type;
  const actionDescription = 
    (actionType && `action "${String(actionType)}"`) || 'an action';
  
  return (
    `Given ${actionDescription}, reducer "${key}" returned undefined. ` +
    `To ignore an action, you must explicitly return the previous state. ` +
    `If you want this reducer to hold no value, you can return null instead of undefined.`
  )
}
```
### compose.js
这个文件的代码非常简单，就是用了一个 reduce 将一个函数的返回值作为另一个函数的参数进行调用，然后返回一个函数去这样调用它，然后返回一个函数去调用他们。接下来我们看代码是怎么做的
1. 首先对参数进行结构赋值，那么funcs就是所有参数形成的数组
2. 如果没有进行传参，则返回一个传入什么就返回什么的函数
3. 如果参数只有一个那么直接返回那个参数
4. 如果参数大于一个就进行迭代，这个reduce将返回一个函数。
5. 这个函数调用将会调用传入compose的所有函数，并且将后一个函数的返回值作为前一个的参数，最后那个函数的参数将会是传入compose返回的函数的参数
6. 这里要注意的是 compose 返回的函数调用后，最先调用的是 compose 参数的最后一个函数,然后从后往前调用
```
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
```