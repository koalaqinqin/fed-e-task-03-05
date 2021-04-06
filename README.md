<!--
 * @Author: your name
 * @Date: 2021-04-06 20:58:08
 * @LastEditTime: 2021-04-06 22:37:10
 * @LastEditors: your name
 * @Description: In User Settings Edit
 * @FilePath: /fe-task/fed-e-task-03-05/README.md
-->
## 1、Vue 3.0 性能提升主要是通过哪几方面体现的？

答：

    1. 使用 Proxy 对象重写了响应式系统
    2. 通过优化编译的过程，和重写虚拟 DOM 提升渲染性能
    3. 优化源码的体积以及 tree-sharking：移除一些可替代的不常用的 api，比如 filter；

## 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？

答：Options Api中，我们使用 data、methods、props等组件选项来组成创建一个组件所需的对象，同一个功能模块的代码被拆分到不同的选项中，查看某个功能需要查看各个选项中的代码，并且当我们需要添加某个功能的时候，需要在各个选项中去添加我们的代码。Composition Api 可以让我们更灵活地组织和重用组件的逻辑。把主要功能封装到函数内部，通过 setup 来使用对应的功能函数。vue3 是使用 ref 来实现 Composition Api 的。

## 3、Proxy 相对于 Object.defineProperty 有哪些优点？

答：Proxy 可以一次性监听对象的所有属性；Proxy 可以直接监听数组的变化；Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而 Object.defineProperty 只能遍历对象属性直接修改；Proxy 有多达 13 种拦截方法,不限于 apply、ownKeys、deleteProperty、has 等等是 Object.defineProperty 不具备的；

## 4、Vue 3.0 在编译方面有哪些优化？

答：提升了首次渲染和更新的性能。vue3 新增了 Fragments，模板不需要唯一根节点，可以放多个同级标签，或者直接放文本内容。vue2 中只是标记静态根节点来优化 diff，而 vue3 是标记和提升所有的静态节点，diff 的时候只需要对比动态节点，另外 vue3 还缓存了事件处理函数，减少了不必要的更新操作。

## 5、Vue.js 3.0 响应式系统的实现原理？

答：Vue3.0 使用 Proxy 来重写响应式系统。这个系统主要由 rective/effect/track/trigger/ref/toRefs/computed 几个核心函数构成。

1. reactive(target)
    - 接收的参数 target 不是对象会直接返回
    - 创建一个拦截器对象 handler, 设置 get/set/deleteProperty。
        - get 用于收集依赖，并判断key 是否是对象，如果是对象那么需要继续为这个 key 创建 handler
        - set 用于判断新旧值是否相等，不相等就触发更新（trigger）
        - deleteProperty 用于删除 key，并触发更新
    - 最后返回一个 Proxy 对象(响应式对象)
2. effect(callback): 访问响应式对象属性，去收集依赖
3. track(target, key)
    - 如果没有 activeEffect，则说明没有创建 effect 依赖，直接返回
    - 如果有 activeEffect，就去判断 targetMap = new WeakMap() 对象 是否有 target 属性，没有就去 targetMap.set(target, (depsMap = new Map()))；有就去判断 targetMap 中是否有 key 属性，如果没有就去 depsMap.set(key, (dep = new Set()))，如果有就去添加这个 activeEffect
4. trigger(target, key)
    - 判断 depsMap 中是否有 target 属性
        - depsMap 中没有 target 属性，则没有 target 相应的依赖，直接返回
        - depsMap 中有 target 属性，则判断 target 属性的 map 值中是否有 key 属性，有的话循环触发收集的 effect()
5. ref(raw)
    - 如果 raw 是ref 创建的对象，直接返回
    - 普通对象的话，就会去调用 reactive 来创建响应式对象
6. toRefs(proxy)
    - 判断 proxy 是不是 reactive 创建的对象
    - 把 reactive 返回的 proxy 对象的每一个属性都转换成一个对象，因为对象都是引用类型，所以每个属性都会是响应式的。
7. computed(getter)
    - 有返回值的函数(getter)作为参数，这个函数的返回值就是计算属性的值(result.value)，并且需要监听这个函数内部响应式数据的变化（通过 effect 来实现）