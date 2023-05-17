# Vue组件

## 1.  **内置组件：component**

`<component/>`是一个用于渲染[动态组件](https://so.csdn.net/so/search?q=动态组件&spm=1001.2101.3001.7020)或元素的“元组件”。

### `<component/>`使用

1.`<component/>`功能类似于`tab`组件，可用于组件的切换

2.**`is`属性决定了`component`当前渲染的组件**，`is`属性可以是组件或者是字符串，当是字符串时代表组件的注册名称或者是标签名

```javascript
interface DynamicComponentProps {
  is: string | Component
}
123
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Math.random() > 0.5 ? Foo : Bar" />
</template>
```

3.**`component`动态组件上可以传任意参数和事件，且会被所有`is`上的当前组件所接收**

4.简而言之，`component`就像一个容器，根据`is`属性来决定渲染什么组件

## 2.  **内置组件：keepalive**

在`vue`中我们可以使用`component`内置组件根据`is`属性来切换组，`keepalive`的功能就是在多个组件间动态切换时**缓存被移除的组件实例**

###  `keepalive`使用

1.默认情况下，一个组件实例在被替换掉后会被销毁，这会导致它丢失其中所有已变化的状态，可以**使用`keepalive`组件使被移除的组件不被销毁，而是处于失活状态**

```html
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```

2.我们可以**通过`include`和`exclude`属性来定制某个组件是否开启默认缓存**

（1）字符串形式，内容为组件名，多个组件用`,`分割

```html
<KeepAlive include='a,b'>
  <component :is="activeComponent" />
</KeepAlive>
```

（2）数组形式

```html
<KeepAlive :include="['a','b']">
  <component :is="activeComponent" />
</KeepAlive>
```

（3）正则形式

```html
<KeepAlive :include="/a|b/">
  <component :is="activeComponent" />
</KeepAlive>
```

3.我们可以**通过`max` 属性来限制可被缓存的最大组件实例数**，超过最大属性时，**最久**没被访问的实例将会被销毁

```html
<KeepAlive :max="6">
  <component :is="activeComponent" />
</KeepAlive>
```

### 缓存组件的生命周期 

1.一个被缓存的组件在被移除`dom`时会执行`onDeactivated()`钩子，在被重新插入`dom`时执行`onActivated()`

2.需要注意的是`onActivated`和`onDeactivated`在挂载和卸载时也会执行


## 3.  **插槽slot**

`vue`插槽`slot`用于组件内容分发，是一个特殊的标签。

###  `<slot/>`使用

1.在`vue`中，插槽可以分为**默认插槽**和**具名插槽**两种

（1）**默认插槽**：`<slot></slot>`，没有名字映射时，内容默认分发的地方

```javascript
// child.vue 组件
<template>
	<div>
		<div>title</div>
		<slot></slot> //内容默认位置
	</div>
</template>
```

（2）**具名插槽**：`<slot name='xxx'></slot>`，可以通过具体的名字映射，将内容分发到指定位置

```javascript
// child.vue组件
<template>
	<div>
		<div>title</div>
		<slot name="content"></slot> //定义主体内容位置
		<slot name="footer"></slot> //定义底部内容位置
	</div>
</template>
```

2.当我们在组件内部定义了插槽后，使用这个组件时，可能没有给插槽分发内容，我么可以通过**插槽后备内容**定义默认渲染的内容，**插槽后备内容在`slot`标签内部定义**。

```javascript
<template>
	<div>
		<div>title</div>
		<slot>如果没有默认插槽值，显示这段话</slot>
		<slot name="footer">如果没有定义footer插槽值，我是默认footer</slot> 
	</div>
</template>
```

#### **分发插槽内容**

1.**向默认插槽内容分发内容**：默认情况下，[自定义组件](https://so.csdn.net/so/search?q=自定义组件&spm=1001.2101.3001.7020)的子元素被分发到默认插槽

```javascript
// 组件
<template>
	<child>默认插槽内容</child>
</template>
```

2.**向具名插槽内容**：通过`template`包裹，和具体的`name`，像指定位置分发内容

```javascript
// 组件
<template>
	<test>	
		<template v-slot:content>hello world</template>  
		//或
		<template #content>hello world</template>  //具名插槽内容，会被分发到组件的相应位置
		
		<template #footer>footer content</template>
	</test>
</template>
```

#### **插槽作用域**

1.当我们在组件内部使用`slot`标签定义插槽时，可以给`slot`标签绑定属性，在父组件上使用该组件时，通过插槽作用域的作用可以获取子组件作用域上的值，

```javascript
//test组件
<template>
	<div><slot :childData="12345" :childData2='6789'></slot></div> //将数据定义在slot标签上 
</template>

//父组件
<template>
	<test>
		<template #default="slotProps"> //获取子组件作用域插槽内容
			{{slotProps.childData}}  // 显示12345
		</template>
	</test>
</template>
//或
<template>
	<test>
		<template v-slot:default="slotProps"> //获取子组件作用域插槽内容
			{{slotProps.childData}}  // 显示12345
		</template>
	</test>
</template>

<template>
	<test>
		<template #default="{ childData,childData2 }"> //获取子组件作用域插槽内容
			{{childData}}  // 显示12345
		</template>
	</test>
</template>

```

2.如上面代码所示，**`slotProps`是子组件作用域上的值，不是按常理理解的父组件作用域上的值**，`slotProps`是一个对象可解构

#### **注意事项**

1.模板上绑定的响应式数据变化时，组件会执行更新生命周期，对于这个理论，我们通常认为是正确的。 然而当使用上组件的插槽，情况就发生变化了，**对于父组件，插槽内容模板上绑定的数据变化，是不会触发父组件的更新生命周期的。**

```javascript
<template>
  <child @click="handleClick">
    {{ num.a }}
  </child>
</template>
<script setup>
import { reactive, onUpdated } from "vue";

const num = reactive({
  a: 1,
});

const handleClick = () => {
  num.a = num.a + 1;
};

onUpdated(() => {
  //即使点击了，也不会触发，因为num.a被绑定在插槽内
  console.log("--update");
});
</script>
```

2.`v-slot:default` 可以缩写成`v-slot`，`v-slot:`可以缩写成`#`

```javascript
//默认插槽的三种写法
<div v-slot:default ></div>  = <div v-slot></div> = <div #default></div>
//具名插槽的三种写法
<div v-slot:header ></div>  = <div #header></div> = <div slot="header"></div>(废弃)
//插槽作用域的三种写法
<div v-slot:header="scopePorps" ></div>   = <div #header = "scopePorps" ></div> =  <div slot-scope = "scopePorps" ></div>(废弃)
```