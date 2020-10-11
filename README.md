## 1. Vue 3.0 性能提升主要是通过哪几个方面体现的？

- 响应式系统的升级，由ES5的 `Object.defineProperty` 升级到 ES6 的 `Proxy`，代理效率得到提升
- 编译过程的优化，标记和提升了所有的静态节点，`diff` 的 `patch` 过程只针对动态节点内容，效率更高
- 源码体积的优化，设计上兼容 `Tree-Shaking`，模块按需加载，构建体积更小

## 2. Vue 3.0 所采用的 Composition API 与 Vue 2.x 使用的 Options API 有什么区别？

- `Composition API` 是 Vue 3.0 新增的一组基于函数的 API，可以解决大型应用下逻辑分散的缺点
- `Composition API` 相比于 `Options API` 代码组织形式更容易维护和阅读
- `Composition API` 有利于提取和封装公共逻辑

## 3. Proxy 相对于 Object.defineProperty 有哪些优点？

- `Proxy` 写法更加优雅
- `Proxy` 可以监听新增属性
- `Proxy` 可以监听删除属性
- `Proxy` 可以监听数组索引和length，无需自己实现方法
- `Proxy` 代理的是对象，而 `Object.defineProperty` 代理的是对象的属性
- `Proxy` 是 ES6 特性，性能由浏览器负责优化，而 `Object.defineProperty` 是 ES5 特性，性能自己优化

## 4. Vue 3.0 在编译方面有哪些优化？

- 支持fragment
- 缓存事件处理函数
- 静态节点提升，Vue 2.x 中通过标记静态根节点优化diff过程，而Vue 3.0 标记所有静态节点，diff过程时只检查动态节点，提升了速度

## 5. Vue 3.0 响应式系统的实现原理？

Vue3 使用 Proxy 对象重写响应式系统，这个系统主要有以下几个函数来组合完成的：

### reactive:

- 接收一个参数，判断这参数是否是对象。不是对象则直接返回这个参数，不做响应式处理
- 创建拦截器对象 handler, 设置 get/set/deleteProperty

```
get
    收集依赖（track）
    返回当前 key 的值。
    如果当前 key 的值是对象，则为当前 key 的对象创建拦截器 handler, 设置 get/set/deleteProperty
    如果当前的 key 的值不是对象，则返回当前 key 的值
```

```
set
    设置的新值和老值不相等时，更新为新值，并触发更新（trigger）
```

```
deleteProperty
    当前对象有这个 key 的时候，删除这个 key 并触发更新（trigger）
```

- 返回 Proxy 对象

### effect

接收一个函数作为参数。作用是：访问响应式对象属性时去收集依赖

### track

- 接收两个参数：target 和 key
- 如果没有 activeEffect，则说明没有创建 effect 依赖
- 如果有 activeEffect，则去判断 WeakMap 集合中是否有 target 属性，

```
WeakMap 集合中没有 target 属性，则 set(target, (depsMap = new Map()))
WeakMap 集合中有 target 属性，则判断 target 属性的 map 值的 depsMap 中是否有 key 属性
depsMap 中没有 key 属性，则 set(key, (dep = new Set()))
depsMap 中有 key 属性，则添加这个 activeEffect
```

### trigger

- 判断 WeakMap 中是否有 target 属性

```
WeakMap 中没有 target 属性，则没有 target 相应的依赖
WeakMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()
