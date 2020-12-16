# defineProperty与Proxy
---

## 简述介绍
 `defineProperty`是`ES5`对象`Object`的`api`，而`Proxy`则是`ES6`实现的的`api`都是可以实现数据监听与劫持，`Proxy`的表面意思上更像是一种代理的意思，它比`ES5`的 `defineProperty`更友好的监听对象中的属性，它代理的是整个对象，实现的监听，而`defineProperty`是遍历对象的每个属性，而且还不能监听数组的变化。`Vue`2.x的数据绑定就是用`defineProperty()`实现的，现在`Vue`3.x版本也是改用了`Proxy`来实现，所以就更深层次理解下这两个`api`。

## `Object.defineProperty()`
引用`MDN`上的相关介绍该方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

`Object.defineProperty(obj, prop, descriptor)`

* 参数`obj`表示定义的对象，
* 参数`prop`表示要定义或修改的属性名称
* 参数`descriptor`表示要定义或修改的属性的相关描述符

相关的属性描述符分别为`configurable`、`enumerable`、`value` 、 `writable`、`get` 、`set`这里就不一一单的说明了，重点还是`set`、`get`这两个。更详细的[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)上有很详细的介绍文。
然后对数据的劫持都是由对象属性触发`getter`与`setter`,以及对象的属性发生变化的时候就行特定的操作。
<img src ="https://raw.githubusercontent.com/jetBn/blog/master/assets/md_images/definproperypng.png"/>

# Proxy
Proxy就像名称一样理解成代理的意思，它其实就在目标对象之前架设了一层“拦截”，就是访问该对象都必须通过该拦截。因此提供了一种机制，可以对目标对象访问进行过滤和改写。

```const proxyObj = new Proxy(target, handler)```
* 参数`target`表示要使用`Proxy`包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
* `handler`一个通常以函数作为属性的对象，各属性中的函数分别定义了再执行各种操作时代理`proxyObj`的行为。

相关`handler`对象的方法有`set`、`get`、`apply`、`definePropery`、`has`、`construct`等等，更详细的[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy#handler_%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%96%B9%E6%B3%95)上有详细的介绍。我们主要是对比与`Object.defineProperty`的数据劫持。

<img src ="https://raw.githubusercontent.com/jetBn/blog/master/assets/md_images/proxypng.png"/>


# 总结
`Proxy`与`Object.defineProperty()`都能实现对数据劫持，对比而下而`Proxy`更优，它可以直接监听对象而非属性，可以直接监听数组的变化，有很多的拦截方法，实例返回的是一个新对象，操作这个对象就可以实现对原对象数据的改变。而`Object.defineProperty()`只能遍历对象的属性进行修改操作。

