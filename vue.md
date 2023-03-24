# Vue基础

## 1. Vue的基本原理

当一个Vue实例创建时，Vue会遍历data中的属性，用 **Object.defineProperty**（vue3.0使用proxy ）将它们转为 getter/setter，并且在内部追踪相关依赖，在属性被访问和修改时通知变化。 每个组件实例都有相应的 **watcher 程序实例**，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的setter被调用时，会通知watcher重新计算，从而致使它关联的组件得以更新。

![Snipaste_2022-10-07_10-07-50.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2369052cf3e244d5ac21c9505da97259~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 1.2 Vue的优点

- 轻量级框架：只关注视图层，是一个构建数据的视图集合，大小只有几十 `kb` ；
- 简单易学：国人开发，中文文档，不存在语言障碍 ，易于理解和学习；
- 双向数据绑定：保留了 `angular` 的特点，在数据操作方面更为简单；
- 组件化：保留了 `react` 的优点，实现了 `html` 的封装和重用，在构建单页面应用方面有着独特的优势；
- 视图，数据，结构分离：使数据的更改更为简单，不需要进行逻辑代码的修改，只需要操作数据就能完成相关操作；
- 虚拟DOM：`dom` 操作是非常耗费性能的，不再使用原生的 `dom` 操作节点，极大解放 `dom` 操作，但具体操作的还是 `dom` 不过是换了另一种方式；
- 运行速度更快：相比较于 `react` 而言，同样是操作虚拟 `dom`，就性能而言， `vue` 存在很大的优势。

## 2. Vue响应式的原理

### **2.1 什么是数据劫持**

数据劫持比较好理解，通常我们利用`Object.defineProperty`劫持对象的访问器，在属性值发生变化时我们可以获取变化，从而进行进一步操作。

### **2.2 发布者模式 / 订阅者模式**

在软件架构中，**发布订阅**是一种消息范式，消息的发送者（称为发布者）不会将消息直接发送给特定的接收者（称为订阅者）。而是将发布的消息分为不同的类别，无需了解哪些订阅者（如果有的话）可能存在。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者（如果有的话）存在。

这里很明显了，区别就在于，不同于观察者和被观察者，**发布者和订阅者是互相不知道对方的存在的，发布者只需要把消息发送到订阅器里面，订阅者只管接受自己需要订阅的内容**

### **2.3 响应式原理**

Vue响应式的原理就是采用**数据劫持**结合**发布者-订阅者模式**的方式，通过**Object.defineProperty()** 来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。主要分为以下几个步骤：

**Observe**(被劫持的数据对象) **Compile**(vue的编译器) **Wather**(订阅者) **Dep**(用于收集Watcher订阅者们)

1.需要给**Observe**的数据对象进行递归遍历，包括子属性对象的属性，都加上**setter**和**getter**这样的属性，给这个对象的某个值赋值，就会触发**setter**，那么就能监听到了数据变化。

2.**Compile**解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图

3.**Watcher**订阅者是**Observer**和**Compile**之间通信的桥梁，主要做的事情是: ①在自身实例化时往属性订阅器(**Dep**)里面添加自己 ②自身必须有一个**update**()方法 ③待属性变动**dep.notice()** 通知时，能调用自身的**update()** 方法，并触发**Compile**中绑定的回调，则功成身退。

4.MVVM作为数据绑定的入口，整合**Observer**、**Compile**和**Watcher**三者，通过**Observer**来监听自己的**model**数据变化，通过**Compile**来解析编译模板指令，最终利用**Watcher**搭起**Observer**和**Compile**之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果。

![Snipaste_2022-10-07_10-09-27.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8b6808985354109a11d2ce29c6903de~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 3. Object.defineProperty的使用方式，有什么缺点

**使用方法**

**Object.defineProperty( obj, prop, descriptor )**

三个参数:

**obj** 要定义的对象

**prop** 要定义或修改的属性名称或 Symbol

**descriptor** 要定义或修改的属性描述符(配置对象)

属性描述符(配置对象)

**get** 属性的 getter 函数，如果没有 getter，则为 `undefined`。当访问该属性时，会调用此函数。执行时不传入任何参数，但是会传入 `this` 对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。该函数的返回值会被用作属性的值。 **默认为 [`undefined`]** **set** 属性的 setter 函数，如果没有 setter，则为 `undefined`。当属性值被修改时，会调用此函数。该方法接受一个参数（也就是被赋予的新值），会传入赋值时的 `this` 对象。 **默认为 [`undefined`]**

**缺点：**

在对一些属性进行操作时，使用这种方法无法拦截，比如**通过下标方式修改数组**数据或者**给对象新增属性**，这都不能触发组件的重新渲染，因为 Object.defineProperty 不能拦截到这些操作。更精确的来说，对于数组而言，大部分操作都是拦截不到的，只是 Vue 内部通过重写函数的方式解决了这个问题。

在 Vue3.0 中已经不使用这种方式了，而是通过使用 Proxy 对对象进行代理，从而实现数据劫持。使用Proxy 的好处是它可以完美的监听到任何方式的数据改变，唯一的缺点是兼容性的问题，因为 Proxy 是 ES6 的语法。

### 3.2 Object.defineProperty(target, key, options)，options可传什么参数？

- value：给target[key]设置初始值
- get：调用target[key]时触发
- set：设置target[key]时触发
- writable：规定target[key]是否可被重写，默认false
- enumerable：规定了key是否会出现在target的枚举属性中，默认为false
- configurable：规定了能否改变options，以及删除key属性，默认false，具体详细请看

## 4. MVVM、MVC、MVP的区别

### **（1）MVC**

**M: model数据模型, V:view视图模型, C: controller控制器**

MVC 通过分离 Model、View 和 Controller 的方式来组织代码结构。其中 View 负责页面的显示逻辑，Model 负责存储页面的业务数据，以及对相应数据的操作。并且 View 和 Model 应用了**观察者模式**，当 Model 层发生改变的时候它会**通知**有关 View 层更新页面。Controller 层是 View 层和 Model 层的**纽带**，它主要负责用户与应用的响应操作，当用户与页面产生交互的时候，Controller 中的事件触发器就开始工作了，通过调用 Model 层，来完成对 Model 的修改，然后 Model 层再去通知View视图更新。

![Snipaste_2022-10-07_10-15-46.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22f6dd1b8a8a4ec293ed91325ebbaa1b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### **（2）MVP**

**M: model数据模型, V:view视图模型, P: Presenter 控制器**

MVP 模式与 MVC 唯一不同的在于 Presenter 和 Controller。在 MVC 模式中使用观察者模式，来实现当 Model 层数据发生变化的时候，通知 View 层的更新。这样 View 层和 Model 层耦合在一起，当项目逻辑变得复杂的时候，可能会造成代码的混乱，并且可能会对代码的复用性造成一些问题。

MVP 的模式通过使用 Presenter 来实现对 View 层和 Model 层的**解耦**。MVC 中的Controller 只知道 Model 的接口，因此它没有办法控制 View 层的更新，MVP 模式中，View 层的接口暴露给了 Presenter 因此可以在 Presenter 中将 Model 的变化和 View 的变化**绑定**在一起，以此来实现 View 和 Model 的**同步更新**。这样就实现了对 View 和 Model 的解耦，Presenter 还包含了其他的响应逻辑。

### **（3）MVVM**

MVVM 分为 Model、View、ViewModel：

- **Model代表数据模型，数据和业务逻辑都在Model层中定义；**
- **View代表UI视图，负责数据的展示；**
- **ViewModel负责监听Model中数据的改变并且控制视图的更新，处理用户交互操作；**

Model和View并无直接关联，而是通过ViewModel来进行联系的，Model和ViewModel之间有着**双向数据绑定**的联系。因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model中同步。

这种模式实现了 Model和View的**数据自动同步**，因此开发者只需要专注于数据的维护操作即可，而不需要自己操作DOM。

![Snipaste_2022-10-07_10-12-52.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/deffd15d0d9647f599de5af394309ff8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 4.2 **MVVM**的优缺点?

优点:

- 分离视图（View）和模型（Model），降低代码耦合，提⾼视图或者逻辑的重⽤性: ⽐如视图（View）可以独⽴于Model变化和修改，⼀个ViewModel可以绑定不同的"View"上，当View变化的时候Model不可以不变，当Model变化的时候View也可以不变。你可以把⼀些视图逻辑放在⼀个ViewModel⾥⾯，让很多view重⽤这段视图逻辑
- 提⾼可测试性: ViewModel的存在可以帮助开发者更好地编写测试代码
- ⾃动更新dom: 利⽤双向绑定,数据更新后视图⾃动更新,让开发者从繁琐的⼿动dom中解放

缺点:

- Bug很难被调试: 因为使⽤双向绑定的模式，当你看到界⾯异常了，有可能是你View的代码有Bug，也可能是Model的代码有问题。数据绑定使得⼀个位置的Bug被快速传递到别的位置，要定位原始出问题的地⽅就变得不那么容易了。另外，数据绑定的声明是指令式地写在View的模版当中的，这些内容是没办法去打断点debug的
- ⼀个⼤的模块中model也会很⼤，虽然使⽤⽅便了也很容易保证了数据的⼀致性，当时⻓期持有，不释放内存就造成了花费更多的内存
- 对于⼤型的图形应⽤程序，视图状态较多，ViewModel的构建和维护的成本都会⽐较⾼

## 5. Vue的常用指令及作用

- v-on 给标签绑定函数，可以缩写为@，例如绑定一个点击函数 函数必须写在methods里面
- v-bind 动态绑定 作用： 及时对页面的数据进行更改, 可以简写成：冒号
- v-slot: 缩写为#, 组件插槽
- v-for 根据数组的个数, 循环数组元素的同时还生成所在的标签
- v-show 显示内容
- v-if 显示与隐藏
- v-else 必须和v-if连用 不能单独使用 否则报错
- v-text 解析文本
- v-html 解析html标签

### 5.2 Vue怎么动态绑定Class 与 Style

```
v-bind:class="{{ '类名': bool, '类名': bool ......}}"
v-bind:class="{ '类名': bool, '类名': bool ......}"
如果值为true 该类样式就会被应用在元素身上, false则不会
注意点：如果类名有 - ，则需要使用引号包起来
```

`v-bind:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。CSS property 名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用引号括起来) 来命名

直接绑定到一个样式对象通常更好，这会让模板更清晰：

```css
<div v-bind:style="styleObject"></div>
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
复制代码
```

## 6. vue常用的修饰符

**v-on**

- `.stop` - 调用 `event.stopPropagation()`。 阻止默认事件
- `.prevent` - 调用 `event.preventDefault()`。阻止默认行为
- `.native` - 监听组件根元素的原生事件。

**v-bind**

- `.prop` - 作为一个 DOM property 绑定而不是作为 attribute 绑定。
- `.camel` - (2.1.0+) 将 kebab-case attribute 名转换为 camelCase。(从 2.1.0 开始支持)
- `.sync` (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 `v-on` 侦听器。

**v-model**

- [`.lazy`]- 取代 `input` 监听 `change` 事件
- [`.number`] - 输入字符串转为有效的数字
- [`.trim`] - 输入首尾空格过滤

## 7. Vue的内置组件

### **component**

渲染一个“元组件”为动态组件。依 is 的值，来决定哪个组件被渲染

在一个多标签的界面中使用 is attribute 来切换不同的组件：tap栏切换

### **transition**

用于在 Vue 插入、更新或者移除 DOM 时， 提供多种不同方式的应用过渡、动画效果。

### **transition-group**

`<transition-group>` 用于给列表统一设置过渡动画。

### **keep-alive**

- 主要用于保留组件状态或避免组件重新渲染。
- **include** 属性用于指定哪些组件会被缓存，具有多种设置方式。
- **exclude** 属性用于指定哪些组件不会被缓存。
- **max** 属性用于设置最大缓存个数。

### **slot**

- **name** - string，用于命名插槽。
- < slot> 元素作为组件模板之中的内容分发插槽。< slot> 元素自身将被替换。

## 8. v-if、v-show、v-html 的原理

`v-if`会调用addIfCondition方法，生成vnode的时候会忽略对应节点，render的时候就不会渲染；

`v-show`会生成vnode，render的时候也会渲染成真实节点，只是在render过程中会在节点的属性中修改show属性值，也就是常说的display；

`v-html`会先移除节点下的所有节点，调用html方法，通过addProp添加innerHTML属性，归根结底还是设置innerHTML为v-html的值。

## 9. v-show和v-if的区别

- v-show和v-if的区别? 分别说明其使用场景?
- 相同点： v-show 和v-if都是true的时候显示，false的时候隐藏
- 不同点1：原理不同
  - `v-show`:一定会渲染，只是修改display属性
  - `v-if`:根据条件渲染
- 不同点2：应用场景不同
  - 频繁切换用v-show,不频繁切换用v-if

`v-if` 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建，操作的实际上是dom元素的创建或销毁。

`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换 它操作的是display:none/block属性。

一般来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。

## 10. 为什么避免v-for和v-if在一起使用?

Vue 处理指令时，v-for 比 v-if 具有更高的`优先级`, 虽然用起来也没报错好使, 但是性能不高, 如果你有5个元素被v-for循环, v-if也会分别执行5次.

## 11. v-for 循环为什么一定要绑定key ?

> 提升vue渲染性能

- 1.vue在渲染的时候,会 先把 新DOM 与 旧DOM 进行对比， 如果dom结构一致，则vue会复用旧的dom。 （此时可能造成数据渲染异常）
- 2.使用key可以给dom添加一个标识符，让vue强制更新dom

比如有一个列表 li1 到 li4，我们需要在中间插入一个li3，`li1` 和 `li2` 不会重新渲染，而 `li3、li4、li5` 都会重新渲染

因为在不使用 `key` 或者列表的 `index` 作为 `key` 的时候，每个元素对应的位置关系都是 index，直接导致我们插入的元素到后面的全部元素，对应的位置关系都发生了变更，所以全部都会执行更新操作, 这是不可取的

而在使用唯一 `key` 的情况下，每个元素对应的位置关系就是 `key`，来看一下使用唯一 `key` 值的情况下

这样如图中的 `li3` 和 `li4` 就不会重新渲染，因为元素内容没发生改变，对应的位置关系也没有发生改变。

这也是为什么 v-for 必须要写 key，而且不建议开发中使用数组的 index 作为 key 的原因

总结一下：

- key 的作用主要是为了更高效的更新虚拟 DOM，因为它可以非常精确的找到相同节点，因此 patch 过程会非常高效
- Vue 在 patch 过程中会判断两个节点是不是相同节点时，key 是一个必要条件。比如渲染列表时，如果不写 key，Vue 在比较的时候，就可能会导致频繁更新元素，使整个 patch 过程比较低效，影响性能
- 应该避免使用数组下标作为 key，因为 key 值不是唯一的话可能会导致上面图中表示的 bug，使 Vue 无法区分它他，还有比如在使用相同标签元素过渡切换的时候，就会导致只替换其内部属性而不会触发过渡效果
- 从源码里可以知道，Vue 判断两个节点是否相同时主要判断两者的元素类型和 key 等，如果不设置 key，就可能永远认为这两个是相同节点，只能去做更新操作，就造成大量不必要的 DOM 更新操作，明显是不可取的

### 11.2 为什么不建议用index索引作为key?

使用index 作为 key和没写基本上没区别，因为不管数组的顺序怎么颠倒，index 都是 0, 1, 2...这样排列，导致 Vue 会复用错误的旧子节点，做很多额外的工作。

## 12. v-model 是如何实现的，语法糖实际是什么？

**（1）作用在表单元素上** 动态绑定了 input 的 value 指向了 messgae 变量，并且在触发 input 事件的时候去动态把 message设置为目标值：

![Snipaste_2022-08-11_20-28-07.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f554ba199884e8d8eb96d04d97c9db1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**（2）作用在组件上** 在自定义组件中，v-model 默认会利用名为 value 的 prop和名为 input 的事件

**本质是一个父子组件通信的语法糖，通过prop和$.emit实现。** 因此父组件 v-model 语法糖本质上可以修改为：

![Snipaste_2022-08-11_20-28-55.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8de06e44a2046afaa182a42e1a3b033~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在组件的实现中，可以通过 v-model属性来配置子组件接收的prop名称，以及派发的事件名称。 例子：

![Snipaste_2022-08-11_20-29-31.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7215208de2124bfbbfb2f617e473507f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

默认情况下，一个组件上的v-model 会把 value 用作 `prop`且把 input 用作 `event`。但是一些输入类型比如单选框和复选框按钮可能想使用 value prop 来达到不同的目的。使用 model 选项可以回避这些情况产生的冲突。js 监听input 输入框输入数据改变，用oninput，数据改变以后就会立刻出发这个事件。通过input事件把数据`$emit` 出去，在父组件接受。父组件设置v-model的值为input `$emit`过来的值。

## 13. v-model 可以被用在自定义组件上吗？如果可以，如何使用？

**可以**。v-model 实际上是一个语法糖，用在自定义组件上也是同理：

![Snipaste_2022-10-07_10-19-44.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f5b193577414a5a9a3d3271fe3789a0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

相当于

![Snipaste_2022-10-07_10-19-52.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0601cc2b0964583bd41a63dd009aefe~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

显然，custom-input 与父组件的交互如下：

1. 父组件将`searchText`变量传入custom-input 组件，使用的 prop 名为`value`；
2. custom-input 组件向父组件传出名为`input`的事件，父组件将接收到的值赋值给`searchText`；

所以，custom-input 组件的实现应该类似于这样：

![Snipaste_2022-10-07_10-20-35.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89a368414d114e82b31d44dea7c6410b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 14. v-model和.sync的对比

v-model与.sync的共同点：都是`语法糖`，都可以实现父子组件中的数据的双向通信。

v-model与.sync的不共同点：

**v-model：**

1.父组件 v-model="" 子组件 @(input,value)

2.一个组件只能绑定`一个`v-model

3.v-model针对更多的是最终操作结果，是`双向绑定的结果`，是`value`，是一种`change操作`。

**.sync：**

1.父组件 :my-prop-name.sync 子组件@update:my-prop-name 的模式来替代事件触发，实现父子组件间的双向绑定。

2.一个组件可以`多个`属性用.sync修饰符，可以同时"双向绑定多个“prop”

3..sync针对更多的是各种各样的状态，是`状态的互相传递`，是`status`，是一种`update操作`。

## 15. 计算属性computed 和watch 的区别是什么？

#### **`Computed`：**

- 它**支持缓存**，只有依赖的数据发生了变化，才会重新计算
- **不支持异步**，当Computed中有异步操作时，无法监听数据的变化
- computed的值会**默认走缓存**，计算属性是基于它们的响应式依赖进行缓存的，也就是基于data声明过，或者父组件传递过来的props中的数据进行计算的。
- **如果一个属性是由其他属性计算而来的，这个属性依赖其他的属性，一般会使用computed**
- 如果computed属性的属性值是函数，那么**默认使用get方法**，函数的返回值就是属性的属性值；在computed中，属性有一个get方法和一个set方法，当数据发生变化时，会调用set方法。

#### **`Watch`：**

- 它**不支持缓存**，数据变化时，它就会触发相应的操作
- **支持异步**监听
- 监听的函数接收**两个参数**，第一个参数是最新的值，第二个是变化之前的值
- 当一个属性发生变化时，就需要执行相应的操作
- 监听数据必须是data中声明的或者父组件传递过来的props中的数据，当发生变化时，会触发其他操作，函数有两个的参数：
  - **immediate**：组件加载立即触发回调函数
  - **deep**：深度监听，发现数据内部的变化，在复杂数据类型中使用，例如数组中的对象发生变化。需要注意的是，deep无法监听到数组和对象内部的变化。

当想要执行异步或者昂贵的操作以响应不断的变化时，就需要使用watch。

**总结：**

- `computed` 计算属性 : 依赖其它属性值，并且 computed 的值有缓存，只有它依赖的属性值发生改变，下一次获取 computed 的值时才会重新计算 computed 的值。
- `watch` 侦听器 : 更多的是**观察**的作用，**无缓存性**，类似于某些数据的监听回调，每当监听的数据变化时都会执行回调进行后续操作。

**运用场景：**

- 当需要进行数值计算,并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时都要重新计算。
- 当需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许执行异步操作 ( 访问一个 API )，限制执行该操作的频率，并在得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

## 16. Computed 和 Methods 的区别

可以将同一函数定义为一个 method 或者一个计算属性。对于最终的结果，两种方式是相同的

**不同点：**

- `computed`: 计算属性是基于它们的依赖进行缓存的，只有在它的相关依赖发生改变时才会重新求值；
- `method` 调用总会执行该函数。

## 17. 什么是组件

**组件就是把图形、非图形的各种逻辑均抽象为一个`统一的概念`（组件）来实现开发的模式，在`Vue`中每一个`.vue`文件都可以视为一个组件**

#### **组件的优势**

1. 降低整个系统的耦合度，在保持接口不变的情况下，我们可以替换不同的组件快速完成需求
2. 调试方便，由于整个系统是通过组件组合起来的，在出现问题的时候，可以用排除法直接移除组件，或者根据报错的组件快速定位问题，之所以能够快速定位，是因为每个组件之间低耦合，职责单一，所以逻辑会比分析整个系统要简单
3. 提高可维护性，由于每个组件的职责单一，并且组件在系统中是被复用的，所以对代码进行优化可获得系统的整体升级

## 18. 什么是插件

**插件通常用来为 `Vue` 添加全局功能**。插件的功能范围没有严格的限制——一般有下面几种：

- 添加全局方法或者属性。如: `vue-custom-element`
- 添加全局资源：指令/过滤器/过渡等。如 `vue-touch`
- 添加全局公共组件 Vue.component()
- 添加全局公共指令 Vue.directive()
- 通过全局混入来添加一些组件选项。如`vue-router`
- 添加 `Vue` 实例方法，通过把它们添加到 `Vue.prototype` 上实现。
- 一个库，提供自己的 `API`，同时提供上面提到的一个或多个功能。如`vue-router`

### 18.2 Vue2和Vue3怎么注册全局组件

Vue2使用 `Vue.component('组件名'，组件对象)`

Vue3使用

```scss
const app = createApp(App)
app.component('组件名'，组件对象)
复制代码
```

### 18.3 Vue2、Vue3怎么封装自定义插件并使用/ Vue.use() （install）

**Vue2**

在compoents.index.js里，定义一个函数或对象，在里面可以使用Vue.compoent全局注册组件，并暴露出去

在main.js里使用Vue.use( )，参数类型必须是 object 或 Function

**Vue3**

在compoents.index.ts里，定义一个函数或对象，在里面可以使用app.compoent全局注册组件，并暴露出去

在main.ts里使用app.use( )，参数类型必须是 object 或 Function

------

如果是 Function 那么这个函数就被当做 install 方法

如果是 object 则需要定义一个 install 方法

## 19. 组件通信/ 组件传值的方法

**什么是组件通信**

组件(`.vue`)通过某种方式来传递信息以达到某个目的

**组件通信解决了什么问题**

每个组件之间的都有独自的作用域，组件间的数据是无法共享的但实际开发工作中我们常常需要让组件之间共享数据，这也是组件通信的目的要让它们互相之间能进行通讯，这样才能构成完整系统

### **（1）props / $emit (父子)**

父组件通过`props`向子组件传递数据，子组件通过`$emit`和父组件通信

**1. 父组件向子组件传值**

`props`只能是父组件向子组件进行传值，`props`使得父子组件之间形成了一个单向下行绑定。子组件的数据会随着父组件不断更新。

`props` 可以显示定义一个或一个以上的数据，对于接收的数据，可以是各种数据类型，同样也可以传递一个函数。

`props`属性名规则：若在`props`中使用驼峰形式，模板中需要使用短横线的形式

![Snipaste_2022-10-07_10-23-21.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c606c1619f644618b686d4abfdd8e256~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![Snipaste_2022-10-07_10-23-33.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/428cbedd97c547a080eac446c0c81a18~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**2. 子组件向父组件传值**

- `$emit`绑定一个自定义事件，当这个事件被执行的时就会将参数传递给父组件，而父组件通过`v-on`监听并接收参数。

### （2）依赖注入 provide / inject（父子、祖孙）

这种方式就是Vue中的**依赖注入**，该方法用于**父子组件之间的通信**。当然这里所说的父子不一定是真正的父子，也可以是祖孙组件，在层数很深的情况下，可以使用这种方法来进行传值。就不用一层一层的传递了。

`provide / inject`是Vue提供的两个钩子，和`data`、`methods`是同级的。并且`provide`的书写形式和`data`一样。

- `provide` 钩子用来发送数据或方法
- `inject`钩子用来接收数据或方法

在父组件中：

![1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66f3f6174c2e40a3a48dc628ca15f1d3~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在子组件中：

![2.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f044d6d4dae4eb58da4154b1813658c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

还可以这样写，这样写就可以访问父组件中的所有属性：

![3.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/248a3d57941c4742a9a52e1fdcd47b7c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**注意：** 依赖注入所提供的属性是**非响应式**的。

### （3）ref / $refs （父子，兄弟）

`ref`： 这个属性用在子组件上，它的引用就指向了子组件的实例。可以通过实例来访问组件的数据和方法。

这种方式也是实现**兄弟组件**之间的通信。子组件1通过**this.`$emit`\**通知父组件调用函数，父组件的函数里用\**this.$refs**拿到子组件2的方法，这样就实现兄弟组件之间的通信。

在子组件中：

![4.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b9498b359f940e5bfffe143bb8b367f~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在父组件中：

![5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3e236198198d497d844f0c1c6a091921~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### （4）`$parent` / `$children` (父子)

- 使用`$parent`可以让组件访问父组件的实例（访问的是上一级父组件的属性和方法）
- 使用`$children`可以让组件访问子组件的实例，但是，`$children`并不能保证顺序，并且访问的数据也不是响应式的。

在子组件中：

![6.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b5bce028300843cba63a8588c822b3ae~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在父组件中：

![7.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1b0b23799bf4c55a5abf78488295543~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在上面的代码中，子组件获取到了父组件的`parentVal`值，父组件改变了子组件中`message`的值。 **需要注意：**

- 通过`$parent`访问到的是上一级父组件的实例，可以使用`$root`来访问根组件的实例
- 在组件中使用`$children`拿到的是所有的子组件的实例，它是一个数组，并且是无序的
- 在根组件`#app`上拿`$parent`得到的是`new Vue()`的实例，在这实例上再拿`$parent`得到的是`undefined`，而在最底层的子组件拿`$children`是个空数组
- `$children` 的值是**数组**，而`$parent`是个**对象**

### （5）`$attrs` / `$listeners` (祖孙)

考虑一种场景，如果A是B组件的父组件，B是C组件的父组件。如果想要组件A给组件C传递数据，这种隔代的数据，该使用哪种方式呢？

如果是用`props/$emit`来一级一级的传递，确实可以完成，但是比较复杂；如果使用事件总线，在多人开发或者项目较大的时候，维护起来很麻烦；如果使用Vuex，的确也可以，但是如果仅仅是传递数据，那可能就有点浪费了。

针对上述情况，Vue引入了`$attrs / $listeners`，实现组件之间的跨代通信。

先来看一下`inheritAttrs`，它的默认值true，继承所有的父组件属性除`props`之外的所有属性；`inheritAttrs：false` 只继承class属性 。

- `$attrs`：继承所有的父组件属性（除了prop传递的属性、class 和 style ），一般用在子组件的子元素上
- `$listeners`：该属性是一个对象，里面包含了作用在这个组件上的所有监听器，可以配合 `v-on="$listeners"` 将所有的事件监听器指向这个组件的某个特定的子元素。（相当于子组件继承父组件的事件）

A组件（`APP.vue`）：

![8.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/232791e5abba45b7895ee40944162b6c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

B组件（`Child1.vue`）：

![9.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e403a31494894a52b507e4f6ee115500~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

C 组件 (`Child2.vue`)：

![10.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb0a3c35039847a58fb6d8280bd2d5ff~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在上述代码中：

- C组件中能直接触发test的原因在于 B组件调用C组件时 使用 v-on 绑定了`$listeners` 属性
- 在B组件中通过v-bind 绑定`$attrs`属性，C组件可以直接获取到A组件中传递下来的props（除了B组件中props声明的）

### （6）eventBus事件总线（`$emit / $on`）（任意组件通信）

`eventBus`事件总线适用于**父子组件**、**非父子组件**等之间的通信，使用步骤如下：

**（1）创建事件中心管理组件之间的通信**

![11.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd9993c134b84a788b8bbf212c189dc0~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**（2）发送事件** 假设有两个兄弟组件`firstCom`和`secondCom`：

![12.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7711b36f31a7464f983c936df57bed2a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在`firstCom`组件中发送事件：

![13.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6c64ee6cf2b4c17a89ecf172fc08d02~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**（3）接收事件** 在`secondCom`组件中接收事件：

![14.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/809767d82f5e46ecbbeaa6eee42f5ee8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

在上述代码中，这就相当于将`num`值存贮在了事件总线中，在其他组件中可以直接访问。事件总线就相当于一个桥梁，不用组件通过它来通信。

虽然看起来比较简单，但是这种方法也有不变之处，如果项目过大，使用这种方式进行通信，后期维护起来会很困难。

### （7）总结

**（1）父子组件间通信**

- 子组件通过 props 属性来接受父组件的数据，然后父组件在子组件上注册监听事件，子组件通过 emit 触发事件来向父组件发送数据。
- 通过 ref 属性给子组件设置一个名字。父组件通过 `$refs` 组件名来获得子组件，子组件通过 `$parent` 获得父组件，这样也可以实现通信。
- 使用 provide/inject，在父组件中通过 provide提供变量，在子组件中通过 inject 来将变量注入到组件中。不论子组件有多深，只要调用了 inject 那么就可以注入 provide中的数据。

**（2）兄弟组件间通信**

- 使用 eventBus 的方法，它的本质是通过创建一个空的 Vue 实例来作为消息传递的对象，通信的组件引入这个实例，通信的组件通过在这个实例上监听和触发事件，来实现消息的传递。
- 通过 `$parent/$refs` 来获取到兄弟组件，也可以进行通信。

**（3）任意组件之间**

- 使用 eventBus ，其实就是创建一个事件中心，相当于中转站，可以用它来传递事件和接收事件。

如果业务逻辑复杂，很多组件之间需要同时处理一些公共的数据，这个时候采用上面这一些方法可能不利于项目的维护。这个时候可以使用 vuex ，vuex 的思想就是将这一些公共的数据抽离出来，将它作为一个全局的变量来管理，然后其他组件就可以对这个公共数据进行读写操作，这样达到了解耦的目的。

## 20. 子组件可以直接改变父组件的数据吗？

子组件不可以直接改变父组件的数据。这样做主要是为了维护父子组件的单向数据流。每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。如果这样做了，Vue 会在浏览器的控制台中发出警告。

Vue提倡单向数据流，即父级 props 的更新会流向子组件，但是反过来则不行。这是为了防止意外的改变父组件状态，使得应用的数据流变得难以理解，导致数据流混乱。如果破坏了单向数据流，当应用复杂时，debug 的成本会非常高。

**只能通过 `$emit` 派发一个自定义事件，父组件接收到后，由父组件修改。**

## 21. 生命周期

### (1). 说一下Vue的生命周期

vue 实例从创建到销毁的过程就是生命周期。

也就是从开始创建、初始化数据、编译模板、挂在 dom -> 渲染、更新 -> 渲染、准备销毁、销毁在等一系列过程

vue的声明周期常见的主要分为4大阶段8大钩子函数

**第一阶段：创建前 / 后**

**beforeCreate（创建前）** ：数据观测和初始化事件还未开始，此时 data 的响应式追踪、event/watcher 都还没有被设置，也就是说不能访问到data、computed、watch、methods上的方法和数据。

**created（创建后）** ：实例创建完成，实例上配置的 options 包括 data、computed、watch、methods 等都配置完成，但是此时渲染得节点还未挂载到 DOM，所以不能访问到 `$el` 属性。

**第二阶段: 渲染前 / 后**

**beforeMount（挂载前）** ：在挂载开始之前被调用，相关的render函数首次被调用。实例已完成以下的配置：编译模板，把data里面的数据和模板生成html。此时还没有挂载html到页面上。

**mounted（挂载后）** ：在el被新创建的 vm.$el 替换，并挂载到实例上去之后调用。实例已完成以下的配置：用上面编译好的html内容替换el属性指向的DOM对象。完成模板中的html渲染到html 页面中。此过程中进行ajax交互。

**第三阶段: 更新前 / 后**

**beforeUpdate（更新前）** ：响应式数据更新时调用，此时虽然响应式数据更新了，但是对应的真实 DOM 还没有被渲染。

**updated（更新后）** ：在由于数据更改导致的虚拟DOM重新渲染和打补丁之后调用。此时 DOM 已经根据响应式数据的变化更新了。调用时，组件 DOM已经更新，所以可以执行依赖于DOM的操作。然而在大多数情况下，应该避免在此期间更改状态，因为这可能会导致更新无限循环。该钩子在服务器端渲染期间不被调用。

**第四阶段: 销毁前 / 后**

**beforeDestroy（销毁前）** ：实例销毁之前调用。这一步，实例仍然完全可用，`this` 仍能获取到实例。

**destroyed（销毁后）** ：实例销毁后调用，调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务端渲染期间不被调用。

**另外三个生命周期函数不常用**

另外还有 `keep-alive` 独有的生命周期，分别为 `activated` 和 `deactivated` 。用 `keep-alive` 包裹的组件在切换时不会进行销毁，而是缓存到内存中并执行 `deactivated` 钩子函数，命中缓存渲染后会执行 `activated` 钩子函数。

`errorCapured`钩子，当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 `false` 以阻止该错误继续向上传播。

![Snipaste_2022-08-11_21-23-58.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89deb213d6fe4b64b0724f1cb1e09978~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

![Snipaste_2022-08-11_21-24-17.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c77dcde01e14cfc9bcc0c1550d471df~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### (2). Vue 子组件和父组件执行顺序

**加载渲染过程：**

1. 父组件 beforeCreate
2. 父组件 created
3. 父组件 beforeMount
4. 子组件 beforeCreate
5. 子组件 created
6. 子组件 beforeMount
7. 子组件 mounted
8. 父组件 mounted

**更新过程：**

1. 父组件 beforeUpdate
2. 子组件 beforeUpdate
3. 子组件 updated
4. 父组件 updated

**销毁过程：**

1. 父组件 beforeDestroy
2. 子组件 beforeDestroy
3. 子组件 destroyed
4. 父组件 destoryed

### (3). created和mounted的区别

- `created`:在模板渲染成html前调用，即通常初始化某些属性值，然后再渲染成视图。
- `mounted`:在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作。

### (4). 一般在哪个生命周期请求异步数据

我们可以在钩子函数`created`、`beforeMount`、`mounted` 中进行调用，因为在这三个钩子函数中，data 已经创建，可以将服务端端返回的数据进行赋值。

推荐在 created 钩子函数中调用异步请求，因为在 created 钩子函数中调用异步请求有以下优点：

- 能更快获取到服务端数据，减少页面加载时间，用户体验更好；
- SSR不支持 beforeMount 、mounted 钩子函数，放在 created 中有助于一致性。

## 22. 组件缓存 keep-alive

**组件缓存**

组件的缓存可以在进行动态组件切换的时候对组件内部数据进行缓存,而不是走销毁流程

使用场景: 多表单切换,对表单内数据进行保存

**keep-alive**

包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。 当组件在`<keep-alive>`内被切换，它的`activated`和 `deactivated`这两个生命周期钩子函数将会被对应执行 。

### keep-alive的参数(include,exclude)

- include(包含): 名称匹配的组件会被缓存-->include的值为组件的name。
- exclude(排除): 任何名称匹配的组件都不会被缓存。
- max - 数量 决定最多可以缓存多少组件。

### **keep-alive的使用**

1. 搭配`<component></component>`使用
2. 搭配路由使用 ( 需配置路由meta信息的`keepAlive`属性 )
3. 清除缓存组件
   - 在组件跳转之前使用后置路由守卫判断组件是否缓存
   - ( beforeRouteLeave( to, from, next ){ from.meta.keepAlive = false }

### **keep-alive的两个钩子函数**

| activated                            | deactivated                      |
| ------------------------------------ | -------------------------------- |
| 在 `keep-alive` 组件激活时调用       | 在`keep-alive` 组件停用时调用    |
| 该钩子函数在服务器端渲染期间不被调用 | 该钩子在服务器端渲染期间不被调用 |



使用`keep-alive`会将数据保留在内存中，如果要在每次进入页面的时候获取最新的数据，需要在 `activated`阶段获取数据，承担原来`created`钩子函数中获取数据的任务。

**注意：** 只有组件被`keep-alive`包裹时，这两个生命周期函数才会被调用，如果作为正常组件使用，是不会被调用的

使用 exclude 排除之后，就算被包裹在 keep-alive 中，这两个钩子函数依然不会被调用！在服务端渲染时，此钩子函数也不会被调用。

设置了缓存的组件钩子调用情况：

第一次进入：beforeRouterEnter ->created->…->activated->…->deactivated> beforeRouteLeave

后续进入时：beforeRouterEnter ->activated->deactivated> beforeRouteLeave

### **keep-alive主要流程**

1. 判断组件 name ，不在 include 或者在 exclude 中，直接返回 vnode，说明该组件不被缓存。
2. 获取组件实例 key ，如果有获取实例的 key，否则重新生成。
3. key生成规则，cid +"∶∶"+ tag ，仅靠cid是不够的，因为相同的构造函数可以注册为不同的本地组件。
4. 如果缓存对象内存在，则直接从缓存对象中获取组件实例给 vnode ，不存在则添加到缓存对象中。 5.最大缓存数量，当缓存组件数量超过 max 值时，清除 keys 数组内第一个组件。

### **keep-alive 的实现**

![Snipaste_2022-08-11_21-33-22.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/846880f48f1e450198f2e28444d0f6a9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 23. 过滤器的作用，如何实现一个过滤器

根据过滤器的名称，过滤器是用来过滤数据的，在Vue中使用`filters`来过滤数据，`filters`不会修改数据，而是过滤数据，改变用户看到的输出（计算属性 `computed` ，方法 `methods` 都是通过修改数据来处理数据格式的输出显示）。

**使用场景：**

- 需要格式化数据的情况，比如需要处理时间、价格等数据格式的输出 / 显示。
- 比如后端返回一个 **年月日的日期字符串**，前端需要展示为 **多少天前** 的数据格式，此时就可以用`fliters`过滤器来处理数据。

过滤器是一个函数，它会把表达式中的值始终当作函数的第一个参数。过滤器用在**插值表达式 {{ }} 和 v-bind 表达式** 中，然后放在操作符“ `|` ”后面进行指示。

例如，在显示金额，给商品价格添加单位：

![15.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03181bad44724bb48bd1b651aa66851a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 24. 说说你对slot插槽的理解？slot使用场景有哪些？

### **slot是什么**

`slot`又名插槽，**是Vue的内容分发机制**，组件内部的模板引擎使用slot元素作为承载分发内容的出口。插槽slot是子组件的一个模板标签元素，而这一个标签元素是否显示，以及怎么显示是由父组件决定的。

通过插槽可以让用户可以拓展组件，去更好地复用组件和对其做定制化处理

通过`slot`插槽向组件内部指定位置传递内容，完成这个复用组件在不同场景的应用

比如布局组件、表格列、下拉选、弹框显示内容等

**分类**

### **默认插槽**

子组件用`<slot>`标签来确定渲染的位置，标签里面可以放`DOM`结构，当父组件使用的时候没有往插槽传入内容，标签内`DOM`结构就会显示在页面

父组件在使用的时候，直接在子组件的标签内写入内容即可

### **具名插槽**

子组件用`name`属性来表示插槽的名字，不传为默认插槽

父组件中在使用时在默认插槽的基础上加上`slot`属性，值为子组件插槽`name`属性值

### **作用域插槽**

子组件在作用域上绑定属性来将子组件的信息传给父组件使用，这些属性会被挂在父组件`v-slot`接受的对象上

父组件中在使用时通过`v-slot:`（简写：#）获取子组件的信息，在内容中使用

### **小结**

- `v-slot`属性只能在`<template>`上使用，但在只有默认插槽时可以在组件标签上使用
- 默认插槽名为`default`，可以省略default直接写`v-slot`
- 缩写为`#`时不能不写参数，写成`#default`
- 可以通过解构获取`v-slot={user}`，还可以重命名`v-slot="{user: newName}"`和定义默认值`v-slot="{user = '默认值'}"`

## 25. Vue为什么采用异步渲染呢？

`Vue` 是组件级更新，如果不采用异步更新，那么每次更新数据都会对当前组件进行重新渲染，所以为了性能，`Vue` 会在本轮数据更新后，在异步更新视图。核心思想`nextTick` 。

`dep.notify（）` 通知 watcher进行更新，`subs[i].update` 依次调用 watcher 的`update` ，`queueWatcher` 将watcher 去重放入队列， nextTick（`flushSchedulerQueue` ）在下一tick中刷新watcher队列（异步）。

## 25. $nextTick 原理及作用

其实一句话就可以把`$nextTick`这个东西讲明白：就是你放在`$nextTick`当中的操作不会立即执行，而是等数据更新、DOM更新完成之后再执行，这样我们拿到的肯定就是最新的了。

Vue的响应式并不是只数据发生变化之后，DOM就立刻发生变化，而是按照一定的策略进行DOM的更新。

DOM更新有两种选择，一个是在本次事件循环的最后进行一次DOM更新，另一种是把DOM更新放在下一轮的事件循环当中。Vue优先选择第一种，只有当环境不支持的时候才触发第二种机制。

虽然性能上提高了很多，但这个时候问题就出现了。我已经把数据改掉了，但是它的更新异步的，而我在获取的时候，它还没有来得及改，这个时候就需要用到nextTick

**原理：**

**Vue 的 nextTick 其本质是对 JavaScript 执行原理 `EventLoop` 的一种应用。**

- Vue2刚开始的时候, $nextTick是宏任务(setTimeout)，但是宏任务的性能太差。
- 后来改成了微任务Mutation Observer，但是还是有一些问题：
  - 速度太快了，在一些特殊场景下，DOM还没更新就去获取了
  - 兼容性不好，很多浏览器不支持
- 后来又更新成了微宏并行阶段：先判断是否支持Mutation Observer，如果支持就使用，否则使用宏任务
- Vue2.5版本之后，修复了微任务的那些问题，目前最新的$nextTick采用的是纯微任务。

由于Vue的DOM操作是异步的，所以，在上面的情况中，就要将DOM2获取数据的操作写在`$nextTick`中。

![16.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6276978f438f46bca595ef1b9e340578~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

所以，在以下情况下，会用到nextTick：

- **在数据变化后执行的某个操作，而这个操作需要使用随数据变化而变化的DOM结构的时候，这个操作就需要方法在`nextTick()`的回调函数中。**
- **在vue生命周期中，如果在created()钩子进行DOM操作，也一定要放在`nextTick()`的回调函数中。**

因为在created()钩子函数中，页面的DOM还未渲染，这时候也没办法操作DOM，所以，此时如果想要操作DOM，必须将操作的代码放在`nextTick()`的回调函数中。

## 26. 描述下Vue2的自定义指令

在 Vue2.0 中，代码复用和抽象的主要形式是组件。然而，有的情况下，你仍然需要对普通 DOM 元素进行底层操作，这时候就会用到自定义指令。 一般需要对DOM元素进行底层操作时使用，尽量只用来操作 DOM展示，不修改内部的值。当使用自定义指令直接修改 value 值时绑定v-model的值也不会同步更新；如必须修改可以在自定义指令中使用keydown事件，在vue组件中使用 change事件，回调中修改vue数据;

**（1）自定义指令基本内容**

- 全局定义：`Vue.directive("focus",{})`

- 局部定义：`directives:{focus:{}}`

- 钩子函数：指令定义对象提供钩子函数

  1. ```bash
      bind：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
     复制代码
     ```

  2. ```
      inSerted：被绑定元素插入父节点时调用（仅保证父节点存在，但不一定已被插入文档中）。
     复制代码
     ```

  3. ```sql
      update：所在组件的VNode更新时调用，但是可能发生在其子VNode更新之前调用。指令的值可能发生了改变，也可能没有。但是可以通过比较更新前后的值来忽略不必要的模板更新。
     复制代码
     ```

  4. ```
      ComponentUpdate：指令所在组件的 VNode及其子VNode全部更新后调用。
     复制代码
     ```

  5. ```
      unbind：只调用一次，指令与元素解绑时调用。
     复制代码
     ```

- 钩子函数的参数 ：

  1. el：指令所绑定的元素，可以用来直接操作 DOM
  2. bing： 一个对象，包含以下属性：
     - name: 指令名，不包括 v- 前缀。
     - value: 指令的绑定值， 例如： v-my-directive="1 + 1", value 的值是2。
     - oldValue: 指令绑定的前一个值，仅在 update 和 componentUpdated钩子中可用。无论值是否改变都可用。
     - expression: 绑定值的表达式或变量名。 例如 v-my-directive="1 + 1", expression 的值是 "1 + 1"。
     - arg: 传给指令的参数。例如 v-my-directive:foo， arg 的值是 "foo"。
     - modifiers: 一个包含修饰符的对象。 例如： v-my-directive.foo.bar, 修饰符对象 modifiers 的值是 { foo: true, bar: true }。
  3. vnode： 编译生成的虚拟节点
  4. oldVnode：上一个虚拟节点（更新钩子函数中才有用）

**（2）使用场景**

- 普通DOM元素进行底层操作的时候，可以使用自定义指令
- 自定义指令是用来操作DOM的。尽管Vue推崇数据驱动视图的理念，但并非所有情况都适合数据驱动。自定义指令就是一种有效的补充和扩展，不仅可用于定义任何的DOM操作，并且是可复用的。

**（3）使用案例**

初级应用：

- 鼠标聚焦
- 下拉菜单
- 相对时间转换
- 滚动动画

高级应用：

- 自定义指令实现图片懒加载
- 自定义指令集成第三方插件

## 27. 简述 mixin、extends 的覆盖逻辑

**（1）mixin 和 extends** mixin 和 extends均是用于合并、拓展组件的，两者均通过 mergeOptions 方法实现合并。

- mixins 接收一个混入对象的数组，其中混入对象可以像正常的实例对象一样包含实例选项，这些选项会被合并到最终的选项中。Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。
- extends 主要是为了便于扩展单文件组件，接收一个对象或构造函数。

![Snipaste_2022-08-12_09-13-37.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c1220980f0b4116a3a4db1991a7db8b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**（2）mergeOptions 的执行过程**

- 规范化选项（normalizeProps、normalizelnject、normalizeDirectives)
- 对未合并的选项，进行判断

![Snipaste_2022-08-12_09-15-16.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eeddefe3d5ca471f97fb6fd6ac9c71a6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

- 合并处理。根据一个通用 Vue 实例所包含的选项进行分类逐一判断合并，如 props、data、 methods、watch、computed、生命周期等，将合并结果存储在新定义的 options 对象里。
- 返回合并结果 options。

### extend 有什么作用

这个 API 很少用到，作用是扩展组件生成一个构造器，通常会与 `$mount` 一起使用。

### 什么是 mixin ？

Mixin 使我们能够为 Vue 组件编写可插拔和可重用的功能。

如果希望在多个组件之间重用一组组件选项，例如生命周期 hook、 方法等，则可以将其编写为 mixin，并在组件中简单的引用它。

然后将 mixin 的内容合并到组件中。如果你要在 mixin 中定义生命周期 hook，那么它在执行时将优化于组件自已的 hook。

### mixin 和 mixins 区别

`mixin` 用于全局混入，会影响到每个组件实例，通常插件都是这样做初始化的。

虽然文档不建议在应用中直接使用 `mixin`，但是如果不滥用的话也是很有帮助的，比如可以全局混入封装好的 `ajax` 或者一些工具函数等等。

`mixins` 应该是最常使用的扩展组件的方式了。如果多个组件中有相同的业务逻辑，就可以将这些逻辑剥离出来，通过 `mixins` 混入代码，比如上拉下拉加载数据这种逻辑等等。 另外需要注意的是 `mixins` 混入的钩子函数会先于组件内的钩子函数执行，并且在遇到同名选项的时候也会有选择性的进行合并。

## 28. data为什么是一个函数而不是对象

对象为引用类型，当复用组件时，由于数据对象都指向同一个data对象，当在一个组件中修改data时，其他重用的组件中的data会同时被修改；而使用返回对象的函数，由于每次返回的都是一个新对象（Object的实例），引用地址不同，则不会出现这个问题。

## 29. 动态给vue的data添加一个新的属性时会发生什么？怎样解决？

**问题:** 数据虽然更新了, 但是页面没有更新

**原因:**

1. `vue2`是用过`Object.defineProperty`实现数据响应式
2. 当我们访问定义的属性或者修改属性值的时候都能够触发`setter`与`getter`
3. 但是我们为`obj`添加新属性的时候，却无法触发事件属性的拦截
4. 原因是一开始`obj`的要定义的属性被设成了响应式数据，而`新增的属性`并没有通过`Object.defineProperty`设置成响应式数据

**解决方案:**

1. Vue.set()
   - 通过`Vue.set`向响应式对象中添加一个`property`，并确保这个新 `property`同样是响应式的，且触发视图更新
2. Object.assign()
   - 直接使用`Object.assign()`添加到对象的新属性不会触发更新
   - 应创建一个新的对象，合并原对象和混入对象的属性
3. $forceUpdate
   - 如果你发现你自己需要在 `Vue`中做一次强制更新，99.9% 的情况，是你在某个地方做错了事
   - `$forceUpdate`迫使`Vue` 实例重新渲染
   - PS：仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。

总结

- 如果为对象添加**少量**的新属性，可以直接采用`Vue.set()`
- 如果需要为新对象添加**大量**的新属性，则通过`Object.assign()`创建新对象
- 如果你实在不知道怎么操作时，可采取`$forceUpdate()`进行强制刷新 (不建议)

PS：`vue3`是用过`proxy`实现数据响应式的，直接动态添加新属性仍可以实现数据响应式

## 30. Vue data 中某一个属性的值发生改变后，视图会立即同步执行重新渲染吗

**不会立即同步执行重新渲染**。Vue 实现响应式并不是数据发生变化之后 DOM 立即变化，而是按一定的策略进行 DOM 的更新。Vue 在更新 DOM 时是`异步`执行的。只要侦听到数据变化， Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。

如果同一个watcher被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环tick中，Vue 刷新队列并执行实际（已去重的）工作。

## 31. vue如何监听(检测)对象或者数组某个属性的变化

当在项目中直接设置数组的某一项的值，或者直接设置对象的某个属性值，这个时候，你会发现页面并没有更新。这是因为Object.defineProperty()限制，监听不到变化。

解决方式：

- this.$set(你要改变的数组/对象，你要改变的位置/key，你要改成什么value)

```kotlin
this.$set(this.arr, 0, "OBKoro1"); // 改变数组
this.$set(this.obj, "c", "OBKoro1"); // 改变对象
复制代码
```

- 调用以下几个数组的方法

```scss
splice()、 push()、pop()、shift()、unshift()、sort()、reverse() 
复制代码
```

vue源码里缓存了array的原型链，然后重写了这几个方法，触发这几个方法的时候会observer数据，意思是使用这些方法不用我们再进行额外的操作，视图自动进行更新。 推荐使用splice方法会比较好自定义,因为splice可以在数组的任何位置进行删除/添加操作

**vm.`$set` 的实现原理是：**

- 如果目标是数组，直接使用数组的 splice 方法触发相应式；
- 如果目标是对象，会先判读属性是否存在、对象是否是响应式，最终如果要对属性进行响应式处理，则是通过调用 defineReactive 方法进行响应式处理（ defineReactive 方法就是 Vue 在初始化对象时，给对象属性采用 Object.defineProperty 动态添加 getter 和 setter 的功能所调用的方法）

## **32. assets和static的区别**

- **相同点：** `assets` 和 `static` 两个都是存放静态资源文件。项目中所需要的资源文件图片，字体图标，样式文件等都可以放在这两个文件下，这是相同点
- **不相同点：** `assets` 中存放的静态资源文件在项目打包时，也就是运行 `npm run build` 时会将 `assets` 中放置的静态资源文件进行打包上传，所谓打包简单点可以理解为压缩体积，代码格式化。而压缩后的静态资源文件最终也都会放置在 `static` 文件中跟着 `index.html` 一同上传至服务器。`static` 中放置的静态资源文件就不会要走打包压缩格式化等流程，而是直接进入打包好的目录，直接上传至服务器。因为避免了压缩直接进行上传，在打包时会提高一定的效率，但是 `static` 中的资源文件由于没有进行压缩等操作，所以文件的体积也就相对于 `assets` 中打包后的文件提交较大点。在服务器中就会占据更大的空间。
- **建议：** 将项目中 `template`需要的样式文件js文件等都可以放置在 `assets` 中，走打包这一流程。减少体积。而项目中引入的第三方的资源文件如`iconfoont.css` 等文件可以放置在 `static` 中，因为这些引入的第三方文件已经经过处理，不再需要处理，直接上传。

## 33. Vue的性能优化(项目优化)有哪些

### **（1）编码阶段**

- 尽量减少data中的数据，data中的数据都会增加getter和setter，会收集对应的watcher
- v-if和v-for不能连用
- 如果需要使用v-for给每项元素绑定事件时使用事件代理
- SPA 页面采用keep-alive缓存组件
- 在更多的情况下，使用v-if替代v-show
- key保证唯一
- 使用路由懒加载、异步组件
- 防抖、节流
- 第三方模块按需导入
- 长列表滚动到可视区域动态加载
- 图片懒加载

### **（2）SEO优化**

- 预渲染
- 服务端渲染SSR

### **（3）打包优化**

- 压缩代码
- Tree Shaking/Scope Hoisting
- 使用cdn加载第三方模块
- 多线程打包happypack
- splitChunks抽离公共文件
- sourceMap优化

### **（4）用户体验**

- 骨架屏
- PWA
- 还可以使用缓存(客户端缓存、服务端缓存)优化
- 服务端开启gzip压缩等。

## 34. Vue的template模版编译原理

vue中的模板template无法被浏览器解析并渲染，因为这不属于浏览器的标准，不是正确的HTML语法，所有需要将template转化成一个JavaScript函数，这样浏览器就可以执行这一个函数并渲染出对应的HTML元素，就可以让视图跑起来了，这一个转化的过程，就成为模板编译。模板编译又分三个阶段，解析parse，优化optimize，生成generate，最终生成可执行函数render。

- **解析阶段**：使用大量的正则表达式对template字符串进行解析，将标签、指令、属性等转化为抽象语法树AST。
- **优化阶段**：遍历AST，找到其中的一些静态节点并进行标记，方便在页面重渲染的时候进行diff比较时，直接跳过这一些静态节点，优化runtime的性能。
- **生成阶段**：将最终的AST转化为render函数字符串。

## 35. template和jsx的有什么分别？

对于 runtime 来说，只需要保证组件存在 render 函数即可，而有了预编译之后，只需要保证构建过程中生成 render 函数就可以。在 webpack 中，使用`vue-loader`编译.vue文件，内部依赖的`vue-template-compiler`模块，在 webpack 构建过程中，将template预编译成 render 函数。与 react 类似，在添加了jsx的语法糖解析器`babel-plugin-transform-vue-jsx`之后，就可以直接手写render函数。

所以，**template和jsx的都是render的一种表现形式**，不同的是：JSX相对于template而言，具有更高的灵活性，在复杂的组件中，更具有优势，而 template 虽然显得有些呆滞。但是 template 在代码结构上更符合视图与逻辑分离的习惯，更简单、更直观、更好维护。

## 35.2 讲讲什么是 JSX ？

jsx是JavaScript的一种语法扩展，它跟模板语言很接近，但是它充分具备JavaScript的能力 当 Facebook 第一次发布 React 时，他们还引入了一种新的 JS 方言 JSX，将原始 HTML 模板嵌入到 JS 代码中。JSX 代码本身不能被浏览器读取，必须使用Babel和webpack等工具将其转换为传统的JS。 JSX中的标签可以是单标签，也可以是双标签，但必须保证标签是闭合的。

## 36. 对SSR的理解

SSR也就是服务端渲染，也就是将Vue在客户端把标签渲染成HTML的工作放在服务端完成，然后再把html直接返回给客户端

**SSR的优势**：

- 更好的SEO
- 首屏加载速度更快

**SSR的缺点**：

- 开发条件会受到限制，服务器端渲染只支持beforeCreate和created两个钩子；
- 当需要一些外部扩展库时需要特殊处理，服务端渲染应用程序也需要处于Node.js的运行环境；
- 更多的服务端负载。

## 37. vue初始化页面闪动问题

使用vue开发时，在vue初始化之前，由于div是不归vue管的，所以我们写的代码在还没有解析的情况下会容易出现花屏现象，看到类似于{{message}}的字样，虽然一般情况下这个时间很短暂，但是还是有必要让解决这个问题的。

首先：在css里加上以下代码：

![17.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e0a4cd18fa8432aaaa7ec5951c58836~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

如果没有彻底解决问题，则在根元素加上`style="display: none;" :style="{display: 'block'}"`

## 38. 虚拟DOM

### (1). 什么是虚拟DOM

虚拟(Virtual) DOM 其实就是一棵以 JavaScript 对象（VNode 节点）作为基础的树，用对象属性来描述节点，**相当于在js和真实dom中间加来一个缓存**，利用dom **diff算法避免没有必要的dom操作**，从而提高性能。当然算法有时并不是最优解，因为它需要兼容很多实际中可能发生的情况，比如后续会讲到两个节点的dom树移动。

在vue中一般都是通过修改元素的state,订阅者根据state的变化进行编译渲染，底层的实现可以简单理解为三个步骤：

- 1、用JavaScript对象结构表述dom树的结构，然后用这个树构建一个真正的dom树，插到浏览器的页面中。
- 2、当状态改变了，也就是我们的state做出修改，vue便会重新构造一棵树的对象树，然后用这个新构建出来的树和旧树进行对比（只进行同层对比），记录两棵树之间的差异。
- 3、把记录的差异再重新应用到所构建的真正的dom树，视图就更新了。

它的表达方式就是把每一个标签都转为一个对象，这个对象可以有三个属性：`tag`、`props`、`children`

- **tag**：必选。就是标签。也可以是组件，或者函数
- **props**：非必选。就是这个标签上的属性和方法
- **children**：非必选。就是这个标签的内容或者子节点，如果是文本节点就是字符串，如果有子节点就是数组。换句话说 如果判断 children 是字符串的话，就表示一定是文本节点，这个节点肯定没有子元素

### (2). 虚拟DOM的解析过程

虚拟DOM的解析过程：

- 首先对将要插入到文档中的 DOM 树结构进行分析，使用 js 对象将其表示出来，比如一个元素对象，包含 TagName、props 和 Children 这些属性。然后将这个 js 对象树给保存下来，最后再将 DOM 片段插入到文档中。
- 当页面的状态发生改变，需要对页面的 DOM 的结构进行调整的时候，首先根据变更的状态，重新构建起一棵对象树，然后将这棵新的对象树和旧的对象树进行比较，记录下两棵树的的差异。
- 最后将记录的有差异的地方应用到真正的 DOM 树中去，这样视图就更新了。

### (3). 为什么要用虚拟DOM

**① 保证性能下限，在不进行手动优化的情况下，提供过得去的性能** 看一下页面渲染的流程：**解析HTML -> 生成DOM -> 生成 CSSOM -> Layout -> Paint -> Compiler** 下面对比一下修改DOM时真实DOM操作和Virtual DOM的过程，来看一下它们重排重绘的性能消耗∶

- 真实DOM∶ 生成HTML字符串＋重建所有的DOM元素
- 虚拟DOM∶ 生成vNode+ DOMDiff＋必要的dom更新

Virtual DOM的更新DOM的准备工作耗费更多的时间，也就是JS层面，相比于更多的DOM操作它的消费是极其便宜的。尤雨溪在社区论坛中说道∶ 框架给你的保证是，你不需要手动优化的情况下，依然可以给你提供过得去的性能。

**② 跨平台** Virtual DOM本质上是JavaScript的对象，它可以很方便的跨平台操作，比如服务端渲染、uniapp等。

### (4). 虚拟DOM真的比真实DOM性能好吗

- 首次渲染大量DOM时，由于多了一层虚拟DOM的计算，会比innerHTML插入慢。
- 正如它能保证性能下限，在真实DOM操作的时候进行针对性的优化时，还是更快的。

## 39. Diff算法

在新老虚拟DOM对比时：

- 首先，对比节点本身，**判断是否为同一节点**，如果不为相同节点，则删除该节点重新创建节点进行替换
- 如果为相同节点，进行**patchVnode**，判断如何对该节点的子节点进行处理，先判断一方有子节点一方没有子节点的情况(如果新的children没有子节点，将旧的子节点移除)
- 比较如果都有子节点，则进行**updateChildren**，判断如何对这些新老节点的子节点进行操作（diff核心）。
- 匹配时，找到相同的子节点，递归比较子节点

在diff中，只对同层的子节点进行比较，放弃跨级的节点比较，使得时间复杂从O(n3)降低值O(n)，也就是说，只有当新旧children都为多个子节点时才需要用核心的Diff算法进行同层级比较。

## 40. SPA单页面应用

**概念：**

- SPA单页面应用（SinglePage Web Application），指只有一个主页面的应用，一开始只需要加载一次js、css等相关资源。所有内容都包含在主页面，对每一个功能模块组件化。单页应用跳转，就是切换相关组件，仅仅刷新局部资源。
- MPA多页面应用 （MultiPage Application），指有多个独立页面的应用，每个页面必须重复加载js、css等相关资源。多页应用跳转，需要整页资源刷新。

**单页应用优缺点**

1. 优点：
   - 具有桌面应用的即时性、网站的可移植性和可访问性
   - 用户体验好、快，内容的改变不需要重新加载整个页面
2. 缺点：
   - 首次渲染速度相对较慢
   - 不利于搜索引擎的抓取

**解决首次加载慢的问题**

```markdown
1. 减少入口文件体积
2. 静态资源本地缓存
3. UI框架按需加载
4. 路由懒加载
复制代码
```

## 41. 使用异步组件有什么好处？

1. 节省打包出的结果，异步组件分开打包，采用jsonp的方式进行加载，有效解决文件过大的问题。
2. 核心就是包组件定义变成一个函数，依赖`import（）` 语法，可以实现文件的分割加载。

# Vue-Router 路由

## 1. 对前端路由的理解

在前端技术早期，一个 url 对应一个页面，如果要从 A 页面切换到 B 页面，那么必然伴随着页面的刷新。这个体验并不好，不过在最初也是无奈之举——用户只有在刷新页面的情况下，才可以重新去请求数据。

后来，改变发生了——Ajax 出现了，它允许人们在不刷新页面的情况下发起请求；与之共生的，还有“不刷新页面即可更新页面内容”这种需求。在这样的背景下，出现了 **SPA（单页面应用**）。

SPA极大地提升了用户体验，它允许页面在不刷新的情况下更新页面内容，使内容的切换更加流畅。但是在 SPA 诞生之初，人们并没有考虑到“定位”这个问题——在内容切换前后，页面的 URL 都是一样的，这就带来了两个问题：

- SPA 其实并不知道当前的页面“进展到了哪一步”。可能在一个站点下经过了反复的“前进”才终于唤出了某一块内容，但是此时只要刷新一下页面，一切就会被清零，必须重复之前的操作、才可以重新对内容进行定位——SPA 并不会“记住”你的操作。
- 由于有且仅有一个 URL 给页面做映射，这对 SEO 也不够友好，搜索引擎无法收集全面的信息

**为了解决这个问题，前端路由出现了**。

前端路由可以帮助我们在仅有一个页面的情况下，“记住”用户当前走到了哪一步——为 SPA 中的各个视图匹配一个唯一标识。这意味着用户前进、后退触发的新内容，都会映射到不同的 URL 上去。此时即便他刷新页面，因为当前的 URL 可以标识出他所处的位置，因此内容也不会丢失。

那么如何实现这个目的呢？首先要解决两个问题：

- 当用户刷新页面时，浏览器会默认根据当前 URL 对资源进行重新定位（发送请求）。这个动作对 SPA 是不必要的，因为我们的 SPA 作为单页面，无论如何也只会有一个资源与之对应。此时若走正常的请求-刷新流程，反而会使用户的前进后退操作无法被记录。
- 单页面应用对服务端来说，就是一个URL、一套资源，那么如何做到用“不同的URL”来映射不同的视图内容呢？

从这两个问题来看，服务端已经完全救不了这个场景了。所以要靠咱们前端自力更生，不然怎么叫“前端路由”呢？作为前端，可以提供这样的解决思路：

- 拦截用户的刷新操作，避免服务端盲目响应、返回不符合预期的资源内容。把刷新这个动作完全放到前端逻辑里消化掉。
- 感知 URL 的变化。这里不是说要改造 URL、凭空制造出 N 个 URL 来。而是说 URL 还是那个 URL，只不过我们可以给它做一些微小的处理——这些处理并不会影响 URL 本身的性质，不会影响服务器对它的识别，只有我们前端感知的到。一旦我们感知到了，我们就根据这些变化、用 JS 去给它生成不同的内容。

## 2. VueRouter是什么, 有那些组件

- Vue Router 是官方的路由管理器。它和 Vue.js 的核心深度集成，路径和组件的映射关系, 让构建单页面应用变得易如反掌。
- router-link 实质上最终会渲染成a链接
- router-view 子级路由显示
- keep-alive 包裹组件缓存

## 3. `$route` 和`$router` 的区别

- $route 是“路由信息对象”，包括 path，params，hash，query，fullPath，matched，name 等路由信息参数
- $router 是“路由实例”对象包括了路由的跳转方法，钩子函数等。

## 4. 路由开发的优缺点

优点：

- 整体不刷新页面，用户体验更好
- 数据传递容易, 开发效率高

缺点：

- 开发成本高(需要学习专门知识)
- 首次加载会比较慢一点。不利于seo

## 5. VueRouter的使用方式

1. 使用Vue.use( )将VueRouter插入
2. 创建路由规则
3. 创建路由对象
4. 将路由对象挂到 Vue 实例上
5. 设置路由挂载点

## 6. vue-router 路由模式有几种？

**hash模式、history模式、Abstract模式**

### 前端路由原理

前端路由的核心，就在于改变视图的同时不会向后端发出请求；而是加载路由对应的组件

vue-router就是将组件映射到路由, 然后渲染出来的。并实现了三种模式

Hash模式、History模式以及Abstract模式。默认Hash模式

**hash模式**

是指 url 尾巴后的 # 号以及后面的字符。hash 虽然出现在url中，但不会被包括在http请求中，对后端完全没有影响，因此改变hash不会被重新加载页面。

**history 模式**

URL 就像正常的 url, 不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/user/id` 就会返回 404，这就不好看了

**Abstract模式**

支持所有javascript运行模式。vue-router 自身会对环境做校验，如果发现没有浏览器的 API，路由会自动强制进入 abstract 模式。在移动端原生环境中也是使用 abstract 模式。

**修改路由模式:** 在实例化路由对象时, 传入mode选项和值修改

### Hash模式

**原理** **基于浏览器的hashchange事件**，地址变化时，通过window.location.hash 获取地址上的hash值；并通过构造Router类，配置routes对象设置hash值与对应的组件内容。

**优点**

1. **hash值会出现在URL中, 但是不会被包含在Http请求中, 因此hash值改变不会重新加载页面**
2. **hash改变会触发hashchange事件, 能控制浏览器的前进后退**
3. **兼容性好**

**缺点**

1. 地址栏中携带#，不美观
2. 只可修改 # 后面的部分，因此只能设置与当前 URL 同文档的 URL
3. hash有体积限制，故只可添加短字符串
4. **设置的新值必须与原来不一样才会触发hashchange事件**，并将记录添加到栈中
5. **每次URL的改变不属于一次http请求，所以不利于SEO优化**

### History模式

**原理**

**基于HTML5新增的pushState()和replaceState()两个api，以及浏览器的popstate事件**，地址变化时，通过window.location.pathname找到对应的组件。并通过构造Router类，配置routes对象设置pathname值与对应的组件内容。

**优点**

1. 没有#，更加美观
2. pushState() 设置的新 URL 可以是与当前 URL 同源的任意 URL
3. pushState() 设置的新 URL 可以与当前 URL 一模一样，这样也会把记录添加到栈中
4. pushState() 通过 stateObject 参数可以添加任意类型的数据到记录中
5. pushState() 可额外设置 title 属性供后续使用
6. 浏览器的进后退能触发浏览器的popstate事件，获取window.location.pathname来控制页面的变化

**缺点**

1. URL的改变属于http请求，借助history.pushState实现页面的无刷新跳转，因此会重新请求服务器。所以**前端的 URL 必须和实际向后端发起请求的 URL 一致**。如果用户输入的URL回车或者浏览器刷新或者分享出去某个页面路径，用户点击后，URL与后端配置的页面请求URL不一致，则匹配不到任何静态资源，就会返回404页面。所以需要后台配置支持，覆盖所有情况的候选资源，如果 URL 匹配不到任何静态资源，则应该返回app 依赖的页面或者应用首页。
2. **兼容性差**，特定浏览器支持

### 为什么history模式下路由跳转会报404

1. URL的改变属于http请求，借助history.pushState实现页面的无刷新跳转，因此会重新请求服务器
2. 所以前端的 URL 必须和实际向后端发起请求的 URL 一致

## 7. 路由跳转有那些方式

1、this.$router.push()跳转到指定的url，并在history中添加记录，点击回退返回到上一个页面

2、this.$router.replace()跳转到指定的url，但是history中不会添加记录，点击回退到上上个页面

3、this.$router.go(n)向前或者后跳转n个页面，n可以是正数也可以是负数

### 编程式导航使用的方法以及常用的方法

- 路由跳转 ： this.$router.push()
- 路由替换 : this.$router.replace()
- 后退： this.$router.back()
- 前进 ：this.$router.forward()

## 8. Vue-router跳转和location.href有什么区别

- 使用 `location.href= /url`来跳转，简单方便，但是刷新了页面；
- 使用 `history.pushState( /url )` ，无刷新页面，静态跳转；
- 引进 router ，然后使用 `router.push( /url )` 来跳转，使用了 `diff` 算法，实现了按需加载，减少了 dom 的消耗。其实使用 router 跳转和使用 `history.pushState()` 没什么差别的，因为vue-router就是用了 `history.pushState()` ，尤其是在history模式下。

## 9. 如何获取页面的hash变化

**（1）监听$route的变化**

![111.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d5a3474579e498f81c52466ad40ee60~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**（2）window.location.hash读取#值**

window.location.hash 的值可读可写，读取来判断状态是否改变，写入时可以在不重载网页的前提下，添加一条历史访问记录。

## 10. 路由的传参方式

### 声明式导航传参

在 router-link 上的 to 属性传值,

1. /path?参数名=值
   - 接收传递过来的值: $route.query.参数名
2. /path/值/值 –> 需要路由对象提前配置 path: “/path/参数名”
   - 接收传递过来的值: $route.params.参数名

### 编程式导航传参

this.$router.push( ) 可以不参数,根据传的值自动匹配是path还是name

因为使用path会自动忽略params ,所以会出现两种组合

**(1) name+params 方式传参**

A页面传参

```csharp
this.$router.push({
    name: 'xxx', // 跳转的路由
    params: {
      id: id   // 发送的参数
    }
})
复制代码
```

B页面接收传参：

this.$route.params.id

**(2) path+query 方式传参**

A页面传参

```kotlin
this.$router.push({
    path: '/xxx', // 跳转的路由
    query: {
      id: id    // 发送的参数
    }
})
复制代码
```

B页面接参：

this.$route.query.id

**params 和query 方式传参的区别**

1. 写法上不同
2. 地址栏不同
3. 刷新方式不同

## 11. params和query的区别

**用法**：query要用path来引入，params要用name来引入，接收参数都是类似的，分别是 `this.$route.query.name` 和 `this.$route.params.name` 。

**url地址显示**：query更加类似于ajax中get传参，params则类似于post，说的再简单一点，前者在浏览器地址栏中显示参数，后者则不显示

**注意**：query刷新不会丢失query里面的数据 params刷新会丢失 params里面的数据。

## 12. 路由配置项常用的属性及作用

路由配置参数：

path : 跳转路径  component : 路径相对于的组件  name:命名路由  children:子路由的配置参数(路由嵌套)  props:路由解耦  redirect : 重定向路由

## 13. 路由重定向和404

**路由重定向**

1. 匹配path后, 强制切换到另一个目标path上
2. redirect 是设置要重定向到哪个路由路径
3. 网页默认打开, 匹配路由"/", 强制切换到"/find"上
4. redirect配置项, 值为要强制切换的路由路径
5. 强制重定向后, 还会重新来数组里匹配一次规则

**404页面**

1. 如果路由hash值, 没有和数组里规则匹配
2. path: ' * ' (任意路径)
3. 默认给一个404页面
4. 如果路由未命中任何规则, 给出一个兜底的404页面

## 14. Vue-router 导航守卫有哪些

- 全局守卫：**beforeEach**、beforeResolve、**afterEach**
- 路由独享的守卫：beforeEnter
- 组件内的守卫：beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave

## 15. Vue-router 路由钩子在生命周期的体现

- ### Vue-Router导航守卫

有的时候，需要通过路由来进行一些操作，比如最常见的登录权限验证，当用户满足条件时，才让其进入导航，否则就取消跳转，并跳到登录页面让其登录。 为此有很多种方法可以植入路由的导航过程：全局的，单个路由独享的，或者组件级的

**1. 全局路由钩子**

vue-router全局有三个路由钩子;

- router.beforeEach 全局前置守卫 进入路由之前
- router.beforeResolve 全局解析守卫（2.5.0+）在 beforeRouteEnter 调用之后调用
- router.afterEach 全局后置钩子 进入路由之后

具体使用∶

- beforeEach（判断是否登录了，没登录就跳转到登录页）

![112.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71c9b894026e42ef806ba50492c7eb49~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

- afterEach （跳转之后滚动条回到顶部）

![113.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e923f83b50294e8782fd7c2513fbc27d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**2. 单个路由独享钩子**

**beforeEnter** 如果不想全局配置守卫的话，可以为某些路由单独配置守卫，有三个参数∶ to、from、next

![114.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bc782aef20148688e75213be98b95e7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

**3. 组件内钩子**

beforeRouteUpdate、beforeRouteEnter、beforeRouteLeave

这三个钩子都有三个参数∶to、from、next

- beforeRouteEnter∶ 进入组件前触发
- beforeRouteUpdate∶ 当前地址改变并且改组件被复用时触发，举例来说，带有动态参数的路径foo/∶id，在 /foo/1 和 /foo/2 之间跳转的时候，由于会渲染同样的foa组件，这个钩子在这种情况下就会被调用
- beforeRouteLeave∶ 离开组件被调用

注意点，beforeRouteEnter组件内还访问不到this，因为该守卫执行前组件实例还没有被创建，需要传一个回调给 next来访问，例如：

![115.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b6b67980c2d483691547cb83f90f6f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

- ### Vue路由钩子在生命周期函数的体现

**1. 完整的路由导航解析流程（不包括其他生命周期）**

- 触发进入其他路由。
- 调用要离开路由的组件守卫beforeRouteLeave
- 调用局前置守卫∶ beforeEach
- 在重用的组件里调用 beforeRouteUpdate
- 调用路由独享守卫 beforeEnter。
- 解析异步路由组件。
- 在将要进入的路由组件中调用 beforeRouteEnter
- 调用全局解析守卫 beforeResolve
- 导航被确认。
- 调用全局后置钩子的 afterEach 钩子。
- 触发DOM更新（mounted）。
- 执行beforeRouteEnter 守卫中传给 next 的回调函数

**2. 触发钩子的完整顺序**

路由导航、keep-alive、和组件生命周期钩子结合起来的，触发顺序，假设是从a组件离开，第一次进入b组件∶

- beforeRouteLeave：路由组件的组件离开路由前钩子，可取消路由离开。
- beforeEach：路由全局前置守卫，可用于登录验证、全局路由loading等。
- beforeEnter：路由独享守卫
- beforeRouteEnter：路由组件的组件进入路由前钩子。
- beforeResolve：路由全局解析守卫
- afterEach：路由全局后置钩子
- beforeCreate：组件生命周期，不能访问this。
- created;组件生命周期，可以访问this，不能访问dom。
- beforeMount：组件生命周期
- deactivated：离开缓存组件a，或者触发a的beforeDestroy和destroyed组件销毁钩子。
- mounted：访问/操作dom。
- activated：进入缓存组件，进入a的嵌套子组件（如果有的话）。
- 执行beforeRouteEnter回调函数next。

**3. 导航行为被触发到导航完成的整个过程**

- 导航行为被触发，此时导航未被确认。
- 在失活的组件里调用离开守卫 beforeRouteLeave。
- 调用全局的 beforeEach守卫。
- 在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
- 在路由配置里调用 beforeEnteY。
- 解析异步路由组件（如果有）。
- 在被激活的组件里调用 beforeRouteEnter。
- 调用全局的 beforeResolve 守卫（2.5+），标示解析阶段完成。
- 导航被确认。
- 调用全局的 afterEach 钩子。
- 非重用组件，开始组件实例的生命周期：beforeCreate&created、beforeMount&mounted
- 触发 DOM 更新。
- 用创建好的实例调用 beforeRouteEnter守卫中传给 next 的回调函数。
- 导航完成

# Vuex

## 1. 什么Vuex ,谈谈你对它的理解？

1. 首先vuex的出现是为了解决web组件化开发的过程中，各组件之间传值的复杂和混乱的问题
2. 将我们在多个组件中需要共享的数据放到state中，
3. 要获取或格式化数据需要使用getters，
4. 改变state中的数据，可以使用mutation，但是只能包含同步的操作，在具体组件里面调用的方式`this.$store.commit('xxxx')`
5. Action也是改变state中的数据，不过是提交的mutation，并且可以包含异步操作，在组件中的调用方式`this.$store.dispatch('xxx')`； 在actions里面使用的commit('调用mutation')

**Vuex** 是一个专为 Vue.js应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 devtools extension，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

## 2. Vuex**各模块在核心流程中的主要功能：**

`Vue Components`∶ Vue组件。HTML页面上，负责接收用户操作等交互行为，执行dispatch方法触发对应action进行回应。

`dispatch`∶操作行为触发方法，是唯一能执行action的方法。

`actions`∶ 操作行为处理模块。负责处理Vue Components接收到的所有交互行为。包含同步/异步操作，支持多个同名方法，按照注册的顺序依次触发。向后台API请求的操作就在这个模块中进行，包括触发其他action以及提交mutation的操作。该模块提供了Promise的封装，以支持action的链式触发。

`commit`∶状态改变提交操作方法。对mutation进行提交，是唯一能执行mutation的方法。

`mutations`∶状态改变操作方法。是Vuex修改state的唯一推荐方法，其他修改方式在严格模式下将会报错。该方法只能进行同步操作，且方法名只能全局唯一。操作之中会有一些hook暴露出来，以进行state的监控等。

`state`∶ 页面状态管理容器对象。集中存储Vuecomponents中data对象的零散数据，全局唯一，以进行统一的状态管理。页面显示所需的数据从该对象中进行读取，利用Vue的细粒度数据响应机制来进行高效的状态更新。

`getters`∶ state对象读取方法。图中没有单独列出该模块，应该被包含在了render中，Vue Components通过该方法读取全局state对象。

## 2.1 简述Vuex的数据传输流程

当组件进行数据修改的时候我们需要调**用dispatch来触发actions里面的方法**。actions里面的每个方法中都会有一个commit方法，当方法执行的时候会通过**commit来触发mutations里面的方法进行数据的修改**。mutations里面的每个函数都会有一个state参数，这样就可以在**mutations里面进行state的数据修改**，当数据修改完毕后，会传导给页面。页面的数据也会发生改变

## 3. vuex中有几个核心属性，分别是什么？

一共有5个核心属性，分别是:

- `state`唯一数据源,Vue 实例中的 data 遵循相同的规则
- `mutation`更改 Vuex 的 store 中的状态的唯一方法是提交 mutation,非常类似于事件,通过store.commit 方法触发
- `action`action 类似于 mutation，不同在于action 提交的是 mutation，而不是直接变更状态，action 可以包含任意异步操作
- `module` 由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成模块（module）。

```css
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
复制代码
```

- `getters` 可以认为是 store 的计算属性,就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。Getter 会暴露为 store.getters 对象，你可以以属性的形式访问这些值.

```php
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})

store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
复制代码
```

## 4. Vuex中action和mutation的区别

`mutation`中的操作是一系列的同步函数，用于修改state中的变量的的状态。当使用vuex时需要通过commit来提交需要操作的内容。mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

![221.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9dd15a982994d2592ddd126acccebff~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

当触发一个类型为 increment 的 mutation 时，需要调用此函数：

![222.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03c650fd2cf646a2b38035694aed0cd6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

而`action`类似于mutation，不同点在于：

- action 可以包含任意异步操作。
- action 提交的是 mutation，而不是直接变更状态。

![223.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54727d1d6a70479abec170367d52e8bc~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

`action`函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。 所以，两者的不同点如下：

- mutation专注于修改State，理论上是修改State的唯一途径；action 用来处理业务代码、异步请求。
- mutation：必须同步执行；action ：可以异步，但不能直接操作State。
- 在视图更新时，先触发actions，actions再触发mutation
- mutation的参数是state，它包含store中的数据；store的参数是context，它是 state 的父级，包含 state、getters

## 5. vuex的getter的作用

`getter`有点类似 Vue.js 的**计算属性**，当我们需要从 store 的 state 中派生出一些状态，那么我们就需要使用 getter，getter 会接收 state 作为第 一个参数，而且 getter 的返回值会根据它的依赖被缓存起来，只有 getter 中的依赖值（state 中的某个需要派生状态的值）发生改变的时候才会被重新计算。

## 6. Vuex 和 localStorage 的区别

**（1）** **最重要的区别**

- vuex存储在**内存**中
- localstorage 则以**文件**的方式存储在**本地**，只能存储字符串类型的数据，存储对象需要 JSON的stringify和parse方法进行处理。 读取内存比读取硬盘速度要快

**（2）应用场景**

- Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。vuex用于组件之间的传值。
- localstorage是本地存储，是将数据存储到浏览器的方法，一般是在跨页面传递数据时使用 。
- Vuex能做到数据的响应式，localstorage不能

**（3）永久性**

**刷新页面时vuex存储的值会丢失，localstorage不会。**

**注意：** 对于不变的数据确实可以用localstorage可以代替vuex，但是当两个组件共用一个数据源（对象或数组）时，如果其中一个组件改变了该数据源，希望另一个组件响应该变化时，localstorage无法做到，原因就是上面的最重要的区别。

## 7. Vuex页面刷新时丢失怎么处理

用sessionstorage 或者 localstorage 存储数据

存储： sessionStorage.setItem( '名', JSON.stringify(值) ) 使用： sessionStorage.getItem('名') ---得到的值为字符串类型，用JSON.parse()去引号；

## 8. Vuex和单纯的全局对象有什么区别？

- Vuex 的状态存储是**响应式**的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
- 不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样可以方便地跟踪每一个状态的变化，从而能够实现一些工具帮助更好地了解我们的应用。

## 9. Redux(react的) 和 Vuex 有什么区别，它们的共同思想

**（1）Redux 和 Vuex区别**

- Vuex改进了Redux中的Action和Reducer函数，以mutations变化函数取代Reducer，无需switch，只需在对应的mutation函数里改变state值即可
- Vuex由于Vue自动重新渲染的特性，无需订阅重新渲染函数，只要生成新的State即可
- Vuex数据流的顺序是∶View调用store.commit提交对应的请求到Store中对应的mutation函数->store改变（vue检测到数据变化自动渲染）

通俗点理解就是，vuex 弱化 dispatch，通过commit进行 store状态的一次更变;取消了action概念，不必传入特定的 action形式进行指定变更;弱化reducer，基于commit参数直接对数据进行转变，使得框架更加简易;

**（2）共同思想**

- 单—的数据源
- 变化可以预测

本质上：redux与vuex都是对mvvm思想的服务，将数据从视图中抽离的一种方案; 形式上：vuex借鉴了redux，将store作为全局的数据中心，进行mode管理;

## 10. 为什么要用 Vuex 或者 Redux

由于传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致代码无法维护。

所以需要把组件的**共享**状态抽取出来，以一个全局单例模式管理。在这种模式下，组件树构成了一个巨大的"视图"，不管在树的哪个位置，任何组件都能获取状态或者触发行为。

另外，通过定义和隔离状态管理中的各种概念并强制遵守一定的规则，代码将会变得更结构化且易维护。

## 11. 为什么 Vuex 的 mutation 中不能做异步操作？

- Vuex中所有的状态更新的唯一途径都是mutation，异步操作通过 Action 来提交 mutation实现，这样可以方便地跟踪每一个状态的变化，从而能够实现一些工具帮助更好地了解我们的应用。
- 每个mutation执行完成后都会对应到一个新的状态变更，这样devtools就可以打个快照存下来，然后就可以实现 time-travel 了。如果mutation支持异步操作，就没有办法知道状态是何时更新的，无法很好的进行状态的追踪，给调试带来困难。

## 12. Vuex的严格模式是什么,有什么作用，如何开启？

在严格模式下，无论何时发生了状态变更且不是由mutation函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。

在Vuex.Store 构造器选项中开启,如下

![224.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49de6f4128c6475aa2d9d596b5378c08~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 13. 如何在组件中批量使用Vuex的getter属性

使用`mapGetters`辅助函数, 利用对象展开运算符将getter混入computed 对象中

![225.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ab1d1eb07b74f2ab5ce4eba357bfc4d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 14. 如何在组件中重复使用Vuex的mutation

使用`mapMutations`辅助函数,在组件中这么使用

![226.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96b565782e5648f58c65a7c97e678438~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

然后调用`this.setNumber(10)`相当调用`this.$store.commit('SET_NUMBER',10)`

## 15. Vuex的辅助函数怎么用

比如当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState`辅助函数帮助我们生成计算属性，让你少按几次键。

### mapState

```javascript
import { mapState } from 'vuex'
 
export default {
  // ...
  computed:{
     ...mapState({
         // 箭头函数可使代码更简练
         count: state => state.count,
         // 传字符串参数 'count' 等同于 `state => state.count`
         countAlias: 'count',
 
         // 为了能够使用 `this` 获取局部状态，必须使用常规函数
         countPlusLocalState (state) {
             return state.count + this.localCount
         }
  	})
  }
}
复制代码
```

定义的属性名与state中的名称相同时，可以传入一个数组

```arduino
//定义state
const state={
    count:1,
}
 
//在组件中使用辅助函数
computed:{
    ...mapState(['count'])
}
复制代码
```

### mapGetters

```perl
computed:{
    ...mapGetters({
      // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
      doneCount: 'doneTodosCount'
    })
}
复制代码
```

当属性名与getters中定义的相同时，可以传入一个数组

总结：

- mapState与mapGetters都用computed来进行映射
- 在组件中映射完成后，通过this.映射属性名进行使用

### mapMutations

```css
methods:{
    ...mapMutations({
        add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
}
复制代码
```

当属性名与mapMutatios中定义的相同时，可以传入一个数组

```perl
methods:{
    ...mapMutations([
        'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
 
        // `mapMutations` 也支持载荷：
        'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
}
复制代码
```

### mapActions

```css
mathods:{
    ...mapActions({
        add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
}
复制代码
```

当属性名与mapActios中定义的相同时，可以传入一个数组

```perl
methods:{
    ...mapActions([
        'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`	
        // `mapActions` 也支持载荷：
        'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
}
复制代码
```

总结

- mapMutations与mapActios都在methods中进行映射
- 映射之后变成一个方法

# Vue面试题补充

## 1.怎样理解 Vue 的单向数据流？

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。

额外的，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。子组件想修改时，只能通过 $emit 派发一个自定义事件，父组件接收到后，由父组件修改。

有两种常见的试图改变一个 prop 的情形 :

- **这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用。** 在这种情况下，最好定义一个本地的 data 属性并将这个 prop 用作其初始值：

```kotlin
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
复制代码
```

- **这个 prop 以一种原始的值传入且需要进行转换。** 在这种情况下，最好使用这个 prop 的值来定义一个计算属性

```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
复制代码
```

## 2.在什么阶段才能访问操作DOM？

在钩子函数 mounted 被调用前，Vue 已经将编译好的模板挂载到页面上，所以在 mounted 中可以访问操作 DOM。

## 3. 父组件可以监听到子组件的生命周期吗？

比如有父组件 Parent 和子组件 Child，如果父组件监听到子组件挂载 mounted 就做一些逻辑处理，可以通过以下写法实现：

```typescript
// Parent.vue
<Child @mounted="doSomething"/>
    
// Child.vue
mounted() {
  this.$emit("mounted");
}
复制代码
```

以上需要手动通过 $emit 触发父组件的事件，更简单的方式可以在父组件引用子组件时通过 @hook 来监听即可，如下所示：

```typescript
//  Parent.vue
<Child @hook:mounted="doSomething" ></Child>

doSomething() {
   console.log('父组件监听到 mounted 钩子函数 ...');
},
    
//  Child.vue
mounted(){
   console.log('子组件触发 mounted 钩子函数 ...');
},    
    
// 以上输出顺序为：
// 子组件触发 mounted 钩子函数 ...
// 父组件监听到 mounted 钩子函数 ...     
复制代码
```

当然 @hook 方法不仅仅是可以监听 mounted，其它的生命周期事件，例如：created，updated 等都可以监听。

## 4.Vue 怎么用 vm.$set() 解决对象新增属性不能响应的问题 ？

```kotlin
export function set (target: Array<any> | Object, key: any, val: any): any {
  // target 为数组  
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // 修改数组的长度, 避免索引>数组长度导致splcie()执行有误
    target.length = Math.max(target.length, key)
    // 利用数组的splice变异方法触发响应式  
    target.splice(key, 1, val)
    return val
  }
  // key 已经存在，直接修改属性值  
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = (target: any).__ob__
  // target 本身就不是响应式数据, 直接赋值
  if (!ob) {
    target[key] = val
    return val
  }
  // 对属性进行响应式处理
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}
复制代码
```

我们阅读以上源码可知，vm.$set 的实现原理是：

- 如果目标是数组，直接使用数组的 splice 方法触发相应式；
- 如果目标是对象，会先判读属性是否存在、对象是否是响应式，最终如果要对属性进行响应式处理，则是通过调用 defineReactive 方法进行响应式处理（ defineReactive 方法就是 Vue 在初始化对象时，给对象属性采用 Object.defineProperty 动态添加 getter 和 setter 的功能所调用的方法）

## 5. 请说下封装 vue 组件的过程

> 有复用的地方就有封装
>
> （js如此，vue也是如此）

- 1.先分析需求：确定业务需求，把页面中可以复用的结构，样式以及功能
  - 找出业务需求中存在复用的地方
- 2.具体步骤：Vue.component 或者在new Vue配置项components中, 定义组件名, 可以在props中接受给组件传的参数和值，子组件修改好数据后，想把数据传递给父组件。可以采用$emit方法

## 6. 讲一下组件的命名规范

- 给组件命名有两种方式(在Vue.Component/components时)，一种是使用链式命名"my-component"，一种是使用大驼峰命名"MyComponent"，
- 因为要遵循W3C规范中的自定义组件名 (字母全小写且必须包含一个连字符)，避免和当前以及未来的 HTML 元素相冲突

## 7.scoped作用与原理

- 作用：组件css作用域，避免`子组件`内部的css样式被`父组件`覆盖
  - 默认情况下，如果子组件和父组件css选择器权重相同，优先加载父组件css样式
- 原理：给元素添加一个自定义属性 v-data-xxxxx
  - `一针见血答案`： 通过属性选择题来提高css权重值

## 8. 第一次加载页面会触发哪几个钩子函数？

四个钩子

- beforeCreate,
- created,
- beforeMount,
- mounted 这几个钩子函数

## 9. Vue中如何扩展一个组件

1. 常见的组件扩展方法有：mixins，slots，extends等
2. 混入mixins是分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。

```javascript
// 复用代码：它是一个配置对象，选项和组件里面一样
const mymixin = {
   methods: {
      dosomething(){}
   }
}
// 全局混入：将混入对象传入
Vue.mixin(mymixin)

// 局部混入：做数组项设置到mixins选项，仅作用于当前组件
const Comp = {
   mixins: [mymixin]
}

复制代码
```

1. 插槽主要用于vue组件中的内容分发，也可以用于组件扩展。如果要精确分发到不同位置可以使用具名插槽，如果要使用子组件中的数据可以使用作用域插槽。
2. 组件选项中还有一个不太常用的选项extends，也可以起到扩展组件的目的
3. 混入的数据和方法**不能明确判断来源**且可能和当前组件内变量**产生命名冲突**，vue3中引入的composition api，可以很好解决这些问题，利用独立出来的响应式模块可以很方便的编写独立逻辑并提供响应式的数据，然后在setup选项中组合使用，增强代码的可读性和维护性

## 10. 如果让你从零开始写一个vue路由，说说你的思路

一个SPA应用的路由需要解决的问题是**页面跳转内容改变同时不刷新**，同时路由还需要以插件形式存在，所以：

1. 首先我会定义一个`createRouter`函数，返回路由器实例，实例内部做几件事：
   - 保存用户传入的配置项
   - 监听hash或者popstate事件
   - 回调里根据path匹配对应路由
2. 将router定义成一个Vue插件，即实现install方法，内部做两件事：
   - 实现两个全局组件：router-link和router-view，分别实现页面跳转和内容显示
   - 定义两个全局变量：�����和*ro**u**t**e*和router，组件内可以访问当前路由和路由器实例

## 11. 从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

1. 从0创建一个项目我大致会做以下事情：项目构建、引入必要插件、代码规范、提交规范、常用库和组件
2. 目前vue3项目我会用vite或者create-vue创建项目
3. 接下来引入必要插件：路由插件vue-router、状态管理vuex/pinia、ui库我比较喜欢element-plus和antd-vue、http工具我会选axios
4. 其他比较常用的库有vueuse，nprogress，图标可以使用vite-svg-loader
5. 下面是代码规范：结合prettier和eslint即可
6. 最后是提交规范，可以使用husky，lint-staged，commitlint

------

- 目录结构我有如下习惯： `.vscode`：用来放项目中的 vscode 配置

  `plugins`：用来放 vite 插件的 plugin 配置

  `public`：用来放一些诸如 页头icon 之类的公共文件，会被打包到dist根目录下

  `src`：用来放项目代码文件

  `api`：用来放http的一些接口配置

  `assets`：用来放一些 CSS 之类的静态资源

  `components`：用来放项目通用组件

  `layout`：用来放项目的布局

  `router`：用来放项目的路由配置

  `store`：用来放状态管理Pinia的配置

  `utils`：用来放项目中的工具方法类

  `views`：用来放项目的页面文件

## 12. 实际工作中，你总结的vue最佳实践有哪些？

### 编码风格方面：

- 命名组件时使用“多词”风格避免和HTML元素冲突
- 使用“细节化”方式定义属性而不是只有一个属性名
- 属性名声明时使用“驼峰命名”，模板或jsx中使用“肉串命名”
- 使用v-for时务必加上key，且不要跟v-if写在一起

### 性能方面：

- 路由懒加载减少应用尺寸
- 利用SSR减少首屏加载时间
- 利用v-once渲染那些不需要更新的内容
- 一些长列表可以利用虚拟滚动技术避免内存过度占用
- 对于深层嵌套对象的大数组可以使用shallowRef或shallowReactive降低开销
- 避免不必要的组件抽象

### 安全：

- 不使用不可信模板，例如使用用户输入拼接模板：`template: <div> + userProvidedString + </div>`
- 小心使用v-html，:url，:style等，避免html、url、样式等注入

## 13. 说说从 template 到 render 处理过程(compiler的工作原理)

1. Vue中有个独特的编译器模块，称为“compiler”，它的主要作用是将用户编写的template编译为js中可执行的render函数。
2. 之所以需要这个编译过程是为了便于前端程序员能高效的编写视图模板。相比而言，我们还是更愿意用HTML来编写视图，直观且高效。手写render函数不仅效率底下，而且失去了编译期的优化能力。
3. 在Vue中编译器会先对template进行解析，这一步称为parse，结束之后会得到一个JS对象，我们成为抽象语法树AST，然后是对AST进行深加工的转换过程，这一步成为transform，最后将前面得到的AST生成为JS代码，也就是render函数。

## 14. Vue实例挂载的过程中发生了什么?

1. 挂载过程指的是app.mount()过程，这个过程中整体上做了两件事：**初始化**和**建立更新机制**
2. 初始化会创建组件实例、初始化组件状态，创建各种响应式数据
3. 建立更新机制这一步会立即执行一次组件更新函数，这会首次执行组件渲染函数并执行patch将前面获得vnode转换为dom；同时首次执行渲染函数会创建它内部响应式数据之间和组件更新函数之间的依赖关系，这使得以后数据变化时会执行对应的更新函数。

## 15. 组件中的name属性有什么用？

1. 项目使用keep-alive时，可搭配组件name进行缓存过滤
2. DOM做递归组件时需要调用自身name
3. Vue-devtools调试工具里显示的组见名称是由Vue中组件name决定的
4. 动态切换组件

## 16 . 怎么在组件中监听路由参数的变化？

有两种方法可以监听路由参数的变化，但是只能用在包含的组件内。

**1. 侦听器** watch:{'this.$route'，(to,from){ //在此处监听 }, },

**2. 前置路由守卫** beforeRouteUpdate(to,from,next){ //这里监听 },

## 17. beforeDestroy钩子的作用

如果页面上有很多定时器，可以在data选项中创建一个对象timer，给每个定时器取个名字一一映射在对象timer中，在beforeDestroy构造函数中循环遍历所有定时器 ，一次性取消

```scss
for(let k in this.timer){
    clearInterval(k)
    }；
复制代码
```

如果页面只有单个定时器，可以这么做。

```javascript
const timer= setInterval(()=>{},500);
this.$once('hook:beforeDestroy',()=>{
clearInterval(timer);
})
复制代码
```

## 18. 说说在vue中踩过的坑

**1.** 给对象添加属性或者数组通过下标修改值的时候，直接通过给data里面的对象添加属性然后赋值，新添加的属性不是响应式的。

**原因：** Object.definepropety方法拦截不到这些操作，

【解决办法】通过Vue.set(对象，属性，值)这种方式就可以达到，对象新添加的属性是响应式的。数组也可以用splice()方法修改值

**2.** 在created操作dom的时候，是报错的，获取不到dom，这个时候实例Vue实例没有挂载 【解决办法】通过：Vue.nextTick(回调函数进行获取) ， 或者在mounted钩子里获取dom

**3.** 父组件调用子组件的方法，发送请求，修改子组件数据 ，子组件的视图没有更新。

**原因**：由于Vue的DOM操作是异步的，修改数据的时候子组件的DOM还没生成，this.$refs获取不到。

【解决办法】通过：Vue.nextTick() , 在nextTick里面去发送请求修改数据。

## 19. is这个特性你用过吗？是怎么用的？

**is的作用**

**解决html模板的限制**

比如ul里面嵌套li的写法是html语法的固定写法，如果想在ul里面嵌套自己的组件，但是html在渲染dom的时候，组件对ul来说并不是有效的dom。

解决办法

```xml
<ul>
  <li is='my-component'></li>
</ul>
复制代码
```

**动态组件(组件切换)**

componentName可以是在本页面已经注册的局部组件名和全局组件名, 也可以是一个组件的选项对象。当控制 componentName改变时就可以动态切换选择组件。

```ruby
<component :is="componentName"></component>
复制代码
```

## 20. Vue 项目进行 SEO 优化

Vue SPA单页面应用对SEO不太友好，当然也有相应的解决方案，下面列出几种SEO方案

1. **SSR服务器渲染**

   服务端渲染, 在服务端html页面节点, 已经解析创建完了, 浏览器直接拿到的是解析完成的页面解构

   关于服务器渲染：[Vue官网介绍](https://link.juejin.cn/?target=https%3A%2F%2Fssr.vuejs.org%2Fzh%2F%23%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E6%B8%B2%E6%9F%93-ssr-%EF%BC%9F) ，对Vue版本有要求，对服务器也有一定要求，需要支持nodejs环境。

   优势: 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面

   缺点: 服务器nodejs环境的要求, 且对原代码的改造成本高! `nuxt.js` (坑比较多, 做好踩坑的准备)

2. **静态化 (博客, 介绍性官网)**

   Nuxt.js 可以进行 generate 静态化打包, 缺点: 动态路由会被忽略。 `/users/:id`

   优势：

   - 编译打包时, 就会帮你处理, 纯静态文件，访问速度超快；
   - 对比SSR，不涉及到服务器负载方面问题；
   - 静态网页不宜遭到黑客攻击，安全性更高。

   不足：

   - 如果动态路由参数多的话不适用。

3. **预渲染 prerender-spa-plugin (插件)**

   如果你只是对少数页面需要做SEO处理（例如 / 首页, /about 关于等页面）

   预渲染是一个非常好的方式, 预渲染会在构建时, 简单的针对特定路由, 生成静态 HTML 文件 (打包时可以帮你解析静态化)

   优势: 设置预渲染简单, 对代码的改动小

   缺点: 只适合于做少数页面进行SEO的情况, 如果页面几百上千, 就不推荐了 (会打包很慢)

4. **使用Phantomjs `针对爬虫` 做处理**

   Phantomjs是一个基于webkit内核的无头浏览器，没有UI界面，就是一个浏览器，

   其内的点击、翻页等人为相关操作需要程序设计实现。

   这种解决方案其实是一种旁路机制，原理就是通过Nginx配置， 判断访问的来源UA是否是爬虫访问，

   如果是则将搜索引擎的爬虫请求转发到一个node server，再通过PhantomJS来解析完整的HTML，返回给爬虫

   优势：

   - 完全不用改动项目代码，按原本的SPA开发即可，对比开发SSR成本小太多了；
   - 对已用SPA开发完成的项目，这是不二之选。

   不足：

   - 部署需要node服务器支持；

   - 爬虫访问比网页访问要慢一些，因为定时要定时资源加载完成才返回给爬虫；(不影响用户的访问)

   - 如果被恶意模拟百度爬虫大量循环爬取，会造成服务器负载方面问题，

     解决方法是判断访问的IP，是否是百度官方爬虫的IP。

**小结:**

- 如果构建大型网站，如商城类 => SSR服务器渲染
- 如果只是正常公司官网, 博客网站等 => 预渲染/静态化/Phantomjs 都比较方便
- 如果是已用SPA开发完成的项目进行SEO优化，而且部署环境支持node服务器，使用 Phantomjs

## 21. 后端接口还没有开发好，怎么使用mock数据

Mock: 模拟数据；拦截请求；

```javascript
// mock/index.js
Mock.mock('/api/users', 'get', (req, res) => {
  // 通过 req 拿到前端的信息
  // 根据此信息返回对应的数据（Mock 的数据）
  res.send({ mock 的数据 })
})
复制代码
// main.js
import './mock'
复制代码
```

## 22. vue动画怎么实现

当vue中，显示隐藏，创建移除，一个元素或者一个组件的时候，可以通过transition实现动画。

- 进入（显示，创建）
- - v-enter-from 进入前
  - v-enter-active 进入中
  - v-enter-to 进入后
- 离开（隐藏，移除）
- - v-leave-from 进入前
  - v-leave-active 进入中
  - v-leave-to 进入后

两个步骤

1. 给要加动画的盒子,包裹一个transition标签
2. 在动画类名中写样式

多个transition使用不同动画，可以添加nam属性，name属性的值替换v即可。

## 23. Vue初始化过程中（new Vue(options)）都做了什么？

- 处理组件配置项；初始化根组件时进行了选项合并操作，将全局配置合并到根组件的局部配置上；初始化每个子组件时做了一些性能优化，将组件配置对象上的一些深层次属性放到 vm.$options 选项中，以提高代码的执行效率；
- 初始化组件实例的关系属性，比如`$parent、 $children、 $root、 $refs`等 处理自定义事件
- 调用 beforeCreate 钩子函数
- 初始化组件的 inject 配置项，得到 ret[key] = val 形式的配置对象，然后对该配置对象进行响应式处理，并代理每个 key 到 vm 实例上
- 数据响应式，处理 props、methods、data、computed、watch 等选项
- 解析组件配置项上的 provide 对象，将其挂载到 vm._provided 属性上 调用 created 钩子函数
- 如果发现配置项上有 el 选项，则自动调用`$mount`方法，也就是说有了 el 选项，就不需要再手动调用` $mount` 方法，反之，没提供 el 选项则必须调用 $mount

## 24. vue中data的属性可以和methods中方法同名吗，为什么？

可以同名，methods的方法名会被data的属性覆盖；调试台也会出现报错信息，但是不影响执行；

## 25. 什么是函数式组件？

函数式组件，我们可以理解为没有内部状态，没有生命周期钩子函数，没有`this`(不需要实例化的组件)。

**为什么使用函数式组件**

1. 最主要最关键的原因是函数式组件不需要实例化，无状态，没有生命周期，所以渲染性能要好于普通组件
2. 函数式组件结构比较简单，代码结构更清晰

**函数式组件与普通组件的区别**

函数式组件需要在声明组件是指定functional

函数式组件不需要实例化，所以没有`this`,`this`通过`render`函数的第二个参数来代替

函数式组件没有生命周期钩子函数，不能使用计算属性，watch等等

函数式组件不能通过$emit对外暴露事件，调用事件只能通过`context.listeners.click`的方式调用外部传入的事件

因为函数式组件是没有实例化的，所以在外部通过`ref`去引用组件时，实际引用的是`HTMLElement`

函数式组件的`props`可以不用显示声明，所以没有在`props`里面声明的属性都会被自动隐式解析为`prop`,而普通组件所有未声明的属性都被解析到`$attrs`里面，并自动挂载到组件根元素上面(可以通过`inheritAttrs`属性禁止)

## 26. Vue2怎么内部监听生命周期钩子(hook)

在`Vue`组件中，可以用过`$on`,`$once`去监听所有的生命周期钩子函数，如监听组件的`updated`钩子函数可以写成 `this.$on('hook:updated', () => {})`