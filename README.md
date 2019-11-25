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
### applyMiddleware.js
这个js文件的代码量也非常少，但是需要配合 createStroe 中的代码来配合理解。

我们一般使用 applyMiddleware 的时候会调用它并且将它的返回值作为 createStore 的第三个参数进行使用，这样我们就可以使用到中间件啦

我们是否记得在 createStore 中如果存在第三个参数会怎么处理？让我们来看一下
```
if (typeof enhancer !== 'undefined') {
  if (typeof enhancer !== 'function') {
    throw new Error('Expected the enhancer to be a function.')
  }

  return enhancer(createStore)(reducer, preloadedState)
}
```
它直接将第三个参数调用两次后返回了，第一此调用参数传入的是createStroe，第二次调用传入的是reducer、preloadedState即传入createStroe的第一个和第二个参数

我们再来看 applyMiddleWare 这个函数，它的返回值刚好和上面处理第三个参数需要的函数完全契合，然后我们来解读 applyMiddleWare 函数
#### applyMiddleware
我们可以通过 redux-thunk 这个中间件作为例子辅助理解这段代码(说实话 redux-thunk 的源码还真是短)
```
// redux-thunk
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgunmen);
    }

    return next(action);
  }
}

const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
redux-thunk 传入 applyMiddleware 后然后被 createStore 调用步骤：
1. 首先调用传入redux-thunk的applyMiddleWare获取返回值
```
const middleware = applyMiddleware(thunk);
```
2. 传入 createStroe 的第三个参数进行处理,对 middleware 进行两次调用
3. 让我们来看applyMiddleware返回函数返回的函数做了什么操作
4. 首先调用createStore(...args),createStore 就是第三个参数第一次调用时传入的 createStore, ...args 就是第二次调用传入的 reducer和preloadedState,这样就生成了一个新的store
5. 定义了一个 dispatch 函数调用它的话会抛出一个错误
6. 定义一个 middlewareAPI 对象拥有属性 getState 给他赋值为stroe.getState，dispatch 给他赋值为一个函数这个函数的参数将会传入刚刚定义的 dispatch 并将其调用后的返回值返回
7. 对 applyMiddleware 的参数数组进行循环，并将所有参数函数传入 middlewareAPI 调用后返回其返回值并将所有返回值生成 chain 数组，我们这里只有redux-thunk一个函数传入，所以chain是
```
// 上面那么多箭头函数写法有点难看懂，我们把它简化了
const fn = next => {
  return action => {
    if (action === 'function') {
      return action(dispatch, getState, extraArgunmen);
    }
    return next(action);
  }
}

const chain = [fn]
```
8. 传入 chain 调用 compose 然后调用它的返回值传入store.dispatch,这样我们得到的dispath将会是
```
dispatch = action => {
  if (action === 'function') {
    return action(dispatch, getState, extraArgunmen);
  }
  return next(action);
}
// 这里的 next 是 store.dispatch 因为chain只有一个函数，如果有多个函数有可能是后一个函数的返回值
// 第三个参数可以进行忽略，因为导出的thunk都没给createThunkMiddleware赋值
// dispatch 和 getState 是在生成 chain 的时候就传入了
// 这里用了两个闭包函数进行缓存变量。。。。
```
9. 接下来返回一个对象，对象中的方法就是store的所有方法，唯一不同的就是他重写了dispatch方法
10. 我们可以看到使用了redux-thunk当调用dispatch传入的是函数时他会把dispatch传入该函数让该函数进行下一步操作，这样如果是一个异步操作的函数我们就可以在回调函数中使用dispatch了，这样就实现了redux的异步了hhh
```
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStroe(...args);
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      );
    }

    const middlewareAPI = {
      getState: stroe.getState,
      dispatch: (...args) => dispatch(...args)
    }

    const chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(stroe.dispatch);

    return {
      ...stroe,
      dispatch
    }
  }
}
```
### bindActionCreators.js
这个方法的功能其实就是将类似于
```
const addTodo = (text) => ({
  type: 'ADD_TODO',
  text
})
// 或者
const todoObj = {
  add: (num) => ({
    type: 'ADD_NUM',
    num
  }),
  minus: (num) => ({
    type: 'MINUS_NUM',
    num
  }),
}
```
变成
```
const addTodo = (text) => dispatch({
  type: 'ADD_TODO',
  text
})
// 或者
const todoObj = {
  add: (num) => dispatch({
    type: 'ADD_NUM',
    num
  }),
  minus: (num) => dispatch({
    type: 'MINUS_NUM',
    num
  }) 
}
```
说实话没啥用，感觉也没人会用，但是我们还是来看一下他的源码
#### bindActionCreator
这个算是主要功能实现的函数了

1. 首先在文件开头就定义了一个 bindActionCreator 函数函数接受一个函数和dispatch
2. 然后返回一个函数这个函数返回dispatch结果，dispatch参数就是 actionCreator 函数的调用结果，参数就是bindActionCreator返回的函数的参数(然后他还绑定了一下actionCreator调用的this，说实话我觉得没什么必要)
```
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments));
  }
}
```
#### bindActionCreators
1. 有两个参数，一个是 actionCreators，另一个是dispatch
2. 判断如果 actionCreators 是函数直接将 bindActionCreators 的两个参数传入 bindActionCreator 调用后返回
3. 如果 actionCreators 不是 object 类型直接报错,这里再判断一遍 null 是因为 typeof null === ’object‘
4. 获取所有 actionCreators 的 key 赋值给 keys 并创建一个 boundActionCreators 空对象
5. 循环获取的 keys 然后获取到 actionCreators 的 key 下的值，如果是函数则将其传入 bindActionCreator 并将 bindActionCreator 的返回值设置给 boundActionCreators 对象的相同 key 下
6. 循环过后返回 boundActionCreators
```
function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch);
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

  const keys = Object.keys(actionCreators);
  const boundActionCreators = {};
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i];
    const actionCreator = actionCreators[key];
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }

  return boundActionCreators;
}
```
## 逻辑代码API
### index.js
1. import

import 语句用于导入由一个模块导出的绑定。无论是否声明了strict mode，导入的模块都运行在严格模式下。在浏览器中，import 语句只能在声明了`type="module"`的script的标签中使用

此外还有一个，类似函数的动态 `import()`，它不需要依赖`type="module"`的script标签。并且返回一个 Promise 对象

用法
```
// 1. 导出所有模块的内容
import * as myModule from '**/**';
// 2. 导出单个接口
import { myExport } from '**/**';
// 3. 导出多个接口
import { foo, bar } from '**/**';
// 4. 导出带有别名的接口
import {
  reallyReallyLongModuleExportName as shortName
}
from '**/**';
// 5. 导入时重命名多个接口
import {
  reallyReallyLongModuleExportName as shortName,
  anotherLongModuleName as short
}
// 6. 仅为副作用而导入一个模块
import '**/**';
// 7. 导入默认值
import default from '**/**';
// 8. 也可以将 default 语法和命名空间导入一起使用，但是这种用法 default 必须首先声明
import default, * as myModule from '**/**';
import default, {foo, bar} from '**/**';
// 9. 动态import
import(**/**).then((module) => {
  // to do
})
```

2. export

在创建 JavaScript 模块时，`export`语句用于从模块中导出函数、对象或原始值，以便其他程序可以通过`import`语句使用他们。

无论是否声明，导出的模块都处于严格模式。export 语句不能用在嵌入式脚本中。

用法
```
// 1. 使用命名导出
var a = **;
var b = **;

export { a, b };
// 使用默认导出
export default function cube(x) { return x * x * x; }
// 模拟重定向
exprot { default } from './other';
export * from './other'
```
### createStore.js
1. Array.prototype.slice

slice() 方法返回一个新的对象数组，这一对象是一个由`begin`和`end`决定的原数组的浅拷贝。原始数组不会被改变

参数：(begin, end)这两个参数都是可选的

> begin: 提取起始处的索引，从该索引开始开始提取原数组元素。如果是负数则表示提取原数组的倒数第几个，如果省略begin，则 slice 从索引 0 开始，如果 begin 大于原数组的长度，则会返回空数组。

> end: 提取终止处的索引(从 0 开始)，在该索引处结束提取原数组元素。slice会提取原数组中索引从 begin 到 end 的所有元素。如果是负数，则它表示在原数组的倒数第几个元素结束抽取。如果end被省略或者大于数组的长度，则slice会一直提取到原数组末尾。

> String.prototype.slice 和 Array.prototype.slice 参数一致、用法一致

返回值：被截取的数组
2. Array.prototype.push

方法将一个或多个元素添加到数组的末尾，并返回该数组的长度

参数：`elementN`被添加到数组末尾的元素

返回值：新的 length 属性值

例子
```
// 1. 添加元素到数组
var arr = [1, 2, 3];
arr.push(4, 5, 6);
console.log(arr);// [1, 2, 3, 4, 5, 6];
// 2. 合并两个数组
var arr1 = [1, 2, 3];
var arr2 = [4, 5, 6];
Array.prototype.push.apply(arr1, arr2);
console.log(arr);// [1, 2, 3, 4, 5, 6];
```
3. call/apply/bind

因为刚刚看例子apply突然楞了一下所以重新复习一下

call/apply/bind:
```
// call：直接绑定this为第一个参数并且从第二个参数开始都会传入函数然后调用
func.call(thisArg, (arg1), (arg2), ...);
// apply：和call一样只不过传入参数的方式是一个数组
func.apply(thisArg, ([argunmentsArg])
// bind: 调用方式和call一致，只不过bind是返回this绑定的函数且不会直接调用该函数
func.bind(thisArg, (arg1), (arg2), ...);
```
4. Array.prototype.inedxOf

方法返回在数组中可以找到一个给定的元素的第一个索引，如果不存在则返回-1

```
arr.indexOf(searchElement, (formIndex));
```

参数：

`searchElement`: 要查找的元素

`formIndex`(可选): 开始查找的位置。如果负数就是从倒数位置开始找。

String.prototype.indexOf 用法和 Array.prototype.indexOf 完全一致，只是第二个参数是负数时 String.prototype.indexOf 会当做 0

如果只是为了判断数组是否包含一个指定的值，而不是要 index 使用Array.prototype.includes 更好

`includes` 是 ES6 方法，用来判断一个数组是否包含一个指定的值，有返回true，没有返回false

参数和`Array.prototype.inedxOf`一致

但是`Array.prototype.includes`可以判断是否存在NaN、undefined 而 `Array.prototype.inedxOf`不行

5. Array.prototype.splice

`splice`方法通过删除或替换现有元素或者原地添加新元素来修改数组，并以数组形式返回被修改的内容。此方法会改变原数组

语法
```
array.splice(start,(deleteCount), item1, item2, ...);
```

参数：

`start`：指定修改的开始位置(从0计数)。如果超出了数组长度，则从数组末尾开始添加内容；如果是负值，则从末尾开始的第几位；如果负数的绝对值大于数组长度，则表示开始位置为0

`deleteCount`(可选)：整数，表示要移除的数组元素个数，如果是 0 或者负数则不移除元素, 如果被省略那么start之后的所有元素都会被删除，如果不是number类型会先转换

`item1, item2, item3`(可选): 要添加进数组的元素，从start开始。

返回值：

返回被删除元素组成的数组，如果没有删除则为空，如果只删除一个则返回只包含一个元素的数组。
### combineReducers.js
1. Object.keys/Object.values/Object.entries

Object.keys

参数：

  obj 要返回其枚举自身属性的对象

返回值：
  
  一个表示给定对象的所有可枚举属性的字符串数组。

Object.values

参数：

  obj 要返回其枚举自身属性的对象

返回值：
  
  一个表示给定对象的所有可枚举属性值的数组。

Object.entries

参数：

  obj 要返回其枚举自身属性的对象

返回值：
  
  一个表示给定对象的所有可枚举属性值的键值对数组。
```
var obj = {a: 1,b: 2};
Object.keys(obj); // ['a', 'b']
Object.values(obj); // [1, 2]
Object.entries(obj); // [['a', 1], ['b', 2]]
```

### 迭代器
他们的参数都是(除了reduce)

> callback: 用来测试每个元素的函数，接收三个参数

>> element: 用于测试的当前值

>> index: 用于测试当前当前值的索引

>> array: 调用迭代器的数组

> thisArg(可选): 执行 callback 时使用的 this 值

他们都不会改变原数组

1. Array.prototype.every

  迭代每一个元素，如果每一次都返回truthy值(即转换成Boolean都是true的值)，则返回true，否则false

2. Array.prototype.filter

  迭代每一个元素，返回一个新的、由通过测试的元素组成的数组，遍历到返回truthy值即是通过测试

3. Array.prototype.find 

  迭代数组，如果找到满足测试函数的值就停止迭代，返回第一个满足提供测试函数的元素的值，否则返回undefined

4. Array.prototype.findIndex 

  和find一致只不过返回的是索引不是值，如果没找到是返回 -1

5. Array.prototype.flatMap

  这个和 map 的区别是循环函数返回数组flatMap会将它扁平化
  ```
    var arr = [1, 2, 3, 4];
    arr.map(x => [x + 1]); // [[2], [3], [4], [5]]
    arr.flatMap(x => [x + 1]); // [2, 3, 4, 5]
  ```

6. Array.prototype.forEach

  这个方法只是让每个值都执行一次提供的函数

7. Array.prototype.map

  创建一个新数组，其结果是该数组中每个元素都调用提供的函数返回的结果

8. Array.prototype.some

  和every很像但是every是必须所有都通过才返回true这个是只要一个通过就返回true

9. Array.prototype.reduce

  reduce方法对数组中每个元素执行一个提供的reducer函数，将其结果汇总为单个返回值

  参数也和其他的迭代器不太一样

  > callback: 执行数组中每个值的函数，包含四个参数：

  >> accumulator: 累加器累计回调的返回值；它是上次调用回调时返回的累计值，或者 initialValue

  >> currentValue: 数组中正在处理的元素

  >> index: 数组中正在处理的当前元素的索引。如果提供了initialValue，则起始索引为0，否则从索引1开始

  >> array:调用reduce的数组

  > initialValue: 作为第一次调用callback函数时的第一个参数的值。如果没有提供初始值，则将使用数组中的第一个元素。没有初始值的空数组调用reduce将会报错


  返回值为函数累计处理的结果

### applyMiddleware
1. 扩展运算符

可以在函数调用/数组构造时, 将数组表达式或者string在语法层面展开；还可以在构造字面量对象时, 将对象表达式按key-value的方式展开

用法
```
var arr1 = [1, 2, 3];
var arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

var obj1 = {a:1};
var obj2 = {...obj1, b:2}; //{a:1,b:2}

var fun = function (...args) {console.log(args)}

fun(1, 2, 3, 4, 5); // [1, 2, 3, 4, 5]
```