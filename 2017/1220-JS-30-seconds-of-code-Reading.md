# JS: 30 seconds of code Reading

用一段很短的代码实现 js 功能：

[Chalarangelo/30-seconds-of-code](https://github.com/Chalarangelo/30-seconds-of-code)
[ 30 seconds of code](https://chalarangelo.github.io/30-seconds-of-code/)

看了一下，其实还算简单啦，记一下学到的一些 js 的东西。

1. 匿名函数：`arr => {}`
2. [解构赋值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)：`...arr`，解构赋值还是很有意思的，和 Python 的 `**` 解包有点像，和一些语言的不定长参数也很像，不过 js 里可以更灵活地应用到数组和对象上。
3. `Array.from` 把 iterable 的对象变成数组，后面可惜加 mapFunc，效果如 `map->toArray`。
4. 一些常见的函数式的方法：
   1. arr.filter
   2. arr.reduce
   3. arr.map
5. 匿名函数的神奇用法：`(a=>(a='aaa')) ()`，直接可以返回 a 的值
6. 叹为观止的 shuffle：`arr.sort(() => Math.random() - 0.5)`，就是让 compare 方法返回一个或正或负的数，这种作法真是刷新三观呀。

## 感想

贡献者们很喜欢把一步很小的动作写成方法，比如 head 就是 array 第一个数，tail 是最后一个。

另外，可能 js 一般是写在 client browser 中的，很多 js 代码并不是很考虑效率，毕竟数据量也不会那么多。

还有就是在我看来有些 JS 代码很炫技（我有时也会，不是批评哈），比如在 `zip` 那一块，明明可以用两个 for 写出来的代码，特地用两个嵌套的 `Array.from` 回调来做，让人没法一眼看出来。