# Vue常见指令

## 一、指令系统介绍

1.`vue`的指令系统是`vue`响应式重要的部分，是带有 `v-` 前缀的特殊 `attribute`。

2.指令 `attribute` 的值预期是单个 `JavaScript` [表达式](https://so.csdn.net/so/search?q=表达式&spm=1001.2101.3001.7020)，`v-for`除外。

## 二、常用指令

1.`v-bind`: 为元素属性传递动态参数。

2.`v-on`: 绑定事件

3.`v-for`：列表渲染

4.`v-if v-else v-else-if`: 条件渲染

5.`v-show,v-if` : 控制显隐

6.`v-model` :  [表单](https://so.csdn.net/so/search?q=表单&spm=1001.2101.3001.7020)元素的双向绑定

7.`v-once`: 性能优化

8.`v-slot`: 插槽

## 三、指令缩写

1.`v-bind` 可以缩写为 `:`

2.`v-on` 可以缩写为`@`

3.`v-slot`可以缩写为`#`

## 四、具体实例

####  **条件渲染：`v-if`**

1.`v-if`指令，可以根据条件判断元素渲染与否。

2.**`v-if`接受一个具有`truth`值的表达式，当为`true`时元素渲染，反之元素不渲染**。

3.相应的还有`v-else`和`v-else-if`，当`v-if`为`false`时，渲染`v-else`内容。

4.注意，`v-else`必须和和`v-if`相邻，否则识别不了。

```html
<div v-if="show">show为true时展示</div>
<div v-else>show为false时展示</div>
```

####  **条件渲染：`v-show`**

1.`v-show`也可以用来动态渲染元素。

2.`v-if`会销毁元素，代价比较大，`v-show`只是在`css`的`display`上控制元素的显隐，代价较小，若要频繁控制显隐时使用`v-show`。

3.即使`v-show`为`false`，元素也是会渲染的

```html
<div v-show="show">show为true时展示</div>
```

####  **数组列表渲染：`v-for`**

1.`v-for`指令用于遍历数组或对象来渲染元素

2.**遍历数组**:`v-for="(item,index) of list"`

3.其中使用关键字`in`或者关键字`of`是等价，对于遍历数组`of`更符合`js`语法习惯

4.`item`是数组元素的别名，`index`是数组的索引，`list`是被遍历的数组。

```html
<div v-for="(item,idx) of arr" :key="item.id">{{item.name}}</div>
```

####  **对象列表渲染：`v-for`**

1.**遍历对象**:`v-for="(value,key,index) in object"`

2.其中`value`是对象的值，`key`是对象的键，`index`是遍历的索引

3.在使用`v-for`遍历时，最好给每个元素添加一个`key`属性，绑定独一无二的值来提高性能。

```html
<div v-for="(value,key,idx) of obj" :key="item.key">{{key}}:{{value}}</div>
```

#### **事件绑定：`v-on`**

1.所有标签支持的事件，都可以用`v-on`绑定，例如：`v-on:click`、`v-on:dbldick`、`v-on:change`等等。

2.`v-on`可以使用符号`@`来简写

```markup
<button @sumbit="sengMsg"></button>
```

3.我们在`Vue Option Api`的`methods`选项中定义事件处理函数。

```javascript
methods:{
	sengMsg(e){
		console.log(e) //原生事件对象
	},
}
```

4.除了上述的方法，我们可以使用**内联处理器**的方法，在绑定事件时调用事件函数传递参数。

```html
<button @sumbit="sengMsg('hi')"></button> 
```

5.使用内联方法时，要想获得原生事件对象，需要主动添加`$event`

```html
<button @sumbit="sengMsg($event,'hi')"></button> 
1
methods:{
	sengMsg(e,arg1){
		console.log(arg1) // hi
		console.log(e) //原生事件对象
	},
12345
```

**`v-on`绑定多个事件**

1.**`v-on`可以绑定多个事件，接受一个对象**，对象的键是事件类型，对象的值是事件处理函数

```html
<a v-on="{click:clickMethod,mouseover:mouseover}"></a>
```

2.`v-on`事件还可以绑定多个处理函数，函数按顺序执行。

```html
<a v-on:click='clickmethod1(),cliclmethod2()'></a>
```

**组件上绑定原生事件**

1.要想在**自定义的组件**上绑定一个原生事件，直接绑定是无效的，需要使用`.native`修饰符。

```javascript
<component v-on:focus.native='fn'/>
```

**事件修饰符**

1.`vue`的理念是：方法中只包含纯粹的数据逻辑，而不是去处理 `DOM` 事件细节。因此`vue`框架为我们提供了**事件修饰符**，用于处理`DOM`事件细节。

2.常用事件修饰符

- `.stop` 阻止事件继续传播，也就可以达到`event.preventDefault()`的作用

```markup
<p @click.stop="count+=1">{{count}}</p>
```

- `.once`事件只会触发一次。

```markup
<p @click.once="count+=1">{{count}}</p>
```

- `.self`事件由自身触发，而不是通过内部传出。

```markup
<p @mouseover.self="count+=1">{{count}}</p>
```