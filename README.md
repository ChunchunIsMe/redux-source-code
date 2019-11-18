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
