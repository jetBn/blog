# 理解call()、apply()、bind()用法与区别
***
## 相关介绍

这三个函数都是在`JavaScript`中改变函数调用的`this`指向的，具体的可能就是参数传递不同。

## `call()方法`
该方法使用一个指定的`this`值和给出一个或多个参数来调用一个函数，他们都是`Function`对象的方法，每个函数都能调用。它可传递两个参数第一个为在`function`函数运行时使用的`this`值，第二个为指定的参数。

具体的使用方法

```
let obj = {age: 11}

function fun(arg) {
    console.log(arg)
    console.log(this.age)
}
fun('lili') //直接调用 此时的this指向window所以 第一行打印`lili`第二行打印`undefined`

// 使用call改变函数调用运行时的this

fun.call(obj, 'lili') // 分别打印 lili 11

// 多个参数

function fun(arg) {
    console.log(arguments) 
    console.log(this.age) 
}

fun.call(obj, 'lili', '20')  // 此时的打印分别是arguments中两个参数 lili以及 20，以及11

```

## `apply()方法`
该方法其实跟`call()`方法是一样的就是传递的参数不一样,它的第二个参数为一个数组（或类似数组对象）

具体的使用方法
```
let obj = {age: 11}

function fun() {
    console.log(arguments)
    console.log(this.age)
}

fun.apply(obj, ['lili', '20']) //此时打印个跟上述call传递多个参数的时候是一致的 

```

通过此两个案例我们可以清楚发现，`call()`和`apply()`函数的第一个参数都是改变`this`的指向，而`call()`和`apply()`的区别仅仅就是`call()`传递参数需要一项一项的传， 但是`apply()`直接通过一个数组传递就好了。

## `bind()方法`
该方法与`apply()`和`call()`很相似，也是可以改变函数体内的`this`指向的，它是创建一个新的函数, 在`bind()`被调用的时候，这个新函数的`this`被指定为`bind()`的第一个参数，而其余的的参数将作为新函数的参数，供调用时使用。

具体使用代码
```
let obj = {age: 11}

function fun() {
    console.log(this.age)
}

fun() // undefined

let newFun = fun.bind(obj)

newFun() // 11
```
由上面的运行代码在`bind()`函数改变`this`的指向之前我们打印的是`undefined`,因为我此时的`this`指向是`window`在此里面没有定义`age`的属性，所以我们答应的是`undefined`，当我们通过`bind()`函数改变`this`传入`obj`此时又新生成一个`newFun()`函数,当我们调用此函数的时候，此时的`this`指向就是`obj`了，所以我们打印了 11。

## bind(), apply(), call()的共同和不同点：
* 三个都是`Fucntion`对象的方法都是改变函数的`this`对象的指向的;
* 三个函数的第一个参数都是`this`要指向的对象，也就是想指定的上下文，上下文就是指调用函数的那个对象。（如果没有传以及`null`都是指向`window`）;
* 三个都可以传递参数，`apply()`是数组，`call()`与`bind()`都是有序传入参数;
* `bind`是返回函数，便于稍后调，而`apply()`与`call()`都是立即执行

## 最后分别手动实现

1. `call()`函数

```
Function.prototype.myCall = function(context, ...args) {
    if(typeof context === 'object') {
        context = context || window
    } else {
        context = Object.create(null)
    }

    let fn = Symbol() // 保证fn的唯一性

    context[fn] = this

    let result = context[fn](...args)

    delete context[fn]

    return result
}

```
2. `apply()`函数

```
Function.prototype.myApply = function(context, arg) {
    if(typeof context === 'object') {
        context = context || window
    } else {
        context = Object.create(null)
    }
    let fn = Symbol() // 保证fn的唯一性

    context[fn] = this

    let result = context[fn](...args)
    
    delete context[fn]

    return result
}

```
3. `bind()`函数

```
Function.prototype.myBind = function(context) {
    const that = this
    const args = [...arguments].slice(1)
    const tempFun = function() {}
    const resultFun = function() {
        const nextArgs = [...arguments]
        return that.apply(this instanceof resultFun ? this : context, [...args, ...nextArgs])
    }
    tempFun.prototype = that.prototype
    resultFun.prototype = new tempFun()
    return resultFun
}

```
