# vue 笔记

## 基础


### Vue 实例

#### 创建一个 Vue 实例

每个 Vue 应用程序都是通过 Vue 函数创建出一个新的 Vue 实例开始的：
```
var vm = new Vue({
  // 选项
})
```

尽管没有完全遵循 MVVM 模式，但是 Vue 的设计仍然受到了它的启发。作为约定，通常我们使用变量 vm (ViewModel 的简称) 来表示 Vue 实例。

在创建一个 Vue 实例时，你会传入一个选项对象(options object)。本指南的大部分内容描述了，如果使用选项来达成预期的行为。可以在 API 参考文档中浏览选项(options)的完整列表。

Vue 应用程序由「一个使用 new Vue 创建的 Vue 根实例」、「嵌套的树结构（可选）」和「可复用的组件」组成。例如，一个 todo 应用程序的组件树可能如下所示：
```
根实例(Root Instance)
└─ TodoList
   ├─ TodoItem
   │  ├─ DeleteTodoButton
   │  └─ EditTodoButton
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```
稍后，我们将详细介绍组件系统。现在，只需要知道，所有 Vue 组件，都是 Vue 实例，并且组件都接收相同的选项对象（除了一些根实例特有(root-specific)的选项）。


#### `data` 和 `methods`

在创建 Vue 实例时，会将所有在 `data` 对象中找到的属性，都添加到 Vue 的响应式系统中。每当这些属性的值发生变化时，视图都会“及时响应”，并更新相应的新值。
```
// data 对象
var data = { a: 1 }

// 此对象将会添加到 Vue 实例上
var vm = new Vue({
  data: data
})

// 在实例上获取属性
// 将返回原始数据中的属性
vm.a == data.a // => true

// 设置实例上的属性，
// 也会影响原始数据
vm.a = 2
data.a // => 2

// ... 反之亦然
data.a = 3
vm.a // => 3
```

每当 `data` 对象发生变化，都会触发视图重新渲染。值得注意的是，如果实例已经创建，那么只有那些 `data` 中的原本就已经存在的属性，才是响应式的。

也就是说，如果在实例创建之后，添加一个新的属性，例如：
```
vm.b = 'hi'
```
然后，修改 b 不会触发任何视图更新。如果你已经提前知道，之后将会用到一个开始是空的或不存在的属性，你需要预先设置一些初始值。例如：
```
data: {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}
```
这里唯一的例外是，使用 `Object.freeze()` 来防止已有属性被修改，这也意味着响应式系统无法追踪变化。
```
var obj = {
  foo: 'bar'
}

Object.freeze(obj)

new Vue({
  el: '#app',
  data: obj
})
```

```
<div id="app">
  <p>{{ foo }}</p>
  <!-- 这将不再更新 `foo`! -->
  <button @click="foo = 'baz'">点我修改</button>
</div>
```
除了 `data` 属性， Vue 实例还暴露了一些有用的实例属性和方法。这些属性与方法都具有前缀 `$`，以便与用户定义(user-defined)的属性有所区分。例如：
```
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 此回调函数将在 `vm.a` 改变后调用
})
```
可以查阅 API 参考文档，来获取实例属性(instance property)和方法(methods)的完整列表。

#### 实例生命周期钩子函数

每个 Vue 实例在被创建之前，都要经过一系列的初始化过程

 - 例如，Vue 实例需要设置数据观察(set up data observation)、编译模板(compile the template)、在 DOM 挂载实例(mount the instance to the DOM)，以及在数据变化时更新 DOM(update the DOM when data change)。

在这个过程中，Vue 实例还会调用执行一些生命周期钩子函数，这样用户能够在特定阶段添加自己的代码。

例如，在实例创建后将调用 `created` 钩子函数：
```
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```
也有一些其它的钩子，会在实例生命周期的不同阶段调用，如 `mounted`、`updated` 和 `destroyed`。所有的钩子函数在调用时，其 `this` 上下文都会指向调用它的 Vue 实例。

注意，不要在选项属性或者回调函数中使用箭头函数（例如，`created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`）。

因为箭头函数会绑定父级上下文，所以 `this` 不会按照预期指向 Vue 实例，经常会产生一些错误，例如 `Uncaught TypeError: Cannot read property of undefined` 或者 `Uncaught TypeError: this.myMethod is not a function`。

#### 生命周期示意图

下面是实例生命周期示意图。你不需要现在就完全明白一切，但是，当你深入学习和组织架构的时候，这个示意图会是很有帮助的参考。

![生命周期示意图](https://vuefe.cn/images/lifecycle.png)



### 模板语法

Vue.js 使用基于 HTML 的模板语法，允许声明式地将要渲染的 DOM 和 Vue 实例中的 `data` 数据绑定。所有 Vue.js 的模板都是有效的 HTML，能够被遵循规范的浏览器和 HTML 解析器解析。

在底层的实现上，Vue 将模板编译为虚拟 DOM 渲染函数。结合响应式系统，在应用程序状态改变时，Vue 能够智能地找出重新渲染的最小数量的组件，并应用最少量的 DOM 操作。

如果你熟悉虚拟 DOM 的概念，并且倾向于使用原生 JavaScript，还可以不使用模板，而是直接编写渲染函数(render function)，具备可选的 JSX 语法支持。

#### 插值(Interpolations)

##### 文本(Text)

数据绑定最基本的形式，就是使用 “mustache” 语法（双花括号）的文本插值(text interpolation)：
```
<span>Message: {{ msg }}</span>
```
`mustache` 标签将会被替换为 `data` 对象上对应的 `msg` 属性的值。只要绑定的数据对象上的 `msg` 属性发生改变，插值内容也会随之更新。

也可以通过使用 `v-once` 指令，执行一次性插值，也就是说，在数据改变时，插值内容不会随之更新。但是请牢记，这也将影响到同一节点上的其他所有绑定：
```
<span v-once>这里的值永远不会改变：{{ msg }}</span>
```

##### 原始 HTML(Raw HTML)

双花括号语法，会将数据中的 HTML 转为纯文本后再进行插值。为了输出真正的 HTML，你需要使用 `v-html` 指令：
```
<p>使用双花括号语法：{{ rawHtml }}</p>
<p>使用 v-html 指令：<span v-html="rawHtml"></span></p>
```
`span` 中的内容，将会被替换为 `rawHtml` 属性的值，并且作为原始 HTML 插入 - 会忽略解析属性值中的数据绑定。

请注意，无法使用 `v-html` 来组合局部模板，这是因为 Vue 不是基于字符串(string-based)的模板引擎。反之，对于用户界面(UI)，组件更适合作为可重用和可组合的基本单位。

注意，在网站中动态渲染任意的 HTML 是非常危险的，因为这很容易导致网站受到 XSS 攻击。请只对可信内容使用 HTML 插值，绝对不要对用户提供的内容使用 HTML 插值。

##### 属性(Attributes)

不能在 Vue 模板中的 HTML 属性上使用双花括号语法(mustache)。而是应该使用 `v-bind` 指令：
```
<div v-bind:id="dynamicId"></div>
```
在属性是布尔类型的一些情况中，`v-bind` 的作用有点不同，只要值存在就会隐含为 true。在这个例子中：
```
<button v-bind:disabled="isButtonDisabled">Button</button>
```
如果 `isButtonDisabled` 的值为 `null`, `undefined` 或 `false`，`disabled` 属性甚至不会被包含在渲染后的 <button> 元素中。

##### 使用 JavaScript 表达式

到目前为止，我们只实现了将模板绑定到基本的属性键上。然而，Vue.js 实际上能够支持通过完整的 JavaScript 表达式，将模板与任意的数据绑定在一起：
```
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
这些表达式，将在所属的 Vue 实例的数据作用域下，作为 JavaScript 取值。有个限制是，每个绑定都只能包含单个表达式，所以以下示例都无法运行：
```
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也无法运行，请使用三元表达式 -->
{{ if (ok) return message }}

```

注意，模板表达式放置在沙盒中，只能访问全局变量的一个白名单列表，如 Math 和 Date。在模板表达式中，你不应该试图访问用户定义的全局变量。

#### 指令(Directives)

指令(directive)是带有 `v-` 前缀的特殊属性。指令属性的值期望接收的是单个 JavaScript 表达式（ `v-for ` 是例外情况，稍后我们再讨论）。

指令的职责是，当表达式的值改变时，将其产生的影响效果，响应式地作用于 DOM。回顾我们在介绍中看到的例子：
```
<p v-if="seen">Now you see me</p>
```
这里，`v-if` 指令将根据表达式 `seen` 的值的真假来移除/插入 <p> 元素。

##### 参数(Arguments)

一些指令能够接收一个“参数”，在指令名称之后以 : 表示。例如，`v-bind` 指令可以用于响应式地更新 HTML 属性：
```
<a v-bind:href="url"> ... </a>
```
这里 `href` 是参数，告知 `v-bind` 指令将元素的 `href` 属性与表达式 `url` 的值绑定在一起。

另一个示例是 `v-on` 指令，用于监听 DOM 事件：
```
<a v-on:click="doSomething"> ... </a>
```
这里，参数是要监听事件的名称。

##### 修饰符(Modifiers)

修饰符(modifier)是以 `.` 表示的特殊后缀，表明应当以某种特殊方式绑定指令。

例如，`.prevent` 修饰符告诉 `v-on` 指令，在触发事件后调用 `event.preventDefault()`：
```
<form v-on:submit.prevent="onSubmit"> ... </form>
```

#### 简写(Shorthands)

在模板中，`v-` 前缀作为非常符合直觉的提示，能够十分有效地标识出 Vue 特定(Vue-specific)属性。

当你在使用 Vue.js 为现有标签添加动态行为(dynamic behavior)时，`v-` 前缀很有帮助，然而，对于一些频繁用到的指令来说，就会感到使用繁琐。

同时，在构建由 Vue.js 管理所有模板的单页面应用程序(SPA - single page application)时，`v-` 前缀也变得没那么重要了。

因此，Vue.js 为 `v-bind` 和 `v-on` 这两个最常用的指令，提供了特定简写：

##### `v-bind` 简写

```
<!-- 完整语法 -->
<a v-bind:href="url"> ... </a>

<!-- 简写 -->
<a :href="url"> ... </a>
```

##### `v-on` 简写

```
<!-- 完整语法 -->
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

它们看起来可能与通常我们见到的 HTML 属性略有不同，但是，其实 : 和 @ 都是符合属性名称(attribute name)相关标准的有效字符，并且所有支持 Vue.js 的浏览器都能够正确解析它们。

此外，它们并不会出现在最终渲染的 HTML 标记中。简写语法是完全可选的书写方式，然而，随着你深入地了解它们的用法之后，你会领会到这种简单直接的简写方式，是非常赏心悦目的用法。


### computed 属性和 watcher

#### computed 属性

在模板中使用表达式是非常方便直接的，然而这只适用于简单的操作。在模板中放入太多的逻辑，会使模板过度膨胀和难以维护。例如：
```
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```
在这个地方，模板不再简洁和如声明式直观。你必须仔细观察一段时间才能意识到，这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中多次引用此处的翻转字符串时，就会更加难以处理。

这就是为什么对于所有复杂逻辑，你都应该使用 `computed` 属性(computed property)。

##### 基础示例
```
<div id="example">
  <p>初始 message 是："{{ message }}"</p>
  <p>计算后的翻转 message 是："{{ reversedMessage }}"</p>
</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 一个 computed 属性的 getter 函数
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

这里我们声明了一个 `computed` 属性 `reversedMessage`。然后为 `vm.reversedMessage` 属性提供一个函数，作为它的 `getter` 函数：
```
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```
你可以打开浏览器的控制台，然后如示例中操作 vm。会发现 `vm.reversedMessage` 的值总是依赖于 `vm.message` 的值。

你可以像绑定普通属性一样，将 `computed` 属性的数据，绑定(data-bind)到模板中的表达式上。Vue 能够意识到 `vm.reversedMessage` 依赖于 `vm.message`，也会在 `vm.message` 修改后，更新所有依赖于 `vm.reversedMessage` 的数据绑定。最恰到好处的部分是，我们是通过声明式来创建这种依赖关系：`computed` 属性的 `getter` 函数并无副作用(side effect)，因此也更加易于测试和理解。

##### computed 缓存 vs method 方法

你可能已经注意到，我们可以在表达式中通过调用 `method` 方法的方式，也能够实现与 `computed` 属性相同的结果：
```
<p>翻转 message 是："{{ reverseMessage() }}"</p>

// 在组件中
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```
这里不使用 `computed` 属性，而是在 `methods` 中定义一个相同的函数。对于最终结果，这两种方式确实恰好相同。

然而，细微的差异之处在于，`computed` 属性会基于它所依赖的数据进行缓存。

每个 `computed` 属性，只有在它所依赖的数据发生变化时，才会重新取值(re-evaluate)。这就意味着，只要 `message` 没有发生变化，多次访问 `computed` 属性 `reversedMessage`，将会立刻返回之前计算过的结果，而不必每次都重新执行函数。

这也同样意味着，如下的 `computed` 属性永远不会更新，因为 `Date.now()` 不是一个响应式的依赖数据：
```
computed: {
  now: function () {
    return Date.now()
  }
}
```

相比之下，每当触发重新渲染(re-render)时，`method` 调用方式将总是再次执行函数。

为什么我们需要将依赖数据缓存起来？假设一种场景，我们有一个高性能开销(expensive)的 `computed` 属性 A，在 `computed` 属性的 `getter` 函数内部，需要遍历循环一个巨大数组，并进行大量计算。然后还有其他 `computed` 属性直接或间接依赖于 A。如果没有缓存，我们将不可避免地多次执行 A 的 `getter` 函数，这远多余实际需要执行的次数！然而在某些场景下，你可能不希望有缓存，请使用 `method` 方法替代。


##### `computed` 属性和 `watch` 属性

Vue 其实还提供了一种更加通用的方式，来观察和响应 Vue 实例上的数据变化：`watch` 属性。

`watch` 属性是很吸引人的使用方式，然而，当你有一些数据需要随着另外一些数据变化时，过度滥用 `watch` 属性会造成一些问题 - 尤其是那些具有 AngularJS 开发背景的开发人员。

因此，更推荐的方式是，使用 `computed` 属性，而不是命令式(imperative)的 `watch` 回调函数。思考下面这个示例：
```
<div id="demo">{{ fullName }}</div>

var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```
以上代码是命令式和重复的。对比 computed 属性实现的版本：
```
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```
这样显得更优雅，对吗？

##### `computed` 属性中设置 `setter`

`computed` 属性默认只设置 `getter` 函数，不过在需要时，还可以提供 `setter` 函数：
```
// ...
computed: {
  fullName: {
    // getter 函数
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter 函数
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```
现在当你运行 `vm.fullName = 'John Doe'`，将会调用 `setter`，然后会对应更新 `vm.firstName` 和 `vm.lastName`。

#### watcher

虽然在大多数情况下，更适合使用 `computed` 属性，然而有些时候，还是需要一个自定义 `watcher`。这就是为什么 Vue 要通过 `watch` 选项，来提供一个更加通用的响应数据变化的方式。

当你需要在数据变化响应时，执行异步操作，或高性能消耗的操作，自定义 `watcher` 的方式就会很有帮助。

例如：
```
<div id="watch-example">
  <p>
    问一个答案是 yes/no 的问题：
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>


<!-- 对于 ajax 库(ajax libraries)和通用工具方法的集合(collections of general-purpose utility methods)来说， -->
<!-- 由于已经存在大量与其相关的生态系统， -->
<!-- 因此 Vue 核心无需重新创造，以保持轻量的文件体积。 -->
<!-- 同时这也可以使你自由随意地选择自己最熟悉的。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: '你要先提出问题，我才能给你答案！'
  },
  watch: {
    // 只要 question 发生改变，此函数就会执行
    question: function (newQuestion, oldQuestion) {
      this.answer = '等待输入停止……'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce 是由 lodash 提供的函数，
    // 在运行特别消耗性能的操作时，
    // 可以使用 _.debounce 来限制频率。
    // 在下面这种场景中，我们需要限制访问 yesno.wtf/api 的频率，
    // 等到用户输入完毕之后，ajax 请求才会发出。
    // 想要了解更多关于 _.debounce 函数（以及与它类似的 _.throttle）的详细信息，
    // 请访问：https://lodash.com/docs#debounce
    getAnswer: _.debounce(
      function () {
        if (this.question.indexOf('？') === -1) {
          this.answer = '问题通常需要包含一个中文问号。;-)'
          return
        }
        this.answer = '思考中……'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = '错误！API 无法处理。' + error
          })
      },
      // 这是用户停止输入操作后所等待的毫秒数。
      // （译者注：500毫秒之内，用户继续输入，则重新计时）
      500
    )
  }
})
</script>
```
在这个场景中，使用 `watch` 选项，可以使我们执行一个限制执行频率的（访问一个 API 的）异步操作，并且不断地设置中间状态，直到我们获取到最终的 `answer` 数据之后，才真正执行异步操作。而 `computed` 属性无法实现。

除了 `watch` 选项之外，还可以使用命令式(imperative)的 `vm.$watch ` API。





### 组件

#### 什么是组件？

组件 (Component) 是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。在较高层面上，组件是自定义元素，Vue.js 的编译器为它添加特殊功能。在有些情况下，组件也可以表现为用 is 特性进行了扩展的原生 HTML 元素。

组件系统是 Vue 的另一个重要概念，因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。仔细想想，几乎任意类型的应用界面都可以抽象为一个组件树：

![组件系统](https://vuefe.cn/images/components.png)

在一个大型应用中，有必要将整个应用程序划分为组件，以使开发更易管理。里有一个 (假想的) 例子，以展示使用了组件的应用模板是什么样的：

```
<div id="app">
  <app-nav></app-nav>
  <app-view>
    <app-sidebar></app-sidebar>
    <app-content></app-content>
  </app-view>
</div>
```
##### 与自定义元素的关系

你可能已经注意到 Vue 组件非常类似于自定义元素——它是 Web 组件规范的一部分，这是因为 Vue 的组件语法部分参考了该规范。例如 Vue 组件实现了 Slot API 与 is 特性。但是，还是有几个关键差别：

1、Web 组件规范仍然处于草案阶段，并且未被所有浏览器原生实现。相比之下，Vue 组件不需要任何 polyfill，并且在所有支持的浏览器 (IE9 及更高版本) 之下表现一致。必要时，Vue 组件也可以包装于原生自定义元素之内。

2、Vue 组件提供了纯自定义元素所不具备的一些重要功能，最突出的是跨组件数据流、自定义事件通信以及构建工具集成。


#### 组件使用

在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。

##### 全局注册
要注册一个全局组件，可以使用 Vue.component(tagName, options)。例如：
```
Vue.component('my-component', {
  // 选项对象
   template: "<div @age='alertAge'>{{ name }}</div>",
   data: function(){
       return {
           name: "李一岁",
           age: 2
       }
   },
   methods: {
       alertAge: function(){
           alert(this.name);
       }
   }
})
```
所有的 Vue 组件同时也都是 Vue 的实例，所以可接受相同的选项对象 (除了一些根级特有的选项) 并提供相同的生命周期钩子。

##### 局部注册
你不必把每个组件都注册到全局。你可以通过某个 Vue 实例/组件的实例选项 components 注册仅在其作用域中可用的组件：
```
<div id="example">
  <my-component></my-component>
</div>

var Child = {
  template: '<div>A custom component!</div>'
}

new Vue({
   el: '#example',
   components: {
       'my-component': Child
   }
})
```
这种封装也适用于其它可注册的 Vue 功能，比如指令。

##### 注意：
1、组件的模板只能有一个根元素。
错误示例：
```
template: `<p>这是第一个元素</p><p>这是第二个元素</p>`,
```
正确示例：
```
template: `<div><p>这是第一个元素</p><p>这是第二个元素</p></div>`,
```
2、组件中的data必须是函数

JS中```obj```对象的数据类型为引用数据类型，当多次使用同一个组件时，会共享同一个 data 对象，导致一个组件中数据改变引起其他组件数据发生改变。

我们可以通过为每个组件返回全新的数据对象来修复这个问题。

错误示例1：
```
Vue.component('my-component', {
  // 选项对象
   template: "<div>{{ name }}</div>",
   data: {
      name: "李一岁",
      age: 2
   }
})
```
错误示例2：
```
var myData = {
    name: "李一岁",
    age: 2
}

Vue.component('my-component', {
  // 选项对象
   template: "<div>{{ name }}</div>",
   data: function(){
       return myData
   },
})
```
正确示例：
```
Vue.component('my-component', {
  // 选项对象
   template: "<div>{{ name }}</div>",
   data: function(){
       return {
           name: "李一岁",
           age: 2
       }
   }
})
```
##### 字符串模板(string template)和非字符串模板(non-string template)

1. 字符串模板(string template)

- `<script type="text/x-template">`
- JavaScript 内联模板字符串
- `.vue` 组件

2. 非字符串模板(non-string template)

- DOM 模板


##### DOM 模板解析注意事项
当使用 DOM 作为模板时 (例如，使用 el 选项来把 Vue 实例挂载到一个已有内容的元素上)，你会受到 HTML 本身的一些限制，因为 Vue 只有在浏览器解析、规范化模板之后才能获取其内容。尤其要注意，像 `<ul>、<ol>、<table>、<select>` 这样的元素里允许包含的元素有限制，而另一些像 `<option>` 这样的元素只能出现在某些特定元素的内部。

在自定义组件中使用这些受限制的元素时会导致一些问题，例如：

```
<table>
  <my-row>...</my-row>
</table>
```

自定义组件 <my-row> 会被当作无效的内容，因此会导致错误的渲染结果。变通的方案是使用特殊的 is 特性：

```
<table>
  <tr is="my-row"></tr>
</table>
```

应当注意，如果使用来自以下来源之一的字符串模板，则没有这些限制：

- <script type="text/x-template">
- JavaScript 内联模板字符串
- .vue 组件

其中，前两个模板都不是Vue官方推荐的，所以一般情况下，只有单文件组件.vue可以忽略这种情况。

##### 组件组合

组件设计初衷就是要配合使用的，最常见的就是形成父子组件的关系：组件 A 在它的模板中使用了组件 B。它们之间必然需要相互通信：父组件可能要给子组件下发数据，子组件则可能要将它内部发生的事情告知父组件。然而，通过一个良好定义的接口来尽可能将父子组件解耦也是很重要的。这保证了每个组件的代码可以在相对隔离的环境中书写和理解，从而提高了其可维护性和复用性。

在 Vue 中，父子组件的关系可以总结为 prop 向下传递，事件向上传递。父组件通过 prop 给子组件下发数据，子组件通过事件给父组件发送消息。看看它们是怎么工作的。

![组件组合](https://cn.vuejs.org/images/props-events.png)

###### 父组件给子组件传递数据

组件实例的作用域是孤立的。这意味着不能 (也不应该) 在子组件的模板内直接引用父组件的数据。父组件的数据需要通过 prop 才能下发到子组件中。

子组件要显式地用 props 选项声明它预期的数据：

```
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 也可以在模板中使用
  // 同样也可以在 vm 实例中通过 this.message 来使用
  template: '<span>{{ message }}</span>'
})
```

然后我们可以这样向它传入一个普通字符串：

```
<child message="hello!"></child>
```
注意：


HTML 特性是不区分大小写的。所以，当使用的不是字符串模板时，camelCase (驼峰式命名) 的 prop 需要转换为相对应的 kebab-case (短横线分隔式命名)：

```
Vue.component('child', {
  // 在 JavaScript 中使用 camelCase
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})

<!-- 在 HTML 中使用 kebab-case -->
<child my-message="hello!"></child>
```




###### 子组件向父组件传递数据

子组件通过 Vue 的自定义事件跟父组件通信。

另外，父组件可以在使用子组件的地方直接用 v-on 来监听子组件触发的事件。
```
   <div id="message-event-example" class="demo">
     <p v-for="msg in messages">{{ msg }}</p>
     <button-message v-on:message="handleMessage"></button-message>
   </div>

   <script>
     Vue.component('button-message', {
       template: `<div>
         <input type="text" v-model="message" />
         <button v-on:click="handleSendMessage">Send</button>
       </div>`,
       data: function () {
         return {
           message: 'test message'
         }
       },
       methods: {
         handleSendMessage: function () {
           this.$emit('message', { message: this.message })
         }
       }
     })

     new Vue({
       el: '#message-event-example',
       data: {
         messages: []
       },
       methods: {
         handleMessage: function (payload) {
           this.messages.push(payload.message)
         }
       }
     })
   </script>
```
注意： 使用 `v-on` 来监听子组件触发的事件时，不应该传进任何参数，否则父组件无法通过 `payload` 获取到子组件传递的参数


##### 响应式系统

在创建 Vue 实例时，会将所有在 data 对象中找到的属性，都添加到 Vue 的响应式系统中。每当这些属性的值发生变化时，视图都会“及时响应”，并更新相应的新值。

值得注意的是，如果实例已经创建，那么只有那些 data 中的原本就已经存在的属性，才是响应式的。如果你已经提前知道，之后将会用到一个开始是空的或不存在的属性，你需要预先设置一些初始值。

```
data: {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}
```

使用 Object.freeze() 可以防止已有属性被修改，这也意味着响应式系统无法追踪变化。


#### 动态组件

可以让多个组件使用同一个挂载点，然后动态地在它们之间切换。要实现此效果，我们可以使用 Vue 保留的 `<component>` 元素，将多个组件动态地绑定到 `<component>` 元素的 is 属性上：
```
<component v-bind:is="currentView">
  <!-- 组件在 vm.currentview 变化时改变！ -->
</component>

var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```
也可以直接绑定到组件对象上：
```
 var Home = {
   template: '<p>Welcome home!</p>'
 }

 var vm = new Vue({
   el: '#example',
   data: {
     currentView: Home
   }
 })
```

##### `keep-alive`

如果把切换出去的组件保留在内存中，可以保留它的状态或避免重新渲染。为此可以添加一个 `keep-alive` 指令参数：
```
<keep-alive>
  <component :is="currentView">
    <!-- 非活动组件将被缓存！ -->
  </component>
</keep-alive>
```






#### Prop

##### 动态 Prop

与绑定到任何普通的 HTML 特性相类似，我们可以用 v-bind 来动态地将 prop 绑定到父组件的数据。每当父组件的数据变化时，该变化也会传导给子组件：

```
<div id="prop-example-2">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>


Vue.component('child', {
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
new Vue({
  el: '#prop-example-2',
  data: {
    parentMsg: 'Message from parent'
  }
})
```
使用 `v-bind` 的缩写语法：
```
<child :my-message="parentMsg"></child>
```

如果你想把一个对象的所有属性作为 prop 进行传递，可以使用不带任何参数的 `v-bind `(即用 `v-bind` 而不是 `v-bind:prop-name`)。例如，已知一个 `todo ` 对象：
```
todo: {
  text: 'Learn Vue',
  isComplete: false
}
```
传递`todo`对象给子组件：
```
<todo-item v-bind="todo"></todo-item>
```
等价于：
```
<todo-item
  v-bind:text="todo.text"
  v-bind:is-complete="todo.isComplete"
></todo-item>
```

##### Prop 验证

我们可以为组件的 prop 指定验证规则。如果传入的数据不符合要求，Vue 会发出警告。这对于开发给他人使用的组件非常有用。

要指定验证规则，需要用对象的形式来定义 prop，而不能用字符串数组：
```
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 指允许任何类型)
    propA: Number,
    // 可能是多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数值且有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```
`type` 可以是下面原生构造器：
- String
- Number
- Boolean
- Function
- Object
- Array
- Symbol

`type` 也可以是一个自定义构造器函数，使用 instanceof 检测。

当 prop 验证失败，Vue 会抛出警告 (如果使用的是开发版本)。注意 prop 会在组件实例创建之前进行校验，所以在 `default` 或 `validator` 函数里，诸如 `data`、`computed` 或 `methods` 等实例属性还无法使用。

自定义Person构造器
```
 function Person(name, age) {
    this.name = name
    this.age = age
  }

  Vue.component('my-component', {
   template: `<div>名字: {{ person-prop.name }}， 年龄： {{ person-prop.age }} </div>`,
   props: {
     person-prop: {
       type: Person     // 指定类型
     }
   }
  })
 new Vue({
   el: '#app2',
   data: {
     person: 2		// 传入Number类型会报错
   }
 })
```


##### 字面量语法 vs 动态语法

比较常出现的错误是使用字面量语法传递数值：
```
<!-- 传递了一个字符串 "1" -->
<comp some-prop="1"></comp>
```
因为它是一个字面量 prop，它的值是字符串 "1" 而不是一个数值。如果想传递一个真正的 JavaScript 数值，则需要使用 v-bind，从而让它的值被当作 JavaScript 表达式计算：
```
<!-- 传递真正的数值 -->
<comp v-bind:some-prop="1"></comp>
```

##### 单向数据流

Prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是反过来不会。这是为了防止子组件无意间修改了父组件的状态，来避免应用的数据流变得难以理解。

另外，每次父组件更新时，子组件的所有 prop 都会更新为最新值。这意味着你不应该在子组件内部改变 prop。如果你这么做了，Vue 会在控制台给出警告。

在两种情况下，我们很容易忍不住想去修改 prop 中数据：

1. Prop 作为初始值传入后，子组件想把它当作局部数据来用；

2. Prop 作为原始数据传入，由子组件处理成其它数据输出。

对这两种情况，正确的应对方式是：

1. 定义一个局部变量，并用 prop 的值初始化它：
```
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
```
2. 定义一个计算属性，处理 prop 的值并返回：
```
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
注意在 JavaScript 中对象和数组是引用类型，指向同一个内存空间，如果 prop 是一个对象或数组，在子组件内部改变它会影响父组件的状态。

#### 非 Prop 特性

所谓非 prop 特性，就是指它可以直接传入组件，而不需要定义相应的 prop。

尽管为组件定义明确的 prop 是推荐的传参方式，组件的作者却并不总能预见到组件被使用的场景。所以，组件可以接收任意传入的特性，这些特性都会被添加到组件的根元素上。

例如，假设我们使用了第三方组件 `bs-date-input`，它包含一个 Bootstrap 插件，该插件需要在 `input` 上添加 `data-3d-date-picker` 这个特性。这时可以把特性直接添加到组件上 (不需要事先定义 prop)：
```
<bs-date-input data-3d-date-picker="true"></bs-date-input>
```
添加属性 `data-3d-date-picker="true"` 之后，它会被自动添加到 `bs-date-input` 的根元素上。

##### 替换/合并现有的特性

假设这是 `bs-date-input` 的模板：
```
<input type="date" class="form-control">
```
为了给该日期选择器插件增加一个特殊的主题，我们可能需要增加一个特殊的 class，比如：
```
<bs-date-input
  data-3d-date-picker="true"
  class="date-picker-theme-dark"
></bs-date-input>
```
在这个例子当中，我们定义了两个不同的 class 值：

- `form-control`，来自组件自身的模板
- `date-picker-theme-dark`，来自父组件

对于多数特性来说，传递给组件的值会覆盖组件本身设定的值。即例如传递 `type="large"` 将会覆盖 `type="date"` 且有可能破坏该组件！所幸我们对待 `class` 和 `style` 特性会更聪明一些，这两个特性的值都会做合并 (merge) 操作，让最终生成的值为：`form-control date-picker-theme-dark`。

#### 自定义事件

我们知道，父组件使用 prop 传递数据给子组件。但子组件怎么跟父组件通信呢？这个时候 Vue 的自定义事件系统就派得上用场了。

##### 使用 `v-on` 绑定自定义事件

每个 Vue 实例都实现了事件接口，即：

- 使用 `$on(eventName)` 监听事件
- 使用 `$emit(eventName, optionalPayload)` 触发事件

> Vue 的事件系统与浏览器的 `EventTarget API` 有所不同。尽管它们运行起来类似，但是 `$on` 和 `$emit` 并不是 `addEventListener` 和 `dispatchEvent` 的别名。

示例：
```
vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')
// => "hi"
```

另外，父组件可以在使用子组件的地方直接用 v-on 来监听子组件触发的事件。(子组件与父组件通信)

> 不能用 $on 监听子组件释放的事件，而必须在模板里直接用 v-on 绑定，参见下面的例子。

```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>


Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```
在本例中，子组件已经和它外部完全解耦了。它所做的只是报告自己的内部事件，因为父组件可能会关心这些事件。请注意这一点很重要。


这里有一个如何使用载荷 (payload) 数据的示例：
```
<div id="message-event-example" class="demo">
  <p v-for="msg in messages">{{ msg }}</p>
  <button-message v-on:message="handleMessage"></button-message>
</div>

Vue.component('button-message', {
  template: `<div>
    <input type="text" v-model="message" />
    <button v-on:click="handleSendMessage">Send</button>
  </div>`,
  data: function () {
    return {
      message: 'test message'
    }
  },
  methods: {
    handleSendMessage: function () {
      this.$emit('message', { message: this.message })
    }
  }
})

new Vue({
  el: '#message-event-example',
  data: {
    messages: []
  },
  methods: {
    handleMessage: function (payload) {
      this.messages.push(payload.message)
    }
  }
})
```
第二个示例的重点在于子组件仍然是完全和外界解耦的。它做的事情全都是记录其自身的活动，活动记录是包括一份传入事件触发器的载荷数据在内的，只是为了展示父组件可以不关注的一个场景。

##### 给组件绑定原生事件

有时候，你可能想在某个组件的根元素上监听一个原生事件。可以使用 `v-on` 的修饰符 `.native`。例如：
```
<my-component v-on:click.native="doTheThing"></my-component>
```
而如果你没有添加 `.native` 修饰符的话，`my-component`组件时监听不到 `click` 事件的。
```
<my-component v-on:click="doTheThing"></my-component> // doTheThing事件不会触发
```

##### `.sync` 修饰符（2.3.0+）

在一些情况下，我们可能会需要对一个 prop 进行“双向绑定”。事实上，这正是 Vue 1.x 中的 `.sync` 修饰符所提供的功能。当一个子组件改变了一个带 `.sync` 的 prop 的值时，这个变化也会同步到父组件中所绑定的值。这很方便，但也会导致问题，因为它破坏了单向数据流。由于子组件改变 prop 的代码和普通的状态改动代码毫无区别，当光看子组件的代码时，你完全不知道它何时悄悄地改变了父组件的状态。这在 debug 复杂结构的应用时会带来很高的维护成本。

上面所说的正是我们在 2.0 中移除 `.sync` 的理由。但是在 2.0 发布之后的实际应用中，我们发现  `.sync` 还是有其适用之处，比如在开发可复用的组件库时。我们需要做的只是让子组件改变父组件状态的代码更容易被区分。

从 2.3.0 起我们重新引入了 `.sync` 修饰符，但是这次它只是作为一个编译时的语法糖存在。它会被扩展为一个自动更新父组件属性的 v-on 监听器。

如下代码
```
<comp :foo.sync="bar"></comp>
```
会被扩展为：
```
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```

当子组件需要更新 `foo` 的值时，它需要显式地触发一个更新事件：
```
this.$emit('update:foo', newValue)
```

当使用一个对象一次性设置多个属性的时候，这个 `.sync` 修饰符也可以和 `v-bind` 一起使用：
```
<comp v-bind.sync="{ foo: 1, bar: 2 }"></comp>
```
这个例子会为 foo 和 bar 同时添加用于更新的 v-on 监听器。(然而并不知道怎么触发父组件更新数据)

##### 使用自定义事件的表单输入组件

自定义事件可以用来创建自定义的表单输入组件，使用 `v-model` 来进行数据双向绑定。要牢记：
```
<input v-model="something">
```
这不过是以下示例的语法糖：
```
<input
  v-bind:value="something"
  v-on:input="something = $event.target.value">
```

所以在组件中使用时
```
<custom-input v-model="something"></custom-input>
```
相当于下面的简写：
```
<custom-input
  v-bind:value="something"
  v-on:input="something = arguments[0]">
</custom-input>
```
要让组件的 `v-model` 生效，它应该 (从 2.2.0 起是可配置的)：

- 接受一个 `value` prop
- 在有新的值时触发 `input` 事件并将新值作为参数

 ...0.0？

我们来看一个非常简单的货币输入的自定义控件：
```
<currency-input v-model="price"></currency-input>

Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    // 不是直接更新值，而是使用此方法来对输入值进行格式化和位数限制
    updateValue: function (value) {
      var formattedValue = value
        // 删除两侧的空格符
        .trim()
        // 保留 2 位小数
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      // 如果值尚不合规，则手动覆盖为合规的值
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // 通过 input 事件带出数值
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

##### 自定义组件的 `v-model` （2.2.0 新增）

默认情况下，一个组件的 `v-model` 会使用 `value` prop 和 `input` 事件。但是诸如单选框、复选框之类的输入类型可能把 `value` 用作了别的目的。`model` 选项可以避免这样的冲突：
```
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean,
    // 这样就允许拿 `value` 这个 prop 做其它事了
    value: String
  },
  // ...
})
```

```
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```
上述代码等价于：
```
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="some value">
</my-checkbox>
```

注意你仍然需要显式声明 checked 这个 prop。


##### 非父子组件的通信

有时候，非父子关系的两个组件之间也需要通信。在简单的场景下，可以使用一个空的 Vue 实例作为事件总线：
```
var bus = new Vue()


// 触发组件 A 中的事件
bus.$emit('id-selected', 1)


// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```
在复杂的情况下，我们应该考虑使用专门的状态管理模式。


#### 使用插槽分发内容

在使用组件时，我们常常要像这样组合它们：
```
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```
注意两点：

1. `<app>` 组件不知道它会收到什么内容。这是由使用 `<app>` 的父组件决定的。

2. `<app>` 组件很可能有它自己的模板。

为了让组件可以组合，我们需要一种方式来混合父组件的内容与子组件自己的模板。这个过程被称为内容分发 (即 Angular 用户熟知的“transclusion”)。Vue.js 实现了一个内容分发 API，参照了当前 Web Components 规范草案，使用特殊的 `<slot>` 元素作为原始内容的插槽。

##### 编译作用域

在深入内容分发 API 之前，我们先明确内容在哪个作用域里编译。假定模板为：
```
<child-component>
  {{ message }}
</child-component>
```
`message` 应该绑定到父组件的数据，还是绑定到子组件的数据？答案是父组件。组件作用域简单地说是:
> 父组件模板的内容在父组件作用域内编译；子组件模板的内容在子组件作用域内编译。

一个常见错误是试图在父组件模板内将一个指令绑定到子组件的属性/方法：
```
<!-- 无效 -->
<child-component v-show="someChildProperty"></child-component>
```
假定 `someChildProperty` 是子组件的属性，上例不会如预期那样工作。父组件模板并不感知子组件的状态。

如果要绑定子组件作用域内的指令到一个组件的根节点，你应当在子组件自己的模板里做：
```
Vue.component('child-component', {
  // 有效，因为是在正确的作用域内
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```
类似地，被分发的内容会在父作用域内编译。

##### 单个插槽
除非子组件模板包含至少一个 `<slot>` 插口，否则父组件的内容将会被丢弃。当子组件模板只有一个没有属性的插槽时，父组件传入的整个内容片段将插入到插槽所在的 DOM 位置，并替换掉插槽标签本身。

最初在 `<slot>` 标签中的任何内容都被视为备用内容。备用内容在子组件的作用域内编译，并且只有在宿主元素为空，且没有要插入的内容时才显示备用内容。

假定 `my-component` 组件有如下模板：
```
<div>
  <h2>我是子组件的标题</h2>
  <slot>
    只有在没有要分发的内容时才会显示。
  </slot>
</div>
```
父组件模板：
```
<div>
  <h1>我是父组件的标题</h1>
  <my-component>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </my-component>
</div>
```
渲染结果：
```
<div>
  <h1>我是父组件的标题</h1>
  <div>
    <h2>我是子组件的标题</h2>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </div>
</div>
```

##### 具名插槽

`<slot>` 元素可以用一个特殊的特性 `name` 来进一步配置如何分发内容。多个插槽可以有不同的名字。具名插槽将匹配内容片段中有对应 `slot` 特性的元素。

仍然可以有一个匿名插槽，它是默认插槽，作为找不到匹配的内容片段的备用插槽。如果没有默认插槽，这些找不到匹配的内容片段将被抛弃。

例如，假定我们有一个 `app-layout` 组件，它的模板为：
```
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
父组件模板：
```
<app-layout>
  <h1 slot="header">这里可能是一个页面标题</h1>

  <p>主要内容的一个段落。</p>
  <p>另一个主要段落。</p>

  <p slot="footer">这里有一些联系信息</p>
</app-layout>
```
渲染结果为：
```
<div class="container">
  <header>
    <h1>这里可能是一个页面标题</h1>
  </header>
  <main>
    <p>主要内容的一个段落。</p>
    <p>另一个主要段落。</p>
  </main>
  <footer>
    <p>这里有一些联系信息</p>
  </footer>
</div>
```
在设计组合使用的组件时，内容分发 API 是非常有用的机制。

##### 作用域插槽 （2.1.0+ 新增）

作用域插槽是一种特殊类型的插槽，用于将要渲染的元素(already-rendered-elements)，替换为一个（能够传递数据的）可重用模板。

在子组件中，将数据传递到插槽，就像你将 props 传递给组件一样：
```
<div class="child">
  <slot text="hello from child"></slot>
</div>
```
在父级中，必须存在具有特殊属性 `slot-scope` 的 `<template>` 元素，表示它是作用域插槽的模板。`slot-scope` 的值对应一个临时变量名，此变量接收从子组件中传递的 `props` 对象：
```
<div class="parent">
  <child>
    <template slot-scope="props">
      <span>hello from parent</span>
      <span>{{ props.text }}</span>
    </template>
  </child>
</div>
```
如果我们渲染以上结果，得到的输出会是：
```
<div class="parent">
  <div class="child">
    <span>hello from parent</span>
    <span>hello from child</span>
  </div>
</div>
```

作用域插槽更具代表性的用例是列表组件，允许组件自定义应该如何渲染列表每一项：(在 2.5.0+ 中，`slot-scope` 不再局限于 `<template>`，可以在任何元素或组件上使用。)
```
<my-awesome-list :items="items">
  <!-- 作用域插槽也可以是具名的 -->
  <li
    slot="item"
    slot-scope="props"
    class="my-fancy-item">
    {{ props.text }}
  </li>
</my-awesome-list>
```
列表组件的模板：
```
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- 这里是备用内容 -->
  </slot>
</ul>
```

解构

`slot-scope` 的值实际上是一个有效的 JavaScript 表达式，可以出现在函数签名中的参数所在位置。这意味着在支持的环境中（在单个文件组件或在现代浏览器），表达式中也可以使用 ES2015 解构：
```
<child>
  <span slot-scope="{ text }">{{ text }}</span>
</child>
```

一个完整的例子：
```
<!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Vue作用域插槽</title>
        <script src="https://cdn.bootcss.com/vue/2.3.4/vue.js"></script>
    </head>
    <body>
        <div id="app2">
            <!-- 组件使用者只需传递users数据即可 -->
            <my-stripe-list :items="users" odd-bgcolor="#D3DCE6" even-bgcolor="#E5E9F2">
                <!-- props对象接收来自子组件slot的$index参数 -->
                <template slot="cont" scope="props">
                    <span>{{users[props.$index].id}}</span>
                    <span>{{users[props.$index].name}}</span>
                    <span>{{users[props.$index].age}}</span>
                    <!-- 这里可以自定[编辑][删除]按钮的链接和样式 -->
                    <a :href="'#edit/id/'+users[props.$index].id">编辑</a>
                    <a :href="'#del/id/'+users[props.$index].id">删除</a>
                </template>
            </my-stripe-list>
        </div>
        <script>
            Vue.component('my-stripe-list', {
                /*slot的$index可以传递到父组件中*/
                template: `
                    <div>
                        <div v-for="(item, index) in items" style="line-height:2.2;" :style="index % 2 === 0 ? 'background:'+oddBgcolor : 'background:'+evenBgcolor">
                            <slot name="cont" :$index="index"></slot>
                        </div>
                    </div>
                `,
                props: {
                    items: Array,
                    oddBgcolor: String,
                    evenBgcolor: String
                }
            });
            new Vue({
                el: '#app2',
                data: {
                    users: [
                        {id: 1, name: '张三', age: 20},
                        {id: 2, name: '李四', age: 22},
                        {id: 3, name: '王五', age: 27},
                        {id: 4, name: '张龙', age: 27},
                        {id: 5, name: '赵虎', age: 27}
                    ]
                }
            });
        </script>
    </body>
</html>
```

#### 杂项

##### 组件命名约定

当注册组件（或者 props）时，可以使用 kebab-case，camelCase，或 PascalCase。
```
// 在组件定义中
components: {
  // 使用 kebab-case 形式注册
  'kebab-cased-component': { /* ... */ },
  // register using camelCase
  'camelCasedComponent': { /* ... */ },
  // register using PascalCase
  'PascalCasedComponent': { /* ... */ }
}
```
在 HTML 模板中，请使用 kebab-case 形式：
```
<!-- 在HTML模板中始终使用 kebab-case -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<pascal-cased-component></pascal-cased-component>
```
当使用字符串模式时，可以不受 HTML 的 `case-insensitive` 限制。这意味实际上在模板中，你可以使用下面的方式来引用你的组件：

- kebab-case
- camelCase 或 kebab-case 如果组件已经被定义为 camelCase
- kebab-case，camelCase 或 PascalCase 如果组件已经被定义为 PascalCase

```
components: {
  'kebab-cased-component': { /* ... */ },
  camelCasedComponent: { /* ... */ },
  PascalCasedComponent: { /* ... */ }
}
```

```
<kebab-cased-component></kebab-cased-component>

<camel-cased-component></camel-cased-component>
<camelCasedComponent></camelCasedComponent>

<pascal-cased-component></pascal-cased-component>
<pascalCasedComponent></pascalCasedComponent>
<PascalCasedComponent></PascalCasedComponent>
```
这意味着 PascalCase 是最通用的 声明约定 而 kebab-case 是最通用的 使用约定。

如果组件未经 slot 元素传递内容，你甚至可以在组件名后使用 / 使其自闭合：
```
<my-component/>
```
当然，这只在字符串模板中有效。因为自闭的自定义元素是无效的 HTML，浏览器原生的解析器也无法识别它。


##### 编写可复用组件

在编写组件时，记住是否要复用组件有好处。一次性组件跟其它组件紧密耦合没关系，但是可复用组件应当定义一个清晰的公开接口。

Vue 组件的 API 来自三部分 - props, events 和 slots ：

- Props 允许外部环境传递数据给组件
- Events 允许组件对外部环境产生副作用(side effects)
- Slots 允许外部环境将额外的内容组合在组件中。

使用 `v-bind` 和 `v-on` 的简写语法，模板的缩进清楚且简洁：
```
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

##### 子组件索引

尽管有 props 和 events，但是有时仍然需要在 JavaScript 中直接访问子组件。为此可以使用 `ref` 为子组件指定一个索引 ID。例如：
```
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

```
var parent = new Vue({ el: '#parent' })
// 访问子组件
var child = parent.$refs.profile
```
当 `ref` 和 `v-for` 一起使用时，ref 是一个数组，包含相应的子组件。

> `$refs` 只在组件渲染完成后才填充，并且它是非响应式的。它仅仅作为一个直接访问子组件的应急方案 - 应当避免在模板或计算属性中使用 `$refs`。

##### 异步组件

在大型应用程序中，我们可能需要将应用程序拆分为多个小的分块(chunk)，并且只在实际用到时，才从服务器加载组件。为了让异步按需加载组件这件事变得简单，Vue 允许将组件定义为一个异步解析组件定义的工厂函数。Vue 只在实际需要渲染组件时，才触发调用工厂函数，并且会将结果缓存起来，用于将来再次渲染。例如

```
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Pass the component definition to the resolve callback
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```
工厂函数接收一个 `resolve` 回调函数，在从服务器接收到组件定义时调用。也可以调用 `reject(reason)` 表明加载失败。这里的 `setTimeout` 是为了用于演示；如何异步获取组件定义完全取决于你的实现。要使用异步组件，一个比较推荐的方式是配合 `webpack` 代码分离功能：
```
Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 require 语法
  // 将指示 webpack 自动将构建完成的代码，
  // 拆分到不同的 bundle 中，然后通过 Ajax 请求加载。
  require(['./my-async-component'], resolve)
})
```
还可以在工厂函数中返回一个 `Promise`，所以配合 `webpack 2 + ES2015` 语法，你可以这样实现：
```
Vue.component(
  'async-webpack-example',
  // `import` 函数返回一个 `Promise`.
  () => import('./my-async-component')
)
```
当使用局部注册时，你也可以直接提供一个返回 Promise 的函数：
```
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```
> 如果你是需要使用异步组件的 Browserify 用户，可能就无法使用异步组件了，它的作者已经明确表示很不幸 Browserify 是不支持异步加载的。Browserify 社区找到一些解决方案，这可能有助于现有的复杂应用程序实现异步加载。对于所有其他场景，我们推荐使用 webpack 所内置的表现优异异步支持。

##### 高级异步组件 （2.3.0+ 新增）

自 2.3 起，异步组件的工厂函数也可以返回一个如下的对象：
```
const AsyncComp = () => ({
  // 需要加载的组件. 应当是一个 Promise
  component: import('./MyComp.vue'),
  // loading 时应当渲染的组件
  loading: LoadingComp,
  // 出错时渲染的组件
  error: ErrorComp,
  // 渲染 loading 组件前的等待时间。默认：200ms.
  delay: 200,
  // 最长等待时间。超出此时间
  // 则渲染 error 组件。默认：Infinity
  timeout: 3000
})
```
注意，当一个异步组件被作为 `vue-router` 的路由组件使用时，这些高级选项都是无效的，因为在路由切换前就会提前加载所需要的异步组件。另外，如果你要在路由组件中上述写法，需要使用 `vue-router` 2.4.0+。

##### 递归组件

组件在它的模板内可以递归地调用自己。不过，只有当它有 name 选项时才可以这么做：
```
name: 'unique-name-of-my-component'
```

当你利用 `Vue.component` 全局注册了一个组件，全局的 ID 会被自动设置为组件的 `name`。
```
Vue.component('unique-name-of-my-component', {
  // ...
})
```

如果稍有不慎，递归组件可能导致死循环：
```
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```
上面组件会导致一个“max stack size exceeded”错误，所以要确保递归调用有终止条件 (比如递归调用时使用 `v-if` 并最终解析为 `false`)。


##### 组件间的循环引用

假设你正在构建一个文件目录树，像在 `Finder` 或资源管理器中。你可能有一个 `tree-folder` 组件：

```
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```
以及一个 `tree-folder-contents` 组件：
```
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```
当你仔细看时，会发现在渲染树上这两个组件同时为对方的父节点和子节点——这是矛盾的！当使用 `Vue.component` 将这两个组件注册为全局组件的时候，框架会自动为你解决这个矛盾。如果你已经是这样做的，就跳过下面这段吧。

然而，如果你使用诸如 webpack 或者 Browserify 之类的模块化管理工具来 `require/import` 组件的话，就会报错了：

```
Failed to mount component: template or render function not defined.
```
为了解释为什么会报错，简单的将上面两个组件称为 A 和 B。模块系统看到它需要 A，但是首先 A 需要 B，但是 B 需要 A，而 A 需要 B，循环往复。因为不知道到底应该先解析哪个，所以将会陷入无限循环。要解决这个问题，我们需要在其中一个组件中告诉模块化管理系统：“A 虽然最后会用到 B，但是不需要优先导入 B”。

在我们的例子中，可以选择让 `tree-folder` 组件中来做这件事。我们知道引起矛盾的子组件是 `tree-folder-contents`，所以我们要等到 `beforeCreate` 生命周期钩子中才去注册它：
```
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue').default
}
```
问题解决了！

##### 内联模板

如果子组件有 `inline-template` 特性，组件将把它的内容当作它的模板，而不是把它当作分发内容。这让模板编写起来更灵活
```
<my-component inline-template>
  <div>
    <p>这些将作为组件自身的模板。</p>
    <p>而非父组件透传进来的内容。</p>
  </div>
</my-component>
```
但是 `inline-template` 让模板的作用域难以理解。使用 `template` 选项在组件内定义模板或者在 ` .vue` 文件中使用 `template` 元素才是最佳实践。

##### X-Template

另一种定义模板的方式是在 JavaScript 标签里使用 `text/x-template` 类型，并且指定一个 id。例如：
```
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>

Vue.component('hello-world', {
  template: '#hello-world-template'
})
```
这在有很多大模板的演示应用或者特别小的应用中可能有用，其它场合应该避免使用，因为这将模板和组件的其它定义分离了。

##### 对低开销的静态组件使用 `v-once`
尽管在 Vue 中渲染 HTML 很快，不过当组件中包含大量静态内容时，可以考虑使用 `v-once` 将渲染结果缓存起来，就像这样：
```
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ...很多静态内容...\
    </div>\
  '
})
```

### class 和 style 绑定

在数据绑定中，一个常见需求是，将数据与元素的 class 列表，以及元素的 style 内联样式的操作绑定在一起。由于它们都是属性(attribute)，因此我们可以使用 `v-bind` 来处理它们：只需从表达式中计算出最终的字符串。然而，处理字符串拼接，既麻烦又容易出错。为此，在使用 `v-bind` 指令来处理 class 和 style 时，Vue 对此做了特别的增强。表达式除了可以是字符串，也能够是对象和数组。

#### 与 HTML 的 class 绑定(Binding HTML Classes)

##### 对象语法

我们可以向 `v-bind:class` 传入一个对象，来动态地切换 class：
```
<div v-bind:class="{ active: isActive }"></div>
```
上述语法意味着，`active` 这个 class 的存在与否，取决于 `isActive` 这个 `data` 属性的 `truthy` 值。

这样，可以通过在对象中添加多个字段，来切换多个 class。此外，`v-bind:class` 指令也可以和普通 class 属性共存。所以，给定以下模板：
```
<div class="static"
     v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>
```
然后，给定以下 `data`：
```
data: {
  isActive: true,
  hasError: false
}
```
将会渲染为：
```
<div class="static active"></div>
```
每当 `isActive` 或 `hasError` 发生变化，class 列表就会相应地更新。例如，如果 `hasError` 值是 `true`，class 列表会变为 `"static active text-danger"`。

绑定对象，也可以无需内联，而是外部引用 `data`：
```
<div v-bind:class="classObject"></div>

data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```
（内联、外部引用）这两种方式的渲染结果相同。我们还可以将 class 和 style 与某个 `computed` 属性绑定在一起，此 `computed` 属性所对应的 `getter` 函数需要返回一个对象。这是一种常用且强大的用法：
```
<div v-bind:class="classObject"></div>

data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

##### 数组语法

我们可以向 `v-bind:class` 传入一个数组，来与 class 列表对应：
```
<div v-bind:class="[activeClass, errorClass]"></div>

data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```
会被渲染为：
```
<div class="active text-danger"></div>
```
如果你还想根据条件，切换 class 列表中某个 class，可以通过三元表达式(ternary expression)来实现：
```
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```
这里会直接添加 `errorClass`，但是只有在 `isActive` 值是 truthy 时，才会添加 `activeClass`。

然而，如果有多个条件 class 时，就会显得有些繁琐。这也就是为什么还可以在数组语法中使用对象语法：
```
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

##### 在组件中使用

当你在自定义组件中使用 class 属性，这些 class 会被添加到组件的根元素上。根元素上已经存在的 class 不会被覆盖。

例如，如果你这样声明组件：
```
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```
然后，在调用组件时，再添加一些 class：
```
<my-component class="baz boo"></my-component>
```
那么，最终渲染的 HTML 就是：
```
<p class="foo bar baz boo">Hi</p>
```
同样，class 绑定也是如此：
```
<my-component v-bind:class="{ active: isActive }"></my-component>
```
当 `isActive` 值是 truthy，最终渲染的 HTML 就是：
```
<p class="foo bar active">Hi</p>
```

#### 与内联 style 绑定(Binding Inline Styles)

##### 对象语法

`v-bind:style` 的对象语法是非常简单直接的 - 看起来非常像 CSS，其实它是一个 JavaScript 对象。CSS 属性名称可以使用驼峰式(camelCase)或串联式(kebab-case)（使用串联式需要加引号）：
```
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

data: {
  activeColor: 'red',
  fontSize: 30
}
```
通常，一个比较好的做法是，直接与 `data` 中的 style 对象绑定，这样模板看起来更清晰：
```
<div v-bind:style="styleObject"></div>

data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```
再次申明，`v-bind:style` 的对象语法，通常也会和 `computed` 属性结合使用，此 `computed` 属性所对应的 `getter` 函数需要返回一个对象。

##### 数组语法

`v-bind:style` 的数组语法，可以在同一个元素上，使用多个 style 对象：
```
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

##### 自动添加前缀

在 `v-bind:style` 中使用具有浏览器厂商前缀(vendor prefixes)的 CSS 属性时（例如 `transform`），Vue 会自动检测并向 `style` 添加合适的前缀。

主流浏览器引擎前缀:

- `-webkit-` (谷歌, Safari, 新版Opera浏览器等)
- `-moz-` (火狐浏览器)
- `-o-` (旧版Opera浏览器等)
- `-ms-` (IE浏览器 和 Edge浏览器)

##### 多个值

从 2.3.0+ 起，你可以为每个 style 属性提供一个含有多个（前缀）值的数组，例如：
```
<div v-bind:style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```
最终，这只会从数组中找出「最后一个浏览器所支持的值」进行渲染。在这个示例中，对于支持「无需前缀版本的 flexbox」的浏览器，最终将渲染为 `display: flex`。


### 表单 input 绑定

#### 基础用法

可以通过使用 `v-model` 指令，在表单 input 和 textarea 元素上创建双向数据绑定。`v-model` 指令可以根据 input 的 type 类型，自动地以正确的方式更新元素。虽然略显神奇，然而本质上 `v-model` 不过是「通过监听用户的 input 事件来更新数据」的语法糖，以及对一些边界情况做特殊处理。

> `v-model` 会忽略所有表单元素中 `value`, `checked` 或 `selected` 属性上初始设置的值，而总是将 Vue 实例中的 `data` 作为真实数据来源。因此你应该在 JavaScript 端的组件 `data` 选项中声明这些初始值，而不是 HTML 端。

> 对于需要使用输入法的语言（中文、日文、韩文等），你会发现，在输入法字母组合窗口输入时，`v-model` 并不会触发数据更新。如果你想在此输入过程中，满足更新数据的需求，请使用 `input` 事件。

##### 单行文本(text)
```
<input v-model="message" placeholder="编辑">
<p>message 是：{{ message }}</p>
```

##### 多行文本(multiple text)
```
<span>多行 message 是：</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="添加多行"></textarea>
```
> 在 textarea 中插值（`<textarea>{{text}}</textarea>`）并不会生效。使用` v-model` 来替代。

##### checkbox

单选 `checkbox`，绑定到布尔值：
```
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```

多选 checkbox，绑定到同一个数组：
```
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>勾选的名字是：{{ checkedNames }}</span>
</div>

new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```

##### radio
```
<input type="radio" id="one" value="One" v-model="picked">
<label for="one">One</label>
<br>
<input type="radio" id="two" value="Two" v-model="picked">
<label for="two">Two</label>
<br>
<span>选中的是：{{ picked }}</span>
```

##### select

单选 select：
```
<select v-model="selected">
  <option disabled value="">请选择其中一项</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>选中的是：{{ selected }}</span>

new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```

> 如果 `v-model` 表达式的初始值，不能与任何 option 选项匹配，  `<select>` 元素将会渲染为“未选中”状态。在 iOS 中，这会导致用户无法选中第一项，因为在这种“未选中”状态的情况下，iOS 不会触发 change 事件。因此，推荐按照上面的示例，预先提供一个 value 为空字符串的禁用状态的 option 选项。

多选 select（绑定到一个数组）：
```
<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>选中的是：{{ selected }}</span>
```

使用 `v-for` 渲染动态的 option：
```
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>选中的是：{{ selected }}</span>

new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```

#### 与 value 属性绑定

对于 radio, checkbox 和 select 的 option 选项，通常可以将 `v-model` 与值是静态字符串的 value 属性关联在一起（或者，在 checkbox 中，绑定到布尔值）：
```
<!-- 当选中时，`picked` 的值是字符串 "a"（译者注：如果没有 value 属性，选中时值是 null） -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` 的值是 true 或 false -->
<input type="checkbox" v-model="toggle">

<!-- 当选中第一个选项时，`selected` 的值是字符串 "abc"（译者注：如果没有 value 属性，选中时 selected 值是 option 元素内的文本） -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```
然而，有时可能需要把 value 与 Vue 实例上的一个动态属性绑定在一起。这时我们可以通过 `v-bind` 来实现。`v-bind` 还允许我们将 input 元素的值绑定到非字符串值。

##### checkbox
```
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>

// 选中时：
vm.toggle === 'yes'
// 没有选中时：
vm.toggle === 'no'
```
> `true-value` 和 `false-value` 属性不会影响到 input 元素的 `value` 属性，这是因为浏览器在提交表单时，并不会包含未被选中的 checkbox 的值。如果要确保表单中，这两个值的一个能够被提交（例如 “yes” 或 “no”），请换用类型是 radio 的 input 元素。

##### radio

```
<input type="radio" v-model="pick" v-bind:value="a">

// 选中时
vm.pick === vm.a
```

##### select 选项

```
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>

// 选中时：
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

#### 修饰符(modifiers)

##### `.lazy`

默认情况下， `v-model` 会在每次 `input` 事件触发之后，将数据同步至 input 元素中（除了上述提到的输入法组合文字时不会）。可以添加 `lazy` 修饰符，从而转为在触发 `change` 事件后同步：
```
<!-- 在触发 "change" 事件后同步，而不是在触发 "input" 事件后更新 -->
<input v-model.lazy="msg" >
```

##### `.number`

如果想要将用户的输入，自动转换为 Number 类型（译注：如果转换结果为 NaN 则返回字符串类型的输入值），可以在 `v-model` 之后添加一个 `number` 修饰符，来处理输入值：
```
<input v-model.number="age" type="number">
```
这通常很有用，因为即使是在 `type="number"` 时，HTML 中 `input` 元素也总是返回一个字符串类型的值。

##### ` .trim`

如果想要将用户的输入，自动过滤掉首尾空格，可以在 `v-model` 之后添加一个 `trim` 修饰符，来处理输入值：
```
<input v-model.trim="msg">
```

#### 在组件上使用 `v-model`

HTML 内置的几种 input 类型有时并不总能满足需求。幸运的是，使用 Vue 可以创建出可复用的输入框组件，并且能够完全自定义组件的行为。这些输入框组件甚至可以使用 `v-model`！想要了解更多信息，请阅读组件指南中自定义输入框)。


### 事件处理

#### 监听事件

可以用 `v-on` 指令监听 DOM 事件，并在触发时运行一些 JavaScript 代码。
```
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>

var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```

#### 事件处理方法

然而许多事件处理逻辑会更为复杂，所以直接把 JavaScript 代码写在 `v-on `指令中是不可行的。因此 `v-on` 还可以接收一个需要调用的方法名称。
```
<div id="example-2">
  <!-- `greet` 是在下面定义的方法名 -->
  <button v-on:click="greet">Greet</button>
</div>

var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (event) {
      // `this` 在方法里指向当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// 也可以用 JavaScript 直接调用方法
example2.greet() // => 'Hello Vue.js!'
```

#### 内联处理器中的方法

除了直接绑定到一个方法，也可以在内联 JavaScript 语句中调用方法：
```
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>

new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
```
有时也需要在内联语句处理器中访问原始的 DOM 事件。可以用特殊变量 `$event` 把它传入方法：
```
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>

// ...
methods: {
  warn: function (message, event) {
    // 现在我们可以访问原生事件对象
    if (event) event.preventDefault()
    alert(message)
  }
}
```

#### 事件修饰符

在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

辑，而不是去处理 DOM 事件细节。

为了解决这个问题，Vue.js 为 `v-on` 提供了事件修饰符。之前提过，修饰符是由点开头的指令后缀来表示的。

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`

```
<!-- 阻止单击事件继续传播 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div v-on:click.self="doThat">...</div>
```
使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止所有的点击，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。

> 2.1.4 新增
```
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
```
不像其它只能对原生的 DOM 事件起作用的修饰符，`.once` 修饰符还能被用到自定义的组件事件上。

> 2.3.0 新增

Vue 还对应 `addEventListener` 中的 `passive` 选项提供了 `.passive` 修饰符。
```
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
<!-- 而不会等待 `onScroll` 完成  -->
<!-- 这其中包含 `event.preventDefault()` 的情况 -->
<div v-on:scroll.passive="onScroll">...</div>
```
这个 `.passive` 修饰符尤其能够提升移动端的性能。

> 不要把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，`.passive` 会告诉浏览器你不想阻止事件的默认行为。

#### 按键修饰符

在监听键盘事件时，我们经常需要检查常见的键值。Vue 允许为 `v-on` 在监听键盘事件时添加按键修饰符：
```
<!-- 只有在 `keyCode` 是 13 时调用 `vm.submit()` -->
<input v-on:keyup.13="submit">
```

记住所有的 `keyCode` 比较困难，所以 Vue 为最常用的按键提供了别名：
```
<!-- 同上 -->
<input v-on:keyup.enter="submit">

<!-- 缩写语法 -->
<input @keyup.enter="submit">
```

全部的按键别名：

- `.enter`
- `.tab`
- `.delete` (捕获“删除”和“退格”键)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

可以通过全局 `config.keyCodes` 对象自定义按键修饰符别名：
```
// 可以使用 `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

##### keyCodes

类型：{ [key: string]: number | Array<number> }

默认值：{}

用法：
```
<input type="text" @keyup.media-play-pause="method">

Vue.config.keyCodes = { // 添加多个自定义按键修饰符
  v: 86,
  f1: 112,
  mediaPlayPause: 179,  // camelCase 这种写法不可用
  "media-play-pause": 179,   // 取而代之的是 kebab-case 且用双引号括起来
  up: [38, 87]
}
```
给 `v-on` 自定义键位别名。

##### 自动匹配按键修饰符

> 2.5.0 新增

你也可直接将 `KeyboardEvent.key` 暴露的任意有效按键名转换为 `kebab-case` 来作为修饰符：
```
<input @keyup.page-down="onPageDown">
```
在上面的例子中，处理函数仅在 `$event.key === 'PageDown'` 时被调用。

有一些按键 (`.esc` 以及所有的方向键) 在 IE9 中有不同的 `key` 值, 如果你想支持 IE9，它们的内置别名应该是首选。

##### 系统修饰键

> 2.1.0 新增

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

> 注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。

例如：
```
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

> 请注意修饰键与常规按键不同，在和 `keyup` 事件一起用时，事件触发时修饰键必须处于按下状态。换句话说，只有在按住 `ctrl` 的情况下释放其它按键，才能触发 `keyup.ctrl`。而单单释放 `ctrl` 也不会触发事件。如果你想要这样的行为，请为 `ctrl` 换用 `keyCode：keyup.17`。

##### `.exact` 修饰符

> 2.5.0 新增

`.exact` 修饰符允许你控制由精确的系统修饰符组合触发的事件

```
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

##### 鼠标按钮修饰符

> 2.2.0 新增

- `.left`
- `.right`
- `.middle`

这些修饰符会限制处理函数仅响应特定的鼠标按钮。

#### 为什么在 HTML 中监听事件?

你可能注意到这种事件监听的方式违背了关注点分离 (separation of concern) 这个长期以来的优良传统。但不必担心，因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。实际上，使用 `v-on` 有几个好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。

2. 因为你无须在 JavaScript 里手动绑定事件，你的 ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试。

3. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何自己清理它们。



### 条件渲染

#### 条件判断 `v-if`
```
<h1 v-if="status == 1">在线</h1>
<h1 v-else-if="status == 2">离线</h1>
<h1 v-else>告警</h1>
```
`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。

> `v-else-if` 是 2.1.0+ 新增

#### 条件分组使用 `<template>` 标签包裹

因为 v-if 是一个指令，所以必须将它添加到一个元素上。但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 `v-if`。最终的渲染结果将不包含 `<template> ` 元素。

```
<template v-if="ok">
  <h1>标题</h1>
  <p>段落 1</p>
  <p>段落 2</p>
</template>
```
注意，`v-show` 和 `v-else` 都不支持 `<template>` 元素。（测试发现  `v-else` 可以使用 `<template>`，`v-show` 不可以）

#### 使用 `key` 控制元素是否可复用

Vue 会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。这么做除了使 Vue 变得非常快之外，还有其它一些好处。例如，如果你允许用户在不同的登录方式之间切换：
```
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```

那么在上面的代码中切换 `loginType` 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，`<input>` 不会被替换掉——仅仅是替换了它的 `placeholder`。

但是这样有时并不符合实际需求，所以 Vue 为如下所述的情况提供了一种方式：“这两个元素是完全独立的 - 请不要复用它们”。那就是为它们添加一个具有不同值的 `key` 属性：

```
<template v-if="loginType === 'username'">
  <label>用户名</label>
  <input placeholder="请输入用户名" key="username-input">
</template>
<template v-else>
  <label>邮箱</label>
  <input placeholder="请输入邮箱" key="email-input">
</template>
```
注意，`<label>` 元素仍然被有效地复用，因为它们没有 `key` 属性。


####  v-show指令

另一个用于根据条件展示元素的选项是 `v-show` 指令。用法大致一样：
```
<h1 v-show="ok">Hello!</h1>
```
不同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS 属性 `display`。

注意，`v-show` 和 `v-else` 都不支持 `<template>` 元素。（测试发现  `v-else` 可以使用 `<template>`，`v-show` 不可以）

#### `v-if` 和 `v-show`

`v-if` 是“真实”的条件渲染，因为它会确保条件块(conditional block)在切换的过程中，完整地销毁(destroy)和重新创建(re-create)条件块内的事件监听器和子组件。

`v-if` 是惰性的(lazy)：如果在初始渲染时条件为 false，它不会执行任何操作 - 在条件第一次变为 true 时，才开始渲染条件块。

相比之下，`v-show` 要简单得多 - 不管初始条件如何，元素始终渲染，并且只是基于 CSS 的切换。

通常来说，`v-if` 在切换时有更高的性能开销，而 `v-show` 在初始渲染时有更高的性能开销。因此，如果需要频繁切换，推荐使用 `v-show`，如果条件在运行时改变的可能性较少，推荐使用 `v-if`。

#### `v-if` 与 `v-for` 一起使用

当 `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级。




### 列表渲染

#### 使用 v-for 遍历数组生成元素

我们可以使用 `v-for` 指令，将一个数组渲染为列表项。`v-for` 指令需要限定格式为 `item in items` 的特殊语法，其中，`items` 是原始数据数组(source data array)，而 `item` 是数组中每个迭代元素的指代别名(alias)：
```
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

```
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

在 `v-for` 代码块中，我们可以完全地访问父级作用域下的属性。`v-for` 还支持可选的第二个参数，作为当前项的索引。

```
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>

var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```
结果

 - Parent - 0 - Foo
 - Parent - 1- Bar

你还可以不使用 `in`，而是使用 `of` 作为分隔符，因为它更加接近 JavaScript 迭代器语法：

```
<div v-for="item of items"></div>
```

#### 使用 v-for 遍历对象

可以提供第二个参数，作为对象的键名(key)，第三个参数作为索引(index)

```
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</div>

new Vue({
  el: '#v-for-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
```
在遍历一个对象时，是按照 `Object.keys()` 得出 `key` 的枚举顺序来遍历，无法保证在所有 JavaScript 引擎实现中完全一致。

#### key

为了便于 Vue 跟踪每个节点的身份，从而重新复用(reuse)和重新排序(reorder)现有元素，你需要为每项提供唯一的 `key` 属性，从而给 Vue 一个提示。理想的 `key` 值是每项都有唯一的 id。你需要使用 `v-bind` 将其与动态值绑定在一起：
```
<div v-for="item in items" :key="item.id">
  <!-- content -->
</div>
```
推荐，在使用 ` v-for` 时，尽可能提供一个 `key`，除非迭代的 DOM 内容足够简单，或者你是故意依赖于默认行为来获得性能提升。

由于这是 Vue 识别节点的通用机制，因此 `key` 并不是仅限于与 `v-for` 关联，我们将在之后的指南中看到，`key` 还可以其他场景使用。

#### 数组变化检测(Array Change Detection)

##### 变化数组方法(Mutation Methods)

Vue 将观察数组(observed array)的变化数组方法(mutation method)包裹起来，以便在调用这些方法时，也能够触发视图更新。这些包裹的方法如下：

 - `push()`
 - `pop()`
 - `shift()`
 - `unshift()`
 - `splice()`
 - `sort()`
 - `reverse()`

可以打开控制台，然后对前面示例中的 `items` 数组调用变化数组方法。例如：`example1.items.push({ message: 'Baz' })`。

##### 替换一个数组(Replacing an Array)

变化数组方法(mutation method)，顾名思义，在调用后会改变原始数组。相比之下，还有非变化数组方法(non-mutating method)，例如 `filter()`, `concat()` 和 `slice()`，这些方法都不会直接修改操作原始数组，而是返回一个新数组。当使用非变化数组方法时，可以将旧数组替换为新数组：

```
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```
你可能会认为这将导致 Vue 丢弃现有 DOM 并重新渲染(re-render)整个列表 - 幸运的是，情况并非如此。Vue 实现了一些智能启发式方法(smart heuristic)来最大化 DOM 元素重用(reuse)，因此将一个数组替换为包含重叠对象的另一个数组，会是一种非常高效的操作。

#####  注意事项(Caveats)

由于 JavaScript 的限制，Vue 无法检测到以下数组变动：

 1. 当你使用索引直接设置一项时，例如 `vm.items[indexOfItem] = newValue` 
 2. 当你修改数组长度时，例如 `vm.items.length = newLength`

例如：

```
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应的
vm.items.length = 2 // 不是响应的
```
为了解决第 1 个问题，以下两种方式都可以实现与 `vm.items[indexOfItem] = newValue` 相同的效果，但是却可以通过响应式系统出发状态更新：

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)

// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```
你还可以使用 vm.$set 实例方法，这也是全局 Vue.set 方法的别名：
```
vm.$set(vm.items, indexOfItem, newValue)
```

为了解决第 2 个问题，你可以使用 `splice`：

```
vm.items.splice(newLength)
```

##### 对象变化检测说明(Object Change Detection Caveats)

再次申明，受现代 Javascript 的限制， Vue 无法检测到对象属性的添加或删除。例如：

```
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 是响应的

vm.b = 2
// `vm.b` 不是响应的
```
Vue 不允许在已经创建的实例上，动态地添加新的根级响应式属性(root-level reactive property)。然而，可以使用 `Vue.set(object, key, value)` 方法，将响应式属性添加到嵌套的对象上。例如，给出：

```
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
```
可以向嵌套的 `userProfile` 对象，添加一个新的 `age` 属性：

```
Vue.set(vm.userProfile, 'age', 27)
```

还可以使用 `vm.$set` 实例方法，这也是全局 `Vue.set` 方法的别名：

```
vm.$set(vm.userProfile, 'age', 27)
```


有时，你想要向已经存在的对象上添加一些新的属性，例如使用 `Object.assign()` 或 `_.extend()` 方法。在这种情况下，应该创建一个新的对象，这个对象同时具有两个对象的所有属性，因此，改为：

```
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
可以通过如下方式，添加新的响应式属性：

```
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```
##### 显示过滤/排序结果(Displaying Filtered/Sorted Results)

有时，我们想要显示一个数组过滤或排序后(filtered or sorted)的副本，而不是实际改变或重置原始数据。在这种情况下，可以创建一个返回过滤或排序数组的计算属性。

例如：
```
<li v-for="n in evenNumbers">{{ n }}</li>


data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```
在计算属性不适用的情况下（例如，在嵌套的 `v-for` 循环内），可以使用一个 `method` 方法：
```
<li v-for="n in even(numbers)">{{ n }}</li>

data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
methods: {
  even: function (numbers) {
    return numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```

##### 使用 `v-for` 在整数值范围内迭代

`v-for` 也可以在整数值范围内迭代。在这种情况下，会将模板重复多次。
```
<div>
  <span v-for="n in 10">{{ n }}</span>
</div>
```
结果：
```
 1 2 3 4 5 6 7 8 9 10
```

##### 在 `<template>` 上使用 `v-for`

类似于在 `<template>` 上使用 `v-if`，你还可以在 `<template>` 标签上使用 `v-for`，来渲染多个元素块。例如：
```
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider"></li>
  </template>
</ul>
```

##### 带有 `v-if` 的 `v-for`

当它们都处于同一节点时，`v-for` 的优先级高于 `v-if`。这意味着，`v-if` 将分别在循环中的每次迭代上运行。当你只想将某些项渲染为节点时，这会非常有用，如下：
```
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```
以上只渲染 todos 中未完成的项。

如果你的意图与此相反，是根据条件跳过执行循环，可以将 `v-if` 放置于包裹元素上（或放置于 `<template>` 上）。例如：
```
<ul v-if="todos.length">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
<p v-else>No todos left!</p>
```

##### 使用 `v-for` 遍历组件

在自定义组件上，你可以直接使用 `v-for`，就像其他普通元素：
```
<my-component v-for="item in items" :key="item.id"></my-component>
```
> 现在，在 2.2.0+ 版本，当对组件使用 v-for 时，必须设置 key 属性。

然而，这里无法自动向组件中传入数据，这是因为组件有自己的独立作用域。为了将组件外部的迭代数据传入组件，我们还需要额外使用 `props`：
```
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index"
  v-bind:key="item.id"
></my-component>
```

没有通过 `v-for` 将 `item` 自动注入到组件中的原因是，一旦自动注入，就会使得组件与 `v-for` 指令的运行方式紧密耦合(tightly coupled)在一起。通过显式声明组件数据来源，可以使得组件可重用于其他场景。

这里是一个 todo list 的完整示例：
```
<div id="todo-list-example">
  <input
    v-model="newTodoText"
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:key="todo.id"
      v-bind:title="todo.title"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```
注意 `is="todo-item" ` 属性。这在 DOM 模板中是必需的，因为在 `<ul>` 中，只有 `<li>` 是有效元素。这与调用 `<todo-item>` 的实际结果相同，但是却可以解决浏览器潜在的解析错误。了解更多信息，请查看 DOM 模板解析注意事项。

像 `<ul>, <ol>, <table>` 和 `<select>` 这样的元素，限制了出现在其中的元素，而像 `<option>` 这样的元素，只能出现在相应的元素中。
  
```
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    </li>\
  ',
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      {
        id: 1,
        title: 'Do the dishes',
      },
      {
        id: 2,
        title: 'Take out the trash',
      },
      {
        id: 3,
        title: 'Mow the lawn'
      }
    ],
    nextTodoId: 4
  },
  methods: {
    addNewTodo: function () {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})
```

##### 备注

- 变化数组方法：会对原数组操作的数组方法，如 `pop()`, `shift()`, `unshift()`, ` splice()`, `sort()` 和 `reverse()`。
- 非变化数组方法：不会对原数组操作、返回新数组的数组方法，如 `filter()`, `concat()` 和 `slice()`。
- 变异方法：Vue 将观察数组(observed array)的变化数组方法(mutation method)包裹起来，以便在调用这些方法时，也能够触发视图更新。



### computed 属性和 watcher

#### computed 属性

在模板中使用表达式是非常方便直接的，然而这只适用于简单的操作。在模板中放入太多的逻辑，会使模板过度膨胀和难以维护。例如：
```
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```
在这个地方，模板不再简洁和如声明式直观。你必须仔细观察一段时间才能意识到，这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中多次引用此处的翻转字符串时，就会更加难以处理。

这就是为什么对于所有复杂逻辑，你都应该使用 `computed` 属性(computed property)。

##### 基础示例
```
<div id="example">
  <p>初始 message 是："{{ message }}"</p>
  <p>计算后的翻转 message 是："{{ reversedMessage }}"</p>
</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 一个 computed 属性的 getter 函数
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

这里我们声明了一个 `computed` 属性 `reversedMessage`。然后为 `vm.reversedMessage` 属性提供一个函数，作为它的 `getter` 函数：
```
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```
你可以打开浏览器的控制台，然后如示例中操作 vm。会发现 `vm.reversedMessage` 的值总是依赖于 `vm.message` 的值。

你可以像绑定普通属性一样，将 `computed` 属性的数据，绑定(data-bind)到模板中的表达式上。Vue 能够意识到 `vm.reversedMessage` 依赖于 `vm.message`，也会在 `vm.message` 修改后，更新所有依赖于 `vm.reversedMessage` 的数据绑定。最恰到好处的部分是，我们是通过声明式来创建这种依赖关系：`computed` 属性的 `getter` 函数并无副作用(side effect)，因此也更加易于测试和理解。

##### computed 缓存 vs method 方法

你可能已经注意到，我们可以在表达式中通过调用 `method` 方法的方式，也能够实现与 `computed` 属性相同的结果：
```
<p>翻转 message 是："{{ reverseMessage() }}"</p>

// 在组件中
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```
这里不使用 `computed` 属性，而是在 `methods` 中定义一个相同的函数。对于最终结果，这两种方式确实恰好相同。

然而，细微的差异之处在于，`computed` 属性会基于它所依赖的数据进行缓存。

每个 `computed` 属性，只有在它所依赖的数据发生变化时，才会重新取值(re-evaluate)。这就意味着，只要 `message` 没有发生变化，多次访问 `computed` 属性 `reversedMessage`，将会立刻返回之前计算过的结果，而不必每次都重新执行函数。

这也同样意味着，如下的 `computed` 属性永远不会更新，因为 `Date.now()` 不是一个响应式的依赖数据：
```
computed: {
  now: function () {
    return Date.now()
  }
}
```

相比之下，每当触发重新渲染(re-render)时，`method` 调用方式将总是再次执行函数。

为什么我们需要将依赖数据缓存起来？假设一种场景，我们有一个高性能开销(expensive)的 `computed` 属性 A，在 `computed` 属性的 `getter` 函数内部，需要遍历循环一个巨大数组，并进行大量计算。然后还有其他 `computed` 属性直接或间接依赖于 A。如果没有缓存，我们将不可避免地多次执行 A 的 `getter` 函数，这远多余实际需要执行的次数！然而在某些场景下，你可能不希望有缓存，请使用 `method` 方法替代。


##### `computed` 属性和 `watch` 属性

Vue 其实还提供了一种更加通用的方式，来观察和响应 Vue 实例上的数据变化：`watch` 属性。

`watch` 属性是很吸引人的使用方式，然而，当你有一些数据需要随着另外一些数据变化时，过度滥用 `watch` 属性会造成一些问题 - 尤其是那些具有 AngularJS 开发背景的开发人员。

因此，更推荐的方式是，使用 `computed` 属性，而不是命令式(imperative)的 `watch` 回调函数。思考下面这个示例：
```
<div id="demo">{{ fullName }}</div>

var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```
以上代码是命令式和重复的。对比 computed 属性实现的版本：
```
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```
这样显得更优雅，对吗？

##### `computed` 属性中设置 `setter`

`computed` 属性默认只设置 `getter` 函数，不过在需要时，还可以提供 `setter` 函数：
```

computed: {
  fullName: {
    // getter 函数
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter 函数
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}

```
现在当你运行 `vm.fullName = 'John Doe'`，将会调用 `setter`，然后会对应更新 `vm.firstName` 和 `vm.lastName`。

#### watcher

虽然在大多数情况下，更适合使用 `computed` 属性，然而有些时候，还是需要一个自定义 `watcher`。这就是为什么 Vue 要通过 `watch` 选项，来提供一个更加通用的响应数据变化的方式。

当你需要在数据变化响应时，执行异步操作，或高性能消耗的操作，自定义 `watcher` 的方式就会很有帮助。

例如：
```
<div id="watch-example">
  <p>
    问一个答案是 yes/no 的问题：
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>


<!-- 对于 ajax 库(ajax libraries)和通用工具方法的集合(collections of general-purpose utility methods)来说， -->
<!-- 由于已经存在大量与其相关的生态系统， -->
<!-- 因此 Vue 核心无需重新创造，以保持轻量的文件体积。 -->
<!-- 同时这也可以使你自由随意地选择自己最熟悉的。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: '你要先提出问题，我才能给你答案！'
  },
  watch: {
    // 只要 question 发生改变，此函数就会执行
    question: function (newQuestion, oldQuestion) {
      this.answer = '等待输入停止……'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce 是由 lodash 提供的函数，
    // 在运行特别消耗性能的操作时，
    // 可以使用 _.debounce 来限制频率。
    // 在下面这种场景中，我们需要限制访问 yesno.wtf/api 的频率，
    // 等到用户输入完毕之后，ajax 请求才会发出。
    // 想要了解更多关于 _.debounce 函数（以及与它类似的 _.throttle）的详细信息，
    // 请访问：https://lodash.com/docs#debounce
    getAnswer: _.debounce(
      function () {
        if (this.question.indexOf('？') === -1) {
          this.answer = '问题通常需要包含一个中文问号。;-)'
          return
        }
        this.answer = '思考中……'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = '错误！API 无法处理。' + error
          })
      },
      // 这是用户停止输入操作后所等待的毫秒数。
      // （译者注：500毫秒之内，用户继续输入，则重新计时）
      500
    )
  }
})
</script>
```
在这个场景中，使用 `watch` 选项，可以使我们执行一个限制执行频率的（访问一个 API 的）异步操作，并且不断地设置中间状态，直到我们获取到最终的 `answer` 数据之后，才真正执行异步操作。而 `computed` 属性无法实现。

除了 `watch` 选项之外，还可以使用命令式(imperative)的 `vm.$watch ` API。




## 过渡 & 动画

### 进入、离开和列表的过渡

#### 概述

当从 DOM 中插入、更新或移除项目时，Vue 提供多种应用过渡效果的方式。包括以下工具：

- 在 CSS 过渡和动画中自动处理 class
- 可以配合使用第三方 CSS 动画库，如 Animate.css
- 在过渡钩子函数中使用 JavaScript 直接操作 DOM
- 可以配合使用第三方 JavaScript 动画库，如 Velocity.js

本页面中，我们只会涉及到进入、离开和列表的过渡，然而你也可以查看下一章节管理过渡状态.

#### 单元素/组件的过渡

Vue 提供了 `transition` 外层包裹容器组件(wrapper component)，可以给下列情形中的任何元素和组件添加进入/离开(enter/leave)过渡

- 条件渲染（使用 `v-if`）
- 条件展示（使用 `v-show`）
- 动态组件
- 组件根节点

这是一个常见行为的简单示例：
```
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active 在低于版本 2.1.8 中 */ {
  opacity: 0;
}

<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>

new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```
当插入或删除包含在 `transition` 组件中的元素时，Vue 将会做以下处理：

1. 自动嗅探目标元素是否使用了 CSS 过渡或动画，如果使用，会在合适的时机添加/移除 CSS 过渡 `class`。

2. 如果过渡组件设置了 JavaScript 钩子函数，这些钩子函数将在合适的时机调用。

3. 如果没有检测到 CSS 过渡/动画，并且也没有设置 JavaScript 钩子函数，插入和/或删除 DOM 的操作会在下一帧中立即执行。（注意：这里的帧是指浏览器逐帧动画机制，和 Vue 的 `nextTick` 概念不同）

##### 过渡类名(Transition Classes)

有 6 种 `class` 类名会在进入/离开(enter/leave)过渡中处理

1. `v-enter`：进入式过渡(entering transition)的开始状态。在插入元素之前添加，在插入元素之后一帧移除。

2. `v-enter-active`：进入式过渡的激活状态。应用于整个进入式过渡时期。在插入元素之前添加，过渡/动画(transition/animation)完成之后移除。此 class 可用于定义进入式过渡的 duration, delay 和 easing 曲线。

3. `v-enter-to`：仅适用于版本 2.1.8+。进入式过渡的结束状态。在插入元素之后一帧添加（同时，移除 `v-enter`），在过渡/动画完成之后移除。

4. `v-leave`：离开式过渡(leaving transition)的开始状态。在触发离开式过渡时立即添加，在一帧之后移除。

5. `v-leave-active`：离开式过渡的激活状态。应用于整个离开式过渡时期。在触发离开式过渡时立即添加，在过渡/动画(transition/animation)完成之后移除。此 class 可用于定义离开式过渡的 duration, delay 和 easing 曲线。

6. `v-leave-to`：仅适用于版本 2.1.8+。离开式过渡的结束状态。在触发离开式过渡之后一帧添加（同时，移除 `v-leave`），在过渡/动画完成之后移除。

![过渡](https://vuefe.cn/images/transition.png)

对于这些过渡中切换 class，每个都以过渡的 name 作为前缀。当你使用没有 name 的 `<transition>` 元素时，会默认前缀为 `v-`。

举个例子，如果你使用 `<transition name="my-transition">`，那么默认的 `v-enter` class 将会被替换为 `my-transition-enter`。

`v-enter-active` 和 `v-leave-active` 可以指定不同的进入/离开过渡 easing 曲线，下面章节可以看到一个示例。

##### CSS 过渡(CSS Transitions)

最常用到的过渡类型是使用 CSS 过渡。下面是一个示例：
```
/* 进入和离开动画可以分别 */
/* 设置不同的持续时间(duration)和动画函数(timing function) */
.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active 在低于 2.1.8 版本中 */ {
  transform: translateX(10px);
  opacity: 0;
}

<div id="example-1">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition name="slide-fade">
    <p v-if="show">hello</p>
  </transition>
</div>

new Vue({
  el: '#example-1',
  data: {
    show: true
  }
})
```

##### CSS 动画(CSS Animations)

CSS 动画用法和 CSS 过渡相同，区别是在动画中 `v-enter` 类名在元素插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除。

这里是一个示例，为了简洁省略了 CSS 规则的前缀：
```
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}

<div id="example-2">
  <button @click="show = !show">Toggle show</button>
  <transition name="bounce">
    <p v-if="show">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris facilisis enim libero, at lacinia diam fermentum id. Pellentesque habitant morbi tristique senectus et netus.</p>
  </transition>
</div>

new Vue({
  el: '#example-2',
  data: {
    show: true
  }
})
```

##### 自定义过渡的 class 类名(Custom Transition Classes)

你也可以通过提供一下属性来指定自定义过渡类名

- `enter-class`
- `enter-active-class`
- `enter-to-class (2.1.8+)`
- `leave-class`
- `leave-active-class`
- `leave-to-class (2.1.8+)`

它们将覆盖默认约定的类名，这对于将 Vue 的过渡系统和其他现有的第三方 CSS 动画库（如 Animate.css）集成使用会非常有用。

这里是一个示例：
```
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">hello</p>
  </transition>
</div>

new Vue({
  el: '#example-3',
  data: {
    show: true
  }
})
```

##### 同时使用过渡和动画(Using Transitions and Animations Together)

Vue 为了知道过渡何时完成，必须附加相应的事件监听器。它可以是 `transitionend` 或 `animationend`，这取决于给元素应用的 CSS 规则。如果你使用其中任何一种，Vue 能自动识别正确的类型并设置相应的事件监听器。

但是，在一些情况下，你可能需要给同一个元素同时设置过渡和动画，比如由 Vue 触发 CSS 动画，同时在鼠标悬停时触发 CSS 过渡。在这种情况下，你可能需要通过 type 属性，来显式声明需要 Vue 监听的类型，值可以是 `animation` 或 `transition`。


##### 显式过渡持续时间(Explicit Transition Durations)

> 2.2.0+ 新增

在大多数情况下，Vue 可以自动推断出过渡完成时间。默认情况下，Vue 会过渡根元素的第一个 `transitionend` 或 `animationend` 事件触发所需的等待时间。然而，这可能并不总是我们想要的

 - 例如，我们可能具有设计安排的过渡序列(transition sequence)：其中一些嵌套的内部元素（在根元素过渡完成后）还具有延续的过渡效果，或比过渡根元素更长的过渡持续时间。

在这种情况下，你可以使用 `<transition>` 组件上的 `duration` 属性 ，来指定一个显式的过渡持续时间（以毫秒为单位）：
```
<transition :duration="1000">...</transition>
```
你还可以为进入式和离开式持续时间指定不同的值：
```
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

##### JavaScript 钩子函数

可以在属性中声明 JavaScript 钩子
```
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>


// ...
methods: {
  // --------
  // 进入时
  // --------

  beforeEnter: function (el) {
    // ...
  },
  // 在与 CSS 结合使用时
  // 此回调函数 done 是可选项
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },

  // --------
  // 离开时
  // --------

  beforeLeave: function (el) {
    // ...
  },
  // 在与 CSS 结合使用时
  // 此回调函数 done 是可选项
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled 只能配合 v-show 使用
  leaveCancelled: function (el) {
    // ...
  }
}
```
这些钩子函数可以结合 CSS 过渡/动画使用，也可以单独使用。

注意，当仅使用 JavaScript 式过渡的时候， 在 `enter` 和 `leave` 钩子函数中，必须有 `done` 回调函数。否则，这两个钩子函数会被同步调用，过渡会立即完成。

推荐对于仅使用 JavaScript 的过渡显式添加 `v-bind:css="false"`，以便 Vue 可以跳过 CSS 侦测。这也可以防止 CSS 规则意外干涉到过渡。


现在我们深入来看一个示例。这里是一个使用 `Velocity.js` 的 JavaScript 式过渡：
```
<!--
Velocity works very much like jQuery.animate and is
a great option for JavaScript animations
-->
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="example-4">
  <button @click="show = !show">
    Toggle
  </button>
  <transition
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
    v-bind:css="false"
  >
    <p v-if="show">
      Demo
    </p>
  </transition>
</div>

new Vue({
  el: '#example-4',
  data: {
    show: false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      Velocity(el, { opacity: 1, fontSize: '1.4em' }, { duration: 300 })
      Velocity(el, { fontSize: '1em' }, { complete: done })
    },
    leave: function (el, done) {
      Velocity(el, { translateX: '15px', rotateZ: '50deg' }, { duration: 600 })
      Velocity(el, { rotateZ: '100deg' }, { loop: 2 })
      Velocity(el, {
        rotateZ: '45deg',
        translateY: '30px',
        translateX: '30px',
        opacity: 0
      }, { complete: done })
    }
  }
})
```

#### 在初始渲染时过渡

如果你还想在节点初始渲染时应用过渡，可以添加 `appear` 属性：
```
<transition appear>
  <!-- ... -->
</transition>
```
默认情况下，对于进入和离开，会使用特定过渡。但是，如果你有需要，也可以指定自定义 CSS 类名：
```
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class"（仅 >= 2.1.8 支持）
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```
以及指定自定义 JavaScript 钩子函数：
```
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```

#### 多个元素之间切换过渡

我们将在下面讨论多个元素之间切换过渡，但是还是可以使用 `v-if/v-else`，来对初始元素之间进行切换过渡。最常见的是，一个列表容器和描述列表为空的消息，这两个元素间的切换过渡：
```
<transition>
  <table v-if="items.length > 0">
    <!-- ... -->
  </table>
  <p v-else>Sorry, no items found.</p>
</transition>
```
可以这样使用，但是有一点事项需要注意：
当在具有相同标签名称的元素之间切换时，需要通过给它们分配唯一的 key 属性，以使 Vue 感知它们是不同的元素。否则 Vue 的编译器将因为效率，只会替换元素内部的内容。

即使在技术上没有必要，但是，给 `<transition>` 组件中的多个元素设置 `key`，被认为是一个最佳实践。

示例:
```
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```
在上面这种场景中，也通过给同一元素的 `key` 属性，设置不同的状态来进行过渡。而无需使用 `v-if` 和 `v-else`，所以上面的示例可以重写为：
```
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```
实际上，使用 `v-if` 的多个元素之间的过渡，还可以改为在单个元素上绑定动态属性的方式，来在任意数量的元素之间进行转换。例如：
```
<transition>
  <button v-if="docState === 'saved'" key="saved">
    Edit
  </button>
  <button v-if="docState === 'edited'" key="edited">
    Save
  </button>
  <button v-if="docState === 'editing'" key="editing">
    Cancel
  </button>
</transition>
```
可以重写为：
```
<transition>
  <button v-bind:key="docState">
    {{ buttonMessage }}
  </button>
</transition>

// ...
computed: {
  buttonMessage: function () {
    switch (this.docState) {
      case 'saved': return 'Edit'
      case 'edited': return 'Save'
      case 'editing': return 'Cancel'
    }
  }
}
```

##### 过渡模式

在 “on” 按钮和 “off” 按钮之间过渡时，这两个按钮会同步渲染 - 当一个过渡进入时，另一个过渡离开。这是 `<transition>` 的默认行为 - 进入和离开同时发生。

有时这样做是非常合理的，比如，在过渡的条目都是绝对定位时。还可以将元素位移，使它们看起来具有滑动过渡效果

同时生效的进入式和离开式过渡不能满足所有要求，所以 Vue 提供了可选的过渡模式：

- `in-out`：新元素先过渡进入(transition in)，过渡完成之后，当前元素过渡离开(transition out)。
- `out-in`：当前元素先过渡离开(transition out)，过渡完成之后，新元素过渡进入(transition in)。

现在，让我们用 `out-in` 模式，更新下前面的 on/off 按钮过渡：
```
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```
只需添加一个额外的属性，就解决了最初的过渡问题，而无需添加任何特殊样式。

#### 多个组件之间过渡

多个组件之间的过渡甚至更简单 - 我们不需要使用 `key` 属性。相反，我们需要使用动态组件:
```
.component-fade-enter-active, .component-fade-leave-active {
  transition: opacity .3s ease;
}
.component-fade-enter, .component-fade-leave-to
/* .component-fade-leave-active 在低于 2.1.8 版本中 */ {
  opacity: 0;
}

<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>

new Vue({
  el: '#transition-components-demo',
  data: {
    view: 'v-a'
  },
  components: {
    'v-a': {
      template: '<div>Component A</div>'
    },
    'v-b': {
      template: '<div>Component B</div>'
    }
  }
})
```

#### 列表过渡

目前为止，关于过渡我们已经完成了：

- 单个节点
- 多个节点，其中每次只渲染一个

那么，当我们整个列表的每一项（例如使用 `v-for`）都需要同时进行渲染呢？

在这种情况下，我们将使用 `<transition-group>` 组件。在我们深入示例之前，先来了解关于这个组件的一些要点：

- 不同于 `<transition>`，它会以一个真实元素渲染：默认为 `span`。你也可以通过 `tag` 属性更换为其他渲染元素
- 它内部的元素必须具有唯一的 `key` 属性

##### 进入式/离开式列表过渡

现在让我们来深入一个示例，进入式过渡和离开式过渡都使用与之前相同的 CSS 类名：
```
.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to /* .list-leave-active 在低于 2.1.8 版本中 */ {
  opacity: 0;
  transform: translateY(30px);
}

<div id="list-demo">
  <button v-on:click="add">Add</button>
  <button v-on:click="remove">Remove</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>

new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
```
这个示例有个问题，当添加和移除项目时，其周围的项目会瞬间猛地移动到新的位置，而不是平滑过渡，我们稍后会解决这个问题。

##### 列表的位移过渡

`<transition-group>` 组件还有一个暗藏玄机之处。不仅可以在进入和离开时进行动画，还可以在位置改变时进行动画。使用此功能所需要知道的唯一新的概念是，在项目位置改变时添加额外的 `v-move` 类名。

与其他类名相同，它的前缀也和设置的 `name` 属性的值相匹配，你也可以通过 `move-class` 属性来手动指定类名。

此类名对于指定过渡时间和 easing 过渡曲线非常有用，如下所示：
```
.flip-list-move {
  transition: transform 1s;
}

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="flip-list-demo" class="demo">
  <button v-on:click="shuffle">Shuffle</button>
  <transition-group name="flip-list" tag="ul">
    <li v-for="item in items" v-bind:key="item">
      {{ item }}
    </li>
  </transition-group>
</div>

new Vue({
  el: '#flip-list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9]
  },
  methods: {
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```
这个看起来很神奇，内部的实现原理是，Vue 使用了一个叫 FLIP 动画技术，可以通过使用 `transform` 将元素从之前的位置平滑过渡到新的位置。

我们可以将此技术与我们以前的实施相结合，为我们列表所有可能的位置变更都添加动画！
```
.list-complete-item {
  transition: all 1s;
  display: inline-block;
  margin-right: 10px;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active 在低于 2.1.8 版本中 */ {
  opacity: 0;
  transform: translateY(30px);
}
.list-complete-leave-active {
  position: absolute;
}

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.14.1/lodash.min.js"></script>

<div id="list-complete-demo" class="demo">
  <button v-on:click="shuffle">Shuffle</button>
  <button v-on:click="add">Add</button>
  <button v-on:click="remove">Remove</button>
  <transition-group name="list-complete" tag="p">
    <span
      v-for="item in items"
      v-bind:key="item"
      class="list-complete-item"
    >
      {{ item }}
    </span>
  </transition-group>
</div>

new Vue({
  el: '#list-complete-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
    shuffle: function () {
      this.items = _.shuffle(this.items)
    }
  }
})
```

需要注意的是，使用 FLIP 过渡的元素，在设置为 `display: inline` 时，无法正常运行。作为替代方案，可以将元素设置为 `display: inline-block`，或者将元素放置于 flex 上下文(flex context)中。

FLIP 动画不局限于单个轴线方向(single axis)，多个维度网格(multidimensional grid)也同样可以过渡

##### 列表的渐进过渡

通过 data 属性与 JavaScript 式过渡的通信，就可以实现列表的逐项渐进过渡：
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="staggered-list-demo">
  <input v-model="query">
  <transition-group
    name="staggered-fade"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
</div>

new Vue({
  el: '#staggered-list-demo',
  data: {
    query: '',
    list: [
      { msg: 'Bruce Lee' },
      { msg: 'Jackie Chan' },
      { msg: 'Chuck Norris' },
      { msg: 'Jet Li' },
      { msg: 'Kung Fury' }
    ]
  },
  computed: {
    computedList: function () {
      var vm = this
      return this.list.filter(function (item) {
        return item.msg.toLowerCase().indexOf(vm.query.toLowerCase()) !== -1
      })
    }
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
      el.style.height = 0
    },
    enter: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 1, height: '1.6em' },
          { complete: done }
        )
      }, delay)
    },
    leave: function (el, done) {
      var delay = el.dataset.index * 150
      setTimeout(function () {
        Velocity(
          el,
          { opacity: 0, height: 0 },
          { complete: done }
        )
      }, delay)
    }
  }
})

```

#### 可复用的过渡

通过 Vue 的组件系统可以实现复用过渡。要创建一个可复用过渡，你需要做的就是将 `<transition>` 或者 `<transition-group>` 作为组件根节点，然后将全部子内容放置在 `transition` 组件中就可以了。

这里是使用 `template` 的组件的简单示例：
```
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```
函数组件更适合完成这个任务：
```
Vue.component('my-special-transition', {
  functional: true,
  render: function (createElement, context) {
    var data = {
      props: {
        name: 'very-special-transition',
        mode: 'out-in'
      },
      on: {
        beforeEnter: function (el) {
          // ...
        },
        afterEnter: function (el) {
          // ...
        }
      }
    }
    return createElement('transition', data, context.children)
  }
})
```

#### 动态过渡

其实，在 Vue 中即使是过渡也是由数据驱动的！动态过渡最基本的例子是，将 `name` 属性(attribute)和动态属性(dynamic property)绑定在一起。
```
<transition v-bind:name="transitionName">
  <!-- ... -->
</transition>
```
当你使用多个 Vue 过渡类名约定，来定义 CSS 过渡/动画，并在不同的类名约定之间切换时，动态过渡会非常有用。

所有的过渡属性都可以动态绑定。并且不仅是属性，由于事件钩子函数都是 Vue 的方法(methods)，所以可以从 `this` 上下文访问到所有数据。这意味着，根据组件的状态，JavaScript 式过渡的表现可能会有所不同。
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/velocity/1.2.3/velocity.min.js"></script>

<div id="dynamic-fade-demo" class="demo">
  Fade In: <input type="range" v-model="fadeInDuration" min="0" v-bind:max="maxFadeDuration">
  Fade Out: <input type="range" v-model="fadeOutDuration" min="0" v-bind:max="maxFadeDuration">
  <transition
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <p v-if="show">hello</p>
  </transition>
  <button
    v-if="stop"
    v-on:click="stop = false; show = false"
  >Start animating</button>
  <button
    v-else
    v-on:click="stop = true"
  >Stop it!</button>
</div>

new Vue({
  el: '#dynamic-fade-demo',
  data: {
    show: true,
    fadeInDuration: 1000,
    fadeOutDuration: 1000,
    maxFadeDuration: 1500,
    stop: true
  },
  mounted: function () {
    this.show = false
  },
  methods: {
    beforeEnter: function (el) {
      el.style.opacity = 0
    },
    enter: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 1 },
        {
          duration: this.fadeInDuration,
          complete: function () {
            done()
            if (!vm.stop) vm.show = false
          }
        }
      )
    },
    leave: function (el, done) {
      var vm = this
      Velocity(el,
        { opacity: 0 },
        {
          duration: this.fadeOutDuration,
          complete: function () {
            done()
            vm.show = true
          }
        }
      )
    }
  }
})
```
最后，创建动态过渡的最终方案是，组件通过接受 prop 来动态修改之前的过渡。还是那句话，唯一限制你的是想象力。


### 状态间的过渡

Vue 的过渡系统提供了很多简便方法，可以在进入、离开时，以及对列表进行动画，但是对于数据变化的动画要如何处理呢？比如：

- 数字和计算
- 颜色显示
- SVG 节点的位置
- 元素大小和其他属性

所有以上这些要么是存储为原始数字，要么可以转化为数字。一旦我们实现这步，就可以结合 Vue 的响应式和组件系统，使用第三方库补充中间状态(tween state)，以对这些状态变化进行动画。

#### 使用 `watcher` 对状态进行动画(Animating State with Watchers)

`watcher` 可以观察到从任何数值属性到另一个属性的变化，然后进行动画。这一抽象听起来很复杂，所以让我们深入一个使用 GreenSock 的例子：
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/1.20.3/TweenMax.min.js"></script>

<div id="animated-number-demo">
  <input v-model.number="number" type="number" step="20">
  <p>{{ animatedNumber }}</p>
</div>

new Vue({
  el: '#animated-number-demo',
  data: {
    number: 0,
    tweenedNumber: 0
  },
  computed: {
    animatedNumber: function() {
      return this.tweenedNumber.toFixed(0);
    }
  },
  watch: {
    number: function(newValue) {
      TweenLite.to(this.$data, 0.5, { tweenedNumber: newValue });
    }
  }
})
```

当更新数值时，输入框下方就会对数值的更改进行动画。这是一个不错的 demo 示例，但是，对于不能直接存储为数字的值，比如有效的 CSS 颜色值？以下是我们如何通过 Tween.js 和 Color.js 来实现对非数值进行动画：
```
.example-7-color-preview {
  display: inline-block;
  width: 50px;
  height: 50px;
}

<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>
<script src="https://cdn.jsdelivr.net/npm/color-js@1.0.3"></script>

<div id="example-7">
  <input
    v-model="colorQuery"
    v-on:keyup.enter="updateColor"
    placeholder="Enter a color"
  >
  <button v-on:click="updateColor">Update</button>
  <p>Preview:</p>
  <span
    v-bind:style="{ backgroundColor: tweenedCSSColor }"
    class="example-7-color-preview"
  ></span>
  <p>{{ tweenedCSSColor }}</p>
</div>

var Color = net.brehaut.Color

new Vue({
  el: '#example-7',
  data: {
    colorQuery: '',
    color: {
      red: 0,
      green: 0,
      blue: 0,
      alpha: 1
    },
    tweenedColor: {}
  },
  created: function () {
    this.tweenedColor = Object.assign({}, this.color)
  },
  watch: {
    color: function () {
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween(this.tweenedColor)
        .to(this.color, 750)
        .start()

      animate()
    }
  },
  computed: {
    tweenedCSSColor: function () {
      return new Color({
        red: this.tweenedColor.red,
        green: this.tweenedColor.green,
        blue: this.tweenedColor.blue,
        alpha: this.tweenedColor.alpha
      }).toCSS()
    }
  },
  methods: {
    updateColor: function () {
      this.color = new Color(this.colorQuery).toRGB()
      this.colorQuery = ''
    }
  }
})
```
#### 动态状态过渡(Dynamic State Transitions)

就像 Vue 的过渡组件那样，动态数据所支撑的状态的过渡也可以实时更新，这对于制作原型十分有用！即使是一个简单的 SVG 多边形，在操作一些变量之后，也可是实现很多难以想象的效果。

#### 通过组件组织过渡(Organizing Transitions into Components)

管理太多的状态过渡，会快速增加 Vue 实例或者组件的复杂性。幸运的是，许多动画可以提取到专用的子组件中。让我们用前面的整数动画来举例：
```
<script src="https://cdn.jsdelivr.net/npm/tween.js@16.3.4"></script>

<div id="example-8">
  <input v-model.number="firstNumber" type="number" step="20"> +
  <input v-model.number="secondNumber" type="number" step="20"> =
  {{ result }}
  <p>
    <animated-integer v-bind:value="firstNumber"></animated-integer> +
    <animated-integer v-bind:value="secondNumber"></animated-integer> =
    <animated-integer v-bind:value="result"></animated-integer>
  </p>
</div>

// 在我们的应用程序中，所有的整数动画，
// 现在都可以复用这种复杂的补间动画逻辑。
// 通过对组件配置更多的动态过渡策略
// 和复杂过渡策略，
// 可以使我们的界面十分清晰
Vue.component('animated-integer', {
  template: '<span>{{ tweeningValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data: function () {
    return {
      tweeningValue: 0
    }
  },
  watch: {
    value: function (newValue, oldValue) {
      this.tween(oldValue, newValue)
    }
  },
  mounted: function () {
    this.tween(0, this.value)
  },
  methods: {
    tween: function (startValue, endValue) {
      var vm = this
      function animate () {
        if (TWEEN.update()) {
          requestAnimationFrame(animate)
        }
      }

      new TWEEN.Tween({ tweeningValue: startValue })
        .to({ tweeningValue: endValue }, 500)
        .onUpdate(function (object) {
          vm.tweeningValue = object.tweeningValue.toFixed(0)
        })
        .start()

      animate()
    }
  }
})

// 现在，可以将所有的动画复杂度，从 Vue 主实例中移除！
new Vue({
  el: '#example-8',
  data: {
    firstNumber: 20,
    secondNumber: 40
  },
  computed: {
    result: function () {
      return this.firstNumber + this.secondNumber
    }
  }
})
```
在子组件中，我们可以使用本页面所涵盖的所有过渡策略进行组合，再通过 Vue 提供的内置过渡系统 。将这些结合在一起，对于要实现的动画效果的限制很少。


#### 赋予设计以生命

通过定义动画，可以给我们的设计带来生命。不幸的是，当设计师创建图标、logo 和吉祥物的时候，他们通常交付的都是图片或静态 SVG。所以，虽然 GitHub 章鱼猫、Twitter 小鸟以及其它许多 logo 都类似于生灵，它们看上去缺乏生气。

Vue 可以提供帮助。由于 SVG 的本质是数据，我们只需要提供这些动物活跃、思考或惊恐状态下的图例。然后 Vue 通过使用这些图例，来实现这几种状态之间的过渡动画，以制作欢迎页面、加载指示、以及触发情感的提示信息。

例子：https://codepen.io/sdras/pen/YZBGNp



## 可重用 & 合成

### mixin

#### 基础

mixin 是分发 Vue 组件的可复用功能的一种非常灵活方式。每个 mixin 对象可以包含全部组件选项。当组件使用 mixin 对象时，mixin 对象中的全部选项，都会被“混入(mix)”到组件自身的选项当中。

例如：
```
// 定义一个 mixin 对象
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('来自 mixin 对象的 hello！')
    }
  }
}

// 定义一个使用以上 mixin 对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // => "来自 mixin 对象的 hello！"
```

#### 选项合并(option merging)

当 mixin 对象和组件自身的选项对象，在二者选项名称相同时，Vue 会选取合适的“合并(merge)”策略。

例如，对 `data` 对象进行浅合并（单个属性深合并），在冲突情况下，优先使用组件的 `data`。
```
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```
具有相同名称的钩子函数，将合并到一个数组中，最终它们会被依次调用。此外，需要注意，mixin 对象中的同名钩子函数，会在组件自身的钩子函数之前调用。
```
var mixin = {
  created: function () {
    console.log('mixin 对象的钩子函数被调用')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('组件的钩子函数被调用')
  }
})

// => "mixin 对象的钩子函数被调用"
// => "组件的钩子函数被调用"
```
例如，`methods`, `components` 和 `directives`，这些值为对象的选项(option)，则会被合并到相应的选项对象上。但是，在二者的键名(key)发生冲突时，就会优先使用组件选项对象中键值对：
```
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})

vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
```
注意，在 `Vue.extend()` 中，Vue 也使用了与此相同的合并策略。

#### 全局 mixin(global mixin)

也可以在全局使用 mixin。请谨慎使用！一旦在全局中使用了 mixin，就会影响到之后创建的每个实例。在用法正确时，可以为自定义选项注入处理逻辑：
```
// 为自定义选项 `myOption` 注入一个处理函数
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

请少量且小心谨慎地使用全局 `mixin`，因为这会影响到之后创建的每个 Vue 实例，包括第三方组件也会受到影响。在多数情况下，如同以上示例中所演示的，你应该只将全局 `mixin`，用于自定义选项的处理逻辑。

还有一个比较不错的做法，就是在插件中使用全局 `mixin`，然后通过调用插件来复用组件功能，以避免应用程序中的重复部分。

#### 自定义选项的合并策略(custom option merge strategies)

在合并自定义选项(custom option)时，Vue 会使用默认策略，即覆盖已有值。如果想要定制自定义选项的合并逻辑，则需要向 `Vue.config.optionMergeStrategies` 添加一个函数：
```
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // return mergedVal
}
```
对于大多数基于对象(object-based)的选项，可以使用 `methods` 的合并策略：
```
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```
可以在 Vuex 1.x 的合并策略里，找到一个高级用法示例：
```
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```

### 自定义指令

#### 简介

除了 Vue 核心携带着的一些默认指令（`v-model` 和 `v-show`）之外，Vue 还允许你注册自己的自定义指令。请注意，在 Vue 2.0 中，代码重用和抽象(reuse and abstraction)的主要是以组件的形式

- 但是，可能某些情况下，还是需要对普通元素进行一些底层 DOM 访问，这也是自定义指令仍然有其使用场景之处。

接着来看一个在输入元素获取焦点的示例

当页面加载时，元素将获取焦点（注意：自动获取焦点在 Safari 手机上无法运行）。事实上，在访问页面时，如果你还没有点击任何地方，上面的输入框现在应该处于获取焦点的状态。现在让我们构建指令以完成此效果：
```
// 注册一个名为 `v-focus` 的全局自定义指令
Vue.directive('focus', {
  // 当绑定的元素插入到 DOM 时调用此函数……
  inserted: function (el) {
    // 元素调用 focus 获取焦点
    el.focus()
  }
})
```
如果你想要注册一个局部指令，也可以通过设置组件的 directives 选项：
```
directives: {
  focus: {
    // 指令定义对象
    inserted: function (el) {
      el.focus()
    }
  }
}
```
然后在模板中，你可以在任何元素上使用新增的 v-focus 属性，就像这样：
```
<input v-focus>
```

#### 钩子函数

指令的定义对象提供了几个钩子函数（全部可选）：

- `bind`：在指令第一次绑定到元素时调用，只会调用一次。可以在此钩子函数中，执行一次性的初始化设置。

- `inserted`：在已绑定的元素插入到父节点时调用（只能保证父节点存在，不一定存在于 document 中）。

- `update`：在包含指令的组件的 VNode 更新后调用，但可能之前其子组件已更新。指令的值可能更新或者还没更新，然而可以通过比较绑定的当前值和旧值，来跳过不必要的更新（参考下面的钩子函数）。

- `componentUpdated`：在包含指令的组件的 VNode 更新后调用，并且其子组件的 VNode 已更新。

- `unbind`：在指令从元素上解除绑定时调用，只会调用一次。


#### 指令钩子函数参数

指令钩子函数可传入以下参数：

- `el`：指令绑定的元素。可以用于直接操作 DOM。
- `binding`：一个对象，包含以下属性：
    - `name`：指令的名称，不包括 `v-` 前缀。
    - `value`：向指令传入的值。例如，在 `v-my-directive="1 + 1"` 中，传入的值是 2。
    - `oldValue`：之前的值，只在 `update` 和 `componentUpdated` 钩子函数中可用。无论值是否发生变化，都可以使用。
    - `expression`：指令绑定的表达式(expression)，以字符串格式。例如，在 `v-my-directive="1 + 1"` 中，表达式是 "1 + 1"。
    - `arg`：向指令传入的参数，如果有的话。例如，在 `v-my-directive:foo` 中，参数是 "foo"。
    - `modifiers`：一个对象，包含修饰符，如果有的话。例如，在 `v-my-directive.foo.bar` 中，修饰符是 `{ foo: true, bar: true }`。
- `vnode`：由 Vue 编译器(Vue’s compiler)生成的虚拟 Node 节点(virtual node)。更多细节请查看 VNode API。
- `oldVnode`：之前的虚拟 Node 节点(virtual node)，只在 `update` 和 `componentUpdated` 钩子函数中可用。

除了 `el` 之外的其他参数，都应该是只读的，并且永远不要去修改它们。如果你需要通过钩子函数共享信息数据，推荐通过元素的 `dataset` 来实现。

下面是使用这些属性的自定义指令的示例：
```
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>

Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

#### 函数简写(Function Shorthand)

在许多情况下，可能只需要在 `bind` 和 `update` 钩子函数上定义过相同的行为就足够了，而无需关心其他钩子函数。例如：
```
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

#### 对象字面量(Object Literals)

如果指令需要多个值，你还可以向指令传入 JavaScript 对象字面量(object literal)。记住，指令能够接收所有有效的 JavaScript 表达式。
```
<div v-demo="{ color: 'white', text: 'hello!' }"></div>

Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```

### 过滤器

在 Vue.js 中，可以定义过滤器(filter)，常用于格式化文本。

过滤器可以在两种场景中使用：双花括号插值(mustache interpolation)和 v-bind 表达式（后者在 2.1.0+ 版本支持）。

过滤器应该追加到 JavaScript 表达式的末尾，以“管道符号(pipe symbol)”表示：
```
<!-- in mustaches -->
{{ message | capitalize }}

<!-- in v-bind -->
<div v-bind:id="rawId | formatId"></div>
```
你可以在组件的选项中定义局部过滤器：
```
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```
或者在创建 Vue 示例之前，定义一个全局的过滤器：
```
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```
过滤器函数总是接收表达式的值（值的结果是，过滤器链中的上一个过滤器返回的值），作为第一个参数。

可以如下链式调用过滤器：
```
{{ message | filterA | filterB }}
```

在这个例子中，`filterA` 被定义为接收单个参数的过滤器函数，表达式 `message` 的值将作为参数传入到函数中，然后继续调用同样被定义为接收单个参数的过滤器函数 `filterB`，将 `filterA` 的结果传递到 `filterB` 中。

由于过滤器函数是 JavaScript 函数，也因此可以接收多个参数：
```
{{ message | filterA('arg1', arg2) }}
```
这里，`filterA` 被定义为接收三个参数的过滤器函数。其中 `message` 的值作为第一个参数，普通字符串 `'arg1'` 作为第二个参数，表达式 `arg2` 取值后的值作为第三个参数。

更多：https://github.com/wy-ei/vue-filter

### 插件

#### 编写插件

插件通常用于为 Vue 添加全局级别的功能。然而对于插件，并没有严格限定其使用范围 - 下面是常见的几种插件类型：

1. 添加一些全局方法或属性。例如 `vue-custom-element`

2. 添加一个或多个全局资源(asset)：指令(directives)/过滤器(filters)/过渡(transitions) 等。例如 `vue-touch`

3. 通过全局 mixin，添加一些组件选项。例如 `vue-router`

4. 添加一些 Vue 实例方法，通过把这些方法添加到 `Vue.prototype `上实现。

5. 一个可以提供 API 的库(library)，与此同时也是以上功能的组合。例如 `vue-router`

Vue.js 插件应该暴露一个 `install` 方法。此方法在调用时，将 Vue 构造函数作为第一个参数传入，以及将一个可选的选项作为第二个参数传入：
```
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 一些逻辑……
  }

  // 2. 添加一个全局资源(asset)
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 一些逻辑……
    }
    ...
  })

  // 3. 注入一些组件选项
  Vue.mixin({
    created: function () {
      // 一些逻辑……
    }
    ...
  })

  // 4. 添加一个实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 一些逻辑……
  }
}
```

#### 使用插件

通过调用全局方法 Vue.use() 使用插件：
```
// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin)
```
可以根据情况，传入一些可选的选项：
```
Vue.use(MyPlugin, { someOption: true })
```
`Vue.use` 会自动阻止多次使用同一个插件，所以对于同一个插件的多次调用，将只安装一次。

Vue.js 官方提供的一些插件（例如 `vue-router`），如果检测到 Vue 是可访问的全局变量，这些插件会自动调用 `Vue.use()`。然而在例如 CommonJS 的模块环境中，你应该始终显式地调用 `Vue.use()`：
```
// 在使用由 Browserify 或 webpack 这些模块打包工具，提供的 CommonJS 模块环境时
var Vue = require('vue')
var VueRouter = require('vue-router')

// 不要忘记调用此方法
Vue.use(VueRouter)
```
在 `awesome-vue` 文档中，汇集了大量由社区贡献的插件(plugins)和库(libraries)。

如果检测到 Vue 是可访问的全局变量，这些插件会自动调用 `Vue.use()`

可以参考 https://github.com/vuejs/vue-router/blob/dev/src/index.js#L233
```
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter)
}
```

### render 函数 & jsx

#### 基础

在绝大多数情况下，Vue 推荐你使用模板来组织构建 HTML。然而在一些场景下，你确实需要完整的 JavaScript 编程能力。作为模板的替代增强，你可以使用render 函数，它更接近编译器。

让我们深入一个例子，其中，使用 `render` 函数是比较推荐的方式。假设你要生成固定的标题：
```
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```
对于以上 HTML，你决定要使用如下组件接口：
```
<anchored-heading :level="1">Hello world!</anchored-heading>
```

当你开始创建一个只根据 `level` prop 生成标题的组件时，你会很快想到这样实现：
```
<script type="text/x-template" id="anchored-heading-template">
  <h1 v-if="level === 1">
    <slot></slot>
  </h1>
  <h2 v-else-if="level === 2">
    <slot></slot>
  </h2>
  <h3 v-else-if="level === 3">
    <slot></slot>
  </h3>
  <h4 v-else-if="level === 4">
    <slot></slot>
  </h4>
  <h5 v-else-if="level === 5">
    <slot></slot>
  </h5>
  <h6 v-else-if="level === 6">
    <slot></slot>
  </h6>
</script>


Vue.component('anchored-heading', {
  template: '#anchored-heading-template',
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```
在这种场景中，并不适合使用模板。不仅会很繁琐，而且要在每个不同级的 `h` 标签中重复 `<slot></slot>`，并且在其中添加固定元素时也必须做同样的事。

虽然大多数组件中都非常适合使用模板，但是在这里明显是一种特殊场景。因此，让我们尝试使用 `render` 函数来重写上面的示例：
```
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // 标签名称
      this.$slots.default // 由子节点构成的数组
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```
现在显得简单清晰！一定程度上。代码变得简短很多，但是需要更加熟悉 Vue 实例属性。

在这种情况下，你必须知道在不使用 `slot` 属性向组件中传递内容时（比如 anchored-heading 中的 Hello world!），这些子节点会被存储到组件实例的 $slots.default 中。

如果你对此还不够了解，在深入 render 函数之前，建议先阅读实例属性 API。


#### nodes, trees 和 virtual DOM

在我们深入 `render` 函数之前，略微了解浏览器的工作原理是比较重要的事情。以如下 HTML 为例：
```
<div>
  <h1>My title</h1>
  Some text content
  <!-- TODO: Add tagline  -->
</div>
```
在浏览器读取这段代码时，会构建一个 “DOM 节点”树(tree of “DOM nodes”)，以帮助浏览器跟踪所有情况，就像通过创建一个家谱树，来跟踪一个大家族。

对于以上 HTML，其 DOM 节点树如下：
![DOM树](对https://vuefe.cn/images/dom-tree.png)

每个元素都是一个节点。每一段文字都是一个节点。甚至注释也都是节点！节点只是页面的一部分。正如在一棵家谱树中一样，每个节点都可以有子节点（也就是说，每个节点都可以包含多个子节点）。

有效地更新所有这些节点可能是很困难的，但幸运的是，你无需手动执行。相反，只需在 Vue 模板中，在页面中添加你需要用到的 HTML：
```
<h1>{{ blogTitle }}</h1>
```
或者通过一个 `render` 函数：
```
render: function (createElement) {
  return createElement('h1', this.blogTitle)
}
```
在这两种场景中，Vue 会自动保持页面更新，更确切地说，在 `blogTitle` 修改时，也同样能够及时更新。

##### `virtual` DOM

Vue 通过实现一个 `virtual` DOM，来跟踪那些「真正需要对 DOM 所做的修改」。仔细看看这一行：
```
return createElement('h1', this.blogTitle)
```
`createElement` 实际返回的是什么呢？准确地说，返回的并非一个真正的 DOM 元素。

可能更确切地说，应该将其命名为 `createNodeDescription`（译注：`createNodeDescription`，创建节点描述），包含「哪些节点应该由 Vue 渲染在页面上」的相关描述信息，也包括所有子节点的相关描述。

我们将以上这个节点描述称为 “virtual node”（译注：virtual node，虚拟节点），通常缩写为 VNode。”virtual DOM” 就是由 Vue 组件树构建出来的，被称为整个 VNodes 树。

##### `createElement` 参数

接下来你必须熟悉的事情是，如何在 `createElement` 函数中使用模板功能。以下是 `createElement` 接受的参数：
```
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // HTML 标签名称，或组件选项对象，或一个函数，
  // 其中函数会返回 HTML 标签名称或组件选项对象。必选参数。
  'div',

  // {Object}
  // 一个数据对象，
  // 和模板中用到的属性相对应。可选参数。
  {
    // （详情请看下一节）
  },

  // {String | Array}
  // 子虚拟 DOM 节点(children VNode)组成，或使用 `createElement()` 生成，
  // 或使用字符串的“文本虚拟 DOM 节点(text VNode)”。可选参数。
  [
    'Some text comes first.',
    createElement('h1', 'A headline'),
    createElement(MyComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```

##### 深入了解数据对象(The Data Object In-Depth)

需要注意的一点：类似于 `v-bind:class` 和 `v-bind:style` 在模板中有着特殊的处理，下面这些字段(field)都处于 VNode 数据对象的顶层(top-level)。

此对象还允许绑定普通的 HTML 属性和 DOM 属性，如 `innerHTML`（这将替换为 v-html 指令）：
```
{
  // 和 `v-bind:class` 的 API 相同
  class: {
    foo: true,
    bar: false
  },
  // 和 `v-bind:style` 的 API 相同
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 普通的 HTML 属性
  attrs: {
    id: 'foo'
  },
  // 组件 props
  props: {
    myProp: 'bar'
  },
  // DOM 属性
  domProps: {
    innerHTML: 'baz'
  },
  // 事件处理程序嵌套在 `on` 字段下，
  // 然而不支持在 `v-on:keyup.enter` 中的修饰符。
  // 因此，你必须手动检查
  // 处理函数中的 keyCode 值是否为 enter 键值。
  on: {
    click: this.clickHandler
  },
  // 仅对于组件，
  // 用于监听原生事件，而不是组件内部
  // 使用 `vm.$emit` 触发的事件。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令。
  // 注意，由于 Vue 会追踪旧值，
  // 所以不能对`绑定`的`旧值`设值
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // 作用域插槽(scoped slot)的格式如下
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果此组件是另一个组件的子组件，
  // 需要为插槽(slot)指定名称
  slot: 'name-of-slot',
  // 其他特殊顶层(top-level)属性
  key: 'myKey',
  ref: 'myRef'
}
```

##### 完整示例

有了这些知识，我们现在就可以完成开始想实现的组件：
```
var getChildrenTextContent = function (children) {
  return children.map(function (node) {
    return node.children
      ? getChildrenTextContent(node.children)
      : node.text
  }).join('')
}

Vue.component('anchored-heading', {
  render: function (createElement) {
    // create kebabCase id
    var headingId = getChildrenTextContent(this.$slots.default)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^\-|\-$)/g, '')

    return createElement(
      'h' + this.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, this.$slots.default)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

##### 限制(Constraints)

VNodes 必须唯一

组件树中的所有 VNode 必须是唯一的。也就是说，以下 `render` 函数无效：
```
render: function (createElement) {
  var myParagraphVNode = createElement('p', 'hi')
  return createElement('div', [
    // 哟 - VNode 重复！
    myParagraphVNode, myParagraphVNode
  ])
}
```
如果你真的需要多次重复相同的元素/组件，你可以使用工厂函数来实现。例如，下面的 `render` 函数，完美有效地渲染了 20 个相同的段落。
```
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```

#### 使用原生 JavaScript 的替换模板功能(Replacing Template Features with Plain JavaScript)

##### `v-if` and `v-for`

无论什么功能，都可以在原生 JavaScript 中轻松实现，所以 Vue 的 `render` 函数无需提供专用的替代方案。例如，在模板中使用 `v-if` 和 `v-for`：
```
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```
这可以在 `render` 函数中，通过重写为 JavaScript 的 `if/else` 和 `map` 来实现：
```
props: ['items'],
render: function (createElement) {
  if (this.items.length) {
    return createElement('ul', this.items.map(function (item) {
      return createElement('li', item.name)
    }))
  } else {
    return createElement('p', 'No items found.')
  }
}
```

##### `v-model`

在 render 函数中没有直接和 `v-model` 对标的功能 - 你必须自己实现逻辑：
```
props: ['value'],
render: function (createElement) {
  var self = this
  return createElement('input', {
    domProps: {
      value: self.value
    },
    on: {
      input: function (event) {
        self.$emit('input', event.target.value)
      }
    }
  })
}
```
这就是深入底层实现需要付出的代价，但是和 `v-model` 相比较，这也可以更好地控制交互细节。

##### 事件 & 按键修饰符(Event & Key Modifiers)

对于 `.passive`, `.capture` 和 `.once` 这样的事件修饰符, Vue 提供了用于 `on` 的前缀:

修饰符	|  前缀
-------|----------
.passive |  &
.capture |  !
.once    |  ~
.capture.once  或 .once.capture |  ~!

示例:
```
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  '~!mouseover': this.doThisOnceInCapturingMode
}
```
对于其他的事件和关键字修饰符, 你可以在处理程序中使用事件方法实现：

Modifier(s)	|  Modifier(s)
-------|----------
.stop    |  event.stopPropagation()
.prevent |  	event.preventDefault()
.self    |  	if (event.target !== event.currentTarget) return
Keys: .enter, .13 |  if (event.keyCode !== 13) return (change 13 to another key code for other key modifiers)
Modifiers Keys: .ctrl, .alt, .shift, .meta |  if (!event.ctrlKey) return (change ctrlKey to altKey, shiftKey, or metaKey, respectively)

这里是所有修饰符一起使用的例子:
```
on: {
  keyup: function (event) {
    // 如果触发事件的元素不是事件绑定的元素
    // 则返回
    if (event.target !== event.currentTarget) return

    // 如果按下去的不是 enter 键或者
    // 没有同时按下 shift 键
    // 则返回
    if (!event.shiftKey || event.keyCode !== 13) return

    // 阻止事件冒泡
    event.stopPropagation()

    // 阻止该元素默认的 keyup 事件
    event.preventDefault()
    // ...
  }
}
```

##### Slots

使用 `this.$slots` 访问静态插槽内容作为 VNode 数组：
```
render: function (createElement) {
  // <div><slot></slot></div>
  return createElement('div', this.$slots.default)
}
```

使用 `this.$scopedSlots` 访问作用域插槽作为返回 VNodes 的函数:
```
props: ['message'],
render: function (createElement) {
  // `<div><slot :text="message"></slot></div>`
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.message
    })
  ])
}
```

使用 `render` 函数传递作用域插槽到子组件，使用 VNode 数据中的 `scopedSlots` 关键字：
```
render: function (createElement) {
  return createElement('div', [
    createElement('child', {
      // pass scopedSlots in the data object
      // in the form of { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
```

#### JSX

如果你写了很多 `render` 函数，可能会觉得痛苦：
```
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)
```
特别是模板如此简单的情况下：
```
<anchored-heading :level="1">
    <span>Hello</span> world!
</anchored-heading>
```
这就是会有一个 Babel plugin 插件，用于在 Vue 中使用 JSX 语法的原因，它可以让我们回到于更接近模板的语法上。（插件：https://github.com/vuejs/babel-plugin-transform-vue-jsx）
```
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render: function (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```
将 h 作为 `createElement` 的别名是 Vue 生态系统中的一个通用惯例，实际上也是 JSX 所要求的，如果在作用域中 h 失去作用， 在应用中会触发报错。

使用文档：https://github.com/vuejs/babel-plugin-transform-vue-jsx#usage

#### 函数式组件

前面我们创建的锚点标题组件比较简单，没有管理维护状态，或者 `watch` 任何传递给它的状态，也没有生命周期方法。它只是一个接收 props 的函数。

在这个例子中，我们将组件记为 `functional`，这意味它无状态（没有 `data`），无实例（没有 `this` 上下文）。一个函数式组件就像这样：
```
Vue.component('my-component', {
  functional: true,
  // 为了弥补缺少的实例
  // 我们提供了第二个参数 context 作为上下文
  render: function (createElement, context) {
    // ...
  },
  // props 是可选项
  props: {
    // ...
  }
})
```

> 注意：在 2.3.0 之前的版本中，如果一个函数式组件想要接收 `props`，则 `props` 选项是必选项。而在 2.3.0+ 版本中，你可以省略 `props` 选项，所有组件节点上的发现的属性，都会被隐式地提取为 `props`。

在 2.5.0+ 版本，如果你使用的是单文件组件，则可以通过如下方式，声明基于模板的函数式组件：
```
<template functional>
</template>
```
组件需要的一切都是通过 `context` 传递，包括：

- `props`：一个提供 `props` 的对象
- `children`：一个由 VNode 子节点构成的数组
- `slots`：一个返回 `slots` 对象的函数
- `data`：传递给组件的整个 `data` 对象
- `parent`：一个父组件的引用
- `listeners`：（2.3.0+）一个包含父组件上注册的事件监听器的对象。这是一个指向 `data.on` 的别名。
- `injections`：（2.3.0+）如果使用了 `inject` 选项，则 `injections` 对象中包含了应当被注入的属性。

在添加 `functional: true` 之后，修改下锚点标题组件的 `render` 函数，需要添加 `context` 参数，将 `this.$slots.default` 修改为 `context.children`，然后将 `this.level` 修改为 `context.props.level`。

由于函数式组件只是函数，因此渲染性能开销也低很多。然而，由于缺少持久性实例，意味着它们不会出现在 Vue devtools 的组件树中。

在作为容器组件(wrapper component)时它们也同样非常有用。例如，当你需要实现如下需求：

- 程序化地在选择多个组件中的一个组件进行委派
- 在将 children, props, data 传递给子组件之前操作它们

以下是一个 `smart-list` 组件示例，我们能将它的行为委派给更加具体的组件中，这取决于向其传入的 `prop`：
```
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  }
})
```

##### 向子元素或子组件传递属性和事件

在普通组件中，非 Prop 属性将自动添加到组件的根元素中，并会替换或智能合并具有相同名称的所有已有属性。

然而，在函数式组件中，则要求你显式定义该行为：
```
Vue.component('my-functional-button', {
  functional: true,
  render: function (createElement, context) {
    // 清晰明确地传入所有属性(attribute)、事件监听器(event listener) 和 children。
    return createElement('button', context.data, context.children)
  }
})
```
向 `createElement` 传递 `context.data` 作为第二个参数，我们就把 `my-functional-button` 组件上所有的属性和事件监听器都向下进行传递。这是非常明确的行为，事实上，这些事件甚至不需要添加 `.native` 修饰符。

如果你在使用基于模板的函数式组件，那么你还必须手动添加属性和监听器。由于我们可以访问到其独立的上下文内容，所以我们可以使用 `data.attrs` 来延续传递所有 HTML 属性，以及使用 `listeners`（也就是 `data.on` 的别名）来延续传递所有事件监听器。
```
<template functional>
  <button
    class="btn btn-primary"
    v-bind="data.attrs"
    v-on="listeners"
  >
    <slot/>
  </button>
</template>
```

##### `slots()` 和 `children` 对比

你可能想知道为什么同时需要 `slots()` 和 `children`。`slots().default` 不是和 `children` 类似的吗？在一些场景中，是这样的 - 但是，如果是具有如下 `children` 的函数式组件呢？
```
<my-functional-component>
  <p slot="foo">
    first
  </p>
  <p>second</p>
</my-functional-component>
```
对于这个组件，`children` 会给你两个 `p` 标签，而 `slots().default` 只会传递第二个匿名 `p` 标签，`slots().foo` 会传递第一个具名 `p` 标签。

同时具有 `children` 和 `slots()`，可以帮助你进行明确选择，使组件确定是通过一个 `slot` 系统处理，还是直接延续传递 `children`，委派给其他组件去负责处理。







## 内部原理

### 深入响应式原理

现在是时候深入底层原理了！Vue 最显著的特性之一，就是侵入性不是很强的响应式系统(reactivity system)。

模型层(model)只是普通 JavaScript 对象，修改它则更新视图(view)。这会让状态管理变得简单且直观，不过理解它的工作原理以避免一些常见的问题也是很重要的。

在本节中，我们将开始深入挖掘 Vue 响应式系统的底层细节。

#### 如何追踪变化

把一个普通 Javascript 对象传给 Vue 实例的 `data` 选项，Vue 将遍历此对象所有的属性，并使用 `Object.defineProperty` 把这些属性全部转为 `getter/setter`。

Object.defineProperty 是仅 ES5 支持，且无法 shim 的特性，这也就是为什么 Vue 不支持 IE8 以及更低版本浏览器的原因。


`getter/setter` 对于用户来说是不可见的，但是在内部，通过它们可以让 Vue 在访问属性时进行依赖追踪，以及修改属性时进行变更通知。

这里需要注意的问题是浏览器控制台在打印数据对象时 `getter/setter` 的格式并不同，所以你可能需要安装 `vue-devtools` 来获取更加友好的检查接口。


每个组件实例都有相应的 `watcher` 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 `setter` 被调用时，会通知 `watcher` 重新计算，从而致使它关联的组件得以更新。

![追踪变化](https://vuefe.cn/images/data.png)


#### 变化检测问题

受现代 Javascript 的限制（以及废弃 Object.observe），Vue 无法检测到对象属性的添加或删除。

由于 Vue 会在初始化实例时对属性执行 `getter/setter` 转化过程，所以属性必须在 `data` 对象上存在才能让 Vue 转换它，这样才能让它是响应的。例如：
```
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```
Vue 不允许在已经创建的实例上动态添加新的根级响应式属性(root-level reactive property)。然而它可以使用 ` Vue.set(object, key, value)` 方法将响应属性添加到嵌套的对象上：
```
Vue.set(vm.someObject, 'b', 2)
```
你还可以使用 `vm.$set` 实例方法，这也是全局 `Vue.set` 方法的别名：
```
this.$set(this.someObject,'b',2)
```

有时你想向已有对象上添加一些属性，例如使用 `Object.assign()` 或 `_.extend()` 方法来添加属性。但是，添加到对象上的新属性不会触发更新。在这种情况下可以创建一个新的对象，让它包含原对象的属性和新的属性：
```
// 代替 `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```
也有一些数组相关的问题，之前已经在列表渲染中讲过。


#### 声明响应式属性

由于 Vue 不允许动态添加根级响应式属性，所以你必须在初始化实例前声明根级响应式属性，哪怕是一个空值：
```
var vm = new Vue({
  data: {
    // 声明 message 为一个空值字符串
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// 之后设置 `message`
vm.message = 'Hello!'
```
如果你在 data 选项中未声明 `message`，Vue 将警告你渲染函数在试图访问的属性不存在。

这样的限制在背后是有其技术原因的，它消除了在依赖项跟踪系统中的一类边界情况，也使 Vue 实例在类型检查系统的帮助下运行的更高效。

而且在代码可维护性方面也有一点重要的考虑：`data` 对象就像组件状态的概要，提前声明所有的响应式属性，可以让组件代码在以后重新阅读或其他开发人员阅读时更易于被理解。

#### 异步更新队列

可能你还没有注意到，Vue 异步执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。

如果同一个  `watcher` 被多次触发，只会一次推入到队列中。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要。

然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际（已去重的）工作。Vue 在内部尝试对异步队列使用原生的 `Promise.then` 和 `MessageChannel`，如果执行环境不支持，会采用 `setTimeout(fn, 0)` 代替。


例如，当你设置 `vm.someData = 'new value'` ，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。

多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。

为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 `Vue.nextTick(callback)` 。这样回调函数在 DOM 更新完成后就会调用。例如：
```
<div id="example">{{ message }}</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message' // 更改数据
vm.$el.textContent === 'new message' // false
Vue.nextTick(function () {
  vm.$el.textContent === 'new message' // true
})
```

在组件内使用 `vm.$nextTick()` 实例方法特别方便，因为它不需要全局 Vue ，并且回调函数中的 `this` 将自动绑定到当前的 Vue 实例上：
```
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => 'not updated'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => 'updated'
      })
    }
  }
})

```




## 工具

### 单文件组件

### 生产环境部署

### 单元测试

### TypeScript 支持



## 扩展升级

### 路由

### 状态管理

### 服务端渲染
