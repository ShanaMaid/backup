---
title: JavaScript数组的思考
date: 2017-03-13 10:33:53
tags:
- JavaScript
- 学习笔记
categories:
- 学习笔记
- web
- JavaScript
---
昨晚睡觉前刷掘金看到一道面试题，由此引发了一系列的拓展与思考。

<!-- more -->

## 面试题
`不使用loop循环，创建一个长度为100的数组，并且每个元素的值等于它的下标`

以下是我的一些解决方案

```
Array.from(Array(100).keys())

[...Array(100).keys()]

Object.keys(Array(100))

Array.prototype.recursion = function(length) {
    if (this.length === length) {
        return this;
    }
    this.push(this.length);
    this.recursion(length);
}
arr = []
arr.recursion(100)

Array(100).map(function (val, index) {
    return index;
})

```
当然还有一种比较作死的方法
```
var arr = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99]
```
用了这种方法你也可能被考官打死！但是确实也没用loop循环。



## 问题与思考
### 问题
将上述方法挨个在控制台上跑了一遍后发现问题了
`Object.keys(Array(100))`的结果为`[]`，即空数组

`map`方法的结果为`[undefined × 100]`

### 思考
初步猜测与`undefined`有关，进行尝试
```
arr = [];
arr[10] = 1;
arr[20] = 2;
Object.keys(arr)  //["10", "20"];

arr.map(function (val, index) {
    return index;
}); //  [undefined × 10, 10, undefined × 9, 20]

```
可以看到显示的是`undefined x 10`,这种显示代表的是数组中未初始化的数量，而并非说是10个`undefined`的值。
也就是说下面式子并不等价，下文会提到前者是稀疏数组，后者是密集数组。
```
Array(5) // [undefined × 5]

[undefined, undefined, undefined, undefined, undefined]
```
也就是说利用`Array()`构造函数创建的数组其实是下面这样的，两者之间是等价的。
```
Array(5)

[,,,,,]
```
问题很明显了，是因为`Object.keys()`与`map`跳过了数组中空的部分，换句话说，也就是数组中没有初始化的部分均会被跳过。
发现问题后就很好解决了，只需要利用es6中的`fill`对数组初始化就可以了。
```
Object.keys(Array(100).fill(0))


Array(100).fill(0).map(function (val, index) {
    return index;
})
```
## 拓展
问题解决后，发现一个问题，以上答案除了递归外均用到了ES6的方法，那么在ES5下怎么解决？查阅相关资料发现了两个词汇`稀疏数组`与`密集数组`。简要概括大概意思就是：

- 稀疏数组：不连续的数组
- 密集数组：连续的数组

```
sparse = []  //稀疏
sparse[1] = 1
sparse[10] = 10

dense = [1, 2, 3, 4, 5]  //密集
```
创建密集数组的关键在于我们要对数组中的每个`index`对应的`value`进行赋值
比如这样
```
Array(5).join().split(',') // ["", "", "", "", ""]
```

这样创建的数组值为`''`，而非空，比较推荐这种。

还可以使用`apply`，比如创建长度为5的密集数组`Array.apply(null, Array(5))`

实际等价为`Array(undefined,undefined,undefined,undefined,undefined)`。

如果考虑ES6的话，我们还可以这样创建密集数组
```
Array.from({length:5})

Array(5).fill(undefined)

```

虽然数组的值是`undefined`，但是却是密集数组，换句话来说就是稀疏与密集数组与数组的值没有任何关系，在于数组中的每个值是否被初始化。

```
sparse = []
sparse[1] = 1
sparse[10] = 10
for (let index in sparse) {
    console.log('sparse:' + 'index=' + index + '    value=' + sparse[index])
}

dense = Array.apply(null, Array(5))
for (let index in dense) {
    console.log('dense:' + 'index=' + index + '    value=' + dense[index])
}

sparse[0] === dense[0]
```
结果为
```
sparse:index=1    value=1
sparse:index=10    value=10
dense:index=0    value=undefined
dense:index=1    value=undefined
dense:index=2    value=undefined
dense:index=3    value=undefined
dense:index=4    value=undefined

true
```
可以看到`dense`中的`value`为`undefined`,但是并没有被`for...in`忽略,并且`sparse[0] === dense[0]`的结果为`true`说明两者之间的`undefined`并没有什么区别,唯一的区别是`sparse`的`undefined`是代表空(未初始化)，`dense`的`undefined`是我们赋值的(已初始化)。

换句话来说，虽然我们赋值是`undefined`，但是由于我们进行了这步赋值操作，js就认为数组已经初始化了，从而不会被`for...in`跳过。

由此可知js中数组相关的方法对于空位(稀疏数组和密集数组)的处理方式是不同，这里在[阮老师的ES6中关于数组的空位](http://es6.ruanyifeng.com/#docs/array#数组的空位)中找到了分析。
> ES5对空位的处理，已经很不一致了，大多数情况下会忽略空位。
- forEach(), filter(), every() 和some()都会跳过空位。
- map()会跳过空位，但会保留这个值
- join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。

> ES6则是明确将空位转为undefined
- Array.from方法会将数组的空位，转为undefined，也就是说，这个方法不会忽略空位。
- 扩展运算符（...）也会将空位转为undefined。
- copyWithin()会连空位一起拷贝。
- fill()会将空位视为正常的数组位置。
- for...of循环也会遍历空位。
- entries()、keys()、values()、find()和findIndex()会将空位处理成undefined

`总之由于对数组的空位的处理规则非常不统一，所以建议避免出现空位。`

实际上，`typeof Array()`我们可以发现结果是`object`,js中的数组就是一个特殊的对象，换句话来说，js中的数组不是传统意义上的数组。
```
arr = []
arr.length // 0

arr[0] = 1
arr.length // 1

arr[100] = 100
arr.length // 101

arr[100] === arr['100'] // true

arr['a'] = 1
arr.length // 101
```
通过上述代码我们可以发现，js的数组中的数字索引其实是字符串形式的数字，同时当我们为数组赋值的时候，如果数值索引的`index`超过数组原有的长度，数组的长度会自动扩充为`index + 1`，同时中间的空隙会自动填充undefined，此时数组会变为稀疏数组。
当`index`不为数值的时候,数值依然会被填充进去，但此时length不会发现变化。

数组插值会引起length变化需要满足两个条件
- isNaN(parseInt(index,10)) === false
- index >= length


继续思考，当数组的长度改变的时候，数组中的值是什么情况呢？
```
arr = [1, 2, 3, 4, 5, 6, 7]
arr['test'] = 1

console.log(arr) // [1, 2, 3, 4, 5, 6, 7, test: 1]

arr.length = 1
console.log(arr) // [1, test: 1]
```
可以看到，如果改变后的`length`如果小于原来的`length`，凡是满足`index`(包括字符串形式的数字,在数组索引中两者等价)大于等于`length`的全部会被删除，即凡是同时满足下列2个条件的都会被删除
- isNaN(parseInt(index,10)) === false
- index >= length

## 总结
面试题
`不使用loop循环，创建一个长度为100的数组，并且每个元素的值等于它的下标`

答案
```
Array.from(Array(100).keys())

[...Array(100).keys()]

Object.keys(Array(100).join().split(','))

Object.keys(Array(100).fill(undefined))

Object.keys(Array.apply(null,{length:100}))

Array.prototype.recursion = function(length) {
    if (this.length === length) {
        return this;
    }
    this.push(this.length);
    this.recursion(length);
}
arr = []
arr.recursion(100)

Array(100).fill(0).map(function (val, index) {
    return index;
})

```

后来仔细想了想，答案中依然存在部分问题，使用了`Object.keys()`的结果数组中的值为字符串形式的数字，`map`在MDN上[Array.prototype.map()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map#Compatibility)的polyfill中的代码来看也是使用了循环。但是出题人的意思应该是指不使用`for...in`、`for...of`、`for`、`while`循环吧。

总之我认为最稳妥的答案是以下几个
```
Array.from(Array(100).keys())

[...Array(100).keys()]


Array.prototype.recursion = function(length) {
    if (this.length === length) {
        return this;
    }
    this.push(this.length);
    this.recursion(length);
}
arr = []
arr.recursion(100)
```





