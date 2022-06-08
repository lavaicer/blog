---
title: "Note Vue"
date: 2022-04-18T09:18:07+08:00
draft: true
weight: 9
categories: ["Vue"]
tags: ["Vue"]
author: ["lavaicer"]
# author: ["Me", "You"] # multiple authors
---

## 声明式渲染

Vue 的核心特性是 **声明式渲染** ：使用扩展 HTML 的模板语法，我们可以根据 JavaScript 状态描述 HTML 的外观。当状态改变时，HTML 会自动更新。

单文件组件 (SFC) 是一个可重用的自包含代码块，它封装了属于一起的 HTML、CSS 和 JavaScript，写在一个 `.vue`文件中。
可以在更改时触发更新的状态被认为是**反应性**的。我们可以使用 Vue 的 `reactive()`API 声明反应状态。创建的对象 `reactive()`是 JavaScript[代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，其工作方式与普通对象一样：

```
import { reactive } from 'vue'

const counter = reactive({
  count: 1
})

console.log(counter.count) // 0
counter.count++
```

reactive()仅适用于对象（包括数组和内置类型，如 Mapand Set）。ref()另一方面，可以采用任何值类型并创建一个对象，该对象在.value 属性下公开内部值：

```
import { ref } from 'vue'

const message = ref('Hello World!')

console.log(message.value) // "Hello World!"
message.value = 'Changed'
```

在组件块中声明的反应状态\<script setup>可以直接在模板中使用。这就是我们如何使用 mustaches 语法根据 counter 对象和 message ref 的值呈现动态文本的方式：

```
<h1>{{ message }}</h1>
<p>count is: {{ counter.count }}</p>
```

.value 注意我们在访问模板中的 ref 时不需要使用 message 它：它会自动展开以更简洁地使用。

胡须内的内容不仅限于标识符或路径——我们可以使用任何有效的 JavaScript 表达式：

```
<h1>{{ message.split('').reverse().join('') }}</h1>
```

示例：

```
<script setup>
import { reactive, ref } from 'vue'

const counter = reactive({ count: 0 })
const message = ref('Hello World!')
</script>

<template>
  <h1>{{ message }}</h1>
  <p>Count is: {{ counter.count }}</p>
</template>
```

效果：
![](https://raw.githubusercontent.com/lavaicer/Img/main/202205211931205.png)

## 属性绑定

在 Vue 中，胡须仅用于文本插值。要将属性绑定到动态值，我们使用 v-bind 指令：

```
<div v-bind:id="dynamicId"></div>
```

指令是一个以 v-前缀开头的特殊属性。它们是 Vue 模板语法的一部分。与文本插值类似，指令值是可以访问组件状态的 JavaScript 表达式。

冒号 ( :id) 之后的部分是指令的“参数”。在这里，元素的 id 属性将与 dynamicId 组件状态中的属性同步。

因为 v-bind 使用如此频繁，它有一个专用的速记语法：

```
<div :id="dynamicId"></div>
```

示例：

```
<script setup>
import { ref } from 'vue'

const titleClass = ref('title')
</script>

<template>
  <h1 :class="titleClass">Make me red</h1>
</template>

<style>
.title {
  color: red;
}
</style>
```

![](https://raw.githubusercontent.com/lavaicer/Img/main/202205231046262.png)

## 事件监听器

v-on 我们可以使用指令监听 DOM 事件：

```
<button v-on:click="increment">{{ count }}</button>
```

由于它的频繁使用，v-on 也有一个简写语法：

```
<button @click="increment">{{ count }}</button>
```

在这里，increment 引用了一个在 中声明的函数`<script setup>`：

```
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  // update component state
  count.value++
}
</script>
```

在函数内部，我们可以通过改变 refs 来更新组件状态。

事件处理程序也可以使用内联表达式，并且可以使用修饰符简化常见任务。指南 - 事件处理中介绍了这些详细信息。

现在，尝试自己实现该功能并将其绑定到使用.increment v-on

示例：

```
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">count is: {{ count }}</button>
</template>
```

![](https://raw.githubusercontent.com/lavaicer/Img/main/202205231052797.png)

## 表单绑定

使用 v-bind and v-on 一起，我们可以在表单输入元素上创建双向绑定：

```
<input :value="text" @input="onInput">
```

```
function onInput(e) {
  // a v-on handler receives the native DOM event
  // as the argument.
  text.value = e.target.value
}
```

尝试在输入框中输入 - 您应该会在输入时看到正在`<p>`更新的文本。

为了简化双向绑定，Vue 提供了一个指令，v-model，它本质上是上面的语法糖：

```
<input v-model="text">
```

v-model 自动将 `<input>`'s 的值与绑定状态同步`<input>`，因此我们不再需要为此使用事件处理程序。

v-model 不仅适用于文本输入，还适用于其他输入类型，例如复选框、单选按钮和选择下拉菜单。
示例：

```
<script setup>
import { ref } from 'vue'

const text = ref('')

function onInput(e) {
  text.value = e.target.value
}
</script>

<template>
  <input :value="text" @input="onInput" placeholder="Type here">
  <p>{{ text }}</p>
</template>
```

现在，尝试重构要使用的代码 v-model。

```
<script setup>
import { ref } from 'vue'

const text = ref('')
</script>

<template>
  <input v-model="text" placeholder="Type here">
  <p>{{ text }}</p>
</template>
```

## 条件渲染

我们可以使用该 v-if 指令有条件地渲染一个元素：

```
<h1 v-if="awesome">Vue is awesome!</h1>
```

仅当`<h1>`的值为真时 awesome 才会呈现。如果 awesome 更改为虚假值，它将从 DOM 中删除。

我们还可以使用 v-elseandv-else-if 来表示条件的其他分支：

```
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

目前演示是同时显示两个`<h1>`，按钮什么也不做。尝试向它们添加 v-if 和 v-else 指令，并实现该 toggle()方法，以便我们可以使用按钮在它们之间切换。
示例：

```
<script setup>
import { ref } from 'vue'

const awesome = ref(true)

function toggle() {
  awesome.value = !awesome.value
}
</script>

<template>
  <button @click="toggle">toggle</button>
  <h1 v-if="awesome">Vue is awesome!</h1>
  <h1 v-else>Oh no 😢</h1>
</template>
```

## 列表渲染

我们可以使用该 v-for 指令基于源数组呈现元素列表：

```
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>
</ul>
```

这 todo 是一个局部变量，表示当前正在迭代的数组元素。它只能在元素上或 v-for 元素内部访问。

请注意我们如何为每个 todo 对象赋予一个唯一的 id，并将其绑定为每个对象的特殊 key 属性`<li>`。key 允许 Vue 准确地移动每个`<li>`对象以匹配其对应对象在数组中的位置。

有两种方法可以更新列表：
在源数组上调用变异方法：

```
todos.value.push(newTodo)
```

用新数组替换数组：

```
todos.value = todos.value.filter(/* ... */)
```

这里我们有一个简单的待办事项列表 - 尝试实现其逻辑 addTodo()和 removeTodo()方法以使其工作！

```
<script setup>
import { ref } from 'vue'

// give each todo a unique id
let id = 0

const newTodo = ref('')
const todos = ref([
  { id: id++, text: 'Learn HTML' },
  { id: id++, text: 'Learn JavaScript' },
  { id: id++, text: 'Learn Vue' }
])

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.text }}
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
</template>
```

## 计算属性

让我们继续在最后一步的待办事项列表之上构建。在这里，我们已经为每个待办事项添加了切换功能。这是通过 done 向每个待办事项对象添加一个属性并使用 v-model 将其绑定到复选框来完成的：

```
<li v-for="todo in todos">
  <input type="checkbox" v-model="todo.done">
  ...
</li>
```

我们可以添加的下一个改进是能够隐藏已经完成的待办事项。我们已经有了一个切换 hideCompleted 状态的按钮。但是我们如何根据该状态呈现不同的列表项呢？

介绍 computed()。.value 我们可以创建一个基于其他响应式数据源计算其的计算 ref ：

```
import { ref, computed } from 'vue'

const hideCompleted = ref(false)
const todos = ref([
  /* ... */
])

const filteredTodos = computed(() => {
  // return filtered todos based on
  // `todos.value` & `hideCompleted.value`
})
```

```
- <li v-for="todo in todos">
+ <li v-for="todo in filteredTodos">
```

计算属性跟踪其计算中使用的其他反应状态作为依赖项。它缓存结果并在其依赖项发生更改时自动更新它。

现在，尝试添加 filteredTodos 计算属性并实现其计算逻辑！如果实施正确，在隐藏已完成项目时检查待办事项也应该立即将其隐藏。

```
<script setup>
import { ref, computed } from 'vue'

let id = 0

const newTodo = ref('')
const hideCompleted = ref(false)
const todos = ref([
  { id: id++, text: 'Learn HTML', done: true },
  { id: id++, text: 'Learn JavaScript', done: true },
  { id: id++, text: 'Learn Vue', done: false }
])

const filteredTodos = computed(() => {
  return hideCompleted.value
    ? todos.value.filter((t) => !t.done)
    : todos.value
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value, done: false })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo" />
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style>
.done {
  text-decoration: line-through;
}
</style>
```

## 声明周期和模版引用

到目前为止，Vue 已经为我们处理了所有的 DOM 更新，这要归功于反应性和声明式渲染。但是，不可避免地会出现我们需要手动使用 DOM 的情况。

我们可以使用特殊属性请求模板引用- 即对模板中元素的引用：ref

```
<p ref="p">hello</p>
```

要访问 ref，我们需要声明一个具有匹配名称的 ref：

```
const p = ref(null)
```

注意 ref 是用 null 值初始化的。这是因为执行时元素还不存在。模板 ref 只能在组件安装后访问。`<script setup>`

要在挂载后运行代码，我们可以使用以下 onMounted()函数：

```
import { onMounted } from 'vue'

onMounted(() => {
  // component is now mounted.
})
```

这称为生命周期钩子——它允许我们注册一个回调，以便在组件生命周期的特定时间调用。还有其他钩子，例如和。查看生命周期图了解更多详情。`onUpdated onUnmounted`

现在，尝试添加一个钩子，访问 via 并对其执行一些直接的 DOM 操作（例如更改它的）。`onMounted <p> p.valuetextContent`

```
<script setup>
import { ref, onMounted } from 'vue'

const p = ref(null)

onMounted(() => {
  p.value.textContent = 'mounted!'
})
</script>

<template>
  <p ref="p">hello</p>
</template>
```

## 观察者

有时我们可能需要反应性地执行“副作用”——例如，当一个数字发生变化时将其记录到控制台。我们可以通过观察者实现这一点：

```
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newCount) => {
  // yes, console.log() is a side effect
  console.log(`new count is: ${newCount}`)
})
```

watch()可以直接观察 ref，并且只要 count' 的值发生变化，就会触发回调。watch()还可以观察其他类型的数据源 - Guide-Watchers 中介绍了更多详细信息。

一个比登录控制台更实际的例子是在 ID 更改时获取新数据。我们的代码是从组件挂载的模拟 API 中获取 todos 数据。还有一个按钮可以增加应获取的待办事项 ID。尝试实现一个观察者，在单击按钮时获取新的待办事项。
示例：

```
<script setup>
import { ref, watch } from 'vue'

const todoId = ref(1)
const todoData = ref(null)

async function fetchData() {
  todoData.value = null
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  todoData.value = await res.json()
}

fetchData()

watch(todoId, fetchData)
</script>

<template>
  <p>Todo id: {{ todoId }}</p>
  <button @click="todoId++">Fetch next todo</button>
  <p v-if="!todoData">Loading...</p>
  <pre v-else>{{ todoData }}</pre>
</template>
```

![](https://raw.githubusercontent.com/lavaicer/Img/main/202205231446210.png)

## 组件

到目前为止，我们只使用了一个组件。真正的 Vue 应用程序通常使用嵌套组件创建。

父组件可以将其模板中的另一个组件渲染为子组件。要使用子组件，我们需要先导入它：

```
import ChildComp from './ChildComp.vue'
```

然后，我们可以将模板中的组件用作：

```
<ChildComp />
```

示例：

```
<script setup>
import ChildComp from './ChildComp.vue'
</script>

<template>
  <ChildComp />
</template>
```

## props

子组件可以通过 props 接受来自父组件的输入。首先，它需要声明它接受的道具：

```
<!-- ChildComp.vue -->
<script setup>
const props = defineProps({
  msg: String
})
</script>
```

注意 defineProps()是一个编译时宏，不需要导入。一旦声明，该 msg 道具就可以在子组件的模板中使用。它也可以在 JavaScript 中通过 defineProps().

父母可以像属性一样将道具传递给孩子。要传递动态值，我们还可以使用以下 v-bind 语法：

```
<ChildComp :msg="greeting" />
```

示例：

```
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const greeting = ref('Hello from parent')
</script>

<template>
  <ChildComp :msg="greeting" />
</template>
```

## Emits

除了接收 props 之外，子组件还可以向父组件发出事件：

```
<script setup>
// declare emitted events
const emit = defineEmits(['response'])

// emit with argument
emit('response', 'hello from child')
</script>
```

第一个参数是事件名称。任何其他参数都将传递给事件侦听器。emit()

父级可以使用 - 监听子级发出的事件 v-on- 这里处理程序从子级发出调用接收额外的参数并将其分配给本地状态：

```
<ChildComp @response="(msg) => childMsg = msg" />
```

示例：

```
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const childMsg = ref('No child msg yet')
</script>

<template>
  <ChildComp @response="(msg) => childMsg = msg" />
  <p>{{ childMsg }}</p>
</template>
```

ChildComp.vue:

```
<script setup>
const emit = defineEmits(['response'])

emit('response', 'hello from child')
</script>

<template>
  <h2>Child component</h2>
</template>
```

## Slots

除了通过 props 传递数据之外，父组件还可以通过 slot 将模板片段传递给子组件：

```
<ChildComp>
  This is some slot content!
</ChildComp>
```

在子组件中，它可以使用`<slot>`元素作为出口从父组件渲染插槽内容：

```
<!-- in child template -->
<slot/>
```

出口内的内容`<slot>`将被视为“后备”内容：如果父级没有传递任何插槽内容，它将显示：

```
<slot>Fallback content</slot>
```

目前我们没有将任何插槽内容传递给`<ChildComp>`，因此您应该会看到后备内容。让我们在利用父 msg 级状态的同时为子级提供一些插槽内容。
示例：

```
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const msg = ref('from parent')
</script>

<template>
  <ChildComp>Message: {{ msg }}</ChildComp>
</template>
```

终示例：

```
<script setup>
import JSConfetti from 'js-confetti'

const confetti = new JSConfetti()

function showConfetti() {
  confetti.addConfetti()
}

showConfetti()
</script>

<template>
  <h1 @click="showConfetti">🎉 Congratulations!</h1>
</template>

<style>
h1 {
  text-align: center;
  cursor: pointer;
  margin-top: 3em;
}
</style>
```
