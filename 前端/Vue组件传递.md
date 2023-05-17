# Vue组件传递

## 1. 父传子（props）

#### **一、前言**

1. `props`用于父子组件的通信。

2. `props`具有单向下行绑定，由父组件传递给子组件。

#### **二、子组件`props`声明**

1. `props`是`Vue Option Api`的一个选项，声明该组件接受父组件传递的`props`，可接受数组和对象。

（1）数组：`props: ['title','content']` 

（2）对象：`props: { title:Number }`

2. 使用对象形式可以对`props`进行 **值类型校验** 和  **设置默认值**。

（1）**类型校验**：

```vue
{
	props:{
		title:String, //单类型  
		title:[Number,Boolean] //多类型
		title:{ type:String } //注意所有类型不能加引号
	}
}
```

（2）**必填校验**

```vue
{
	props:{
		title:{ required:true } 
	}
}
```

（3）**自定义验证函数： 函数返回布尔值，为true验证通过**

```vue
{
	props:{
		title:{ validator:(v)=>typeof v =='number'} 
	}
}
```


（4）**设置默认值：注意对象或数组默认值必须从一个工厂函数获取**

```vue
{
	props:{
		title:{ default:"hello world"} 
		list:{ type:Array, default:()=>[1,2,3]} //对象或数组默认值必须从一个工厂函数获取
	}
}
```

3. 子组件不能修改`props`，若需要修改`props`的值，可以把`props`作为初始值赋值给组件自己的data

4. 在一个组件中，`props`可以像data一样被使用，除了不能被修改。

#### **三、父组件props传递**

1. 子组件使用驼峰命名的`props`,父组件传递给子组件时需要使用-分割 ： dogName -> dog-name。

2. 为了方便书写，当子组件接受的props较多，可以使用一个对象全部传入，此时直接使用v-bind='objName'即可。 （类似于react的{...props}）

```vue
let post ={a,b}
<Post v-bind="post"/>
等价于
<Post v-bind:a="post.a" v-bind:b="post.b" />
```

#### **四、非props的属性**

1. 非`props`的属性是指父组件传递给子组件，而子组件并未声明相应的`props`的属性

2. 默认情况下，非`props`的属性会被直接加在子组件的根节点上。

3. 例如在子组件上定义`style`，`class`,或者事件会被直接加在根元素上。

4. `inheritAttrs`是配置对象的一个属性，我们可以在子组件使用`inheritAttrs:false`来禁用这种默认行为

#### **五、注意事项**

1. 由于在子组件不能修改`props`，所以不能直接把`props`作为`v-model`的值，
   

## 2. 子传父（$emit）

#### **一、前言**

1. `props`和`$attrs`具有数据的单向性，只能由父组件向子组件传递数据，不具备子传父的功能。

2. 在vue中，我们可以使用自定义事件实现子组件向父组件传递数据。

#### **二、自定义事件**

1. 在子组件上使用`v-on`指令绑定自定义事件

```vue
getChildData(data){
	//data是子组件触发事件传递的参数
	console.log('child data is' +data)
}
<child-c v-on:get-child-data='getChildData'></child-c>
```

2. 事件的名称推荐使用-分割

#### **三、$emit触发事件**

1. 每一个组件实例都要`$emit`方法用来触发自定义事件。

2. `$emit(fn,arg)`接受两个参数

（1）第一个参数是触发的事件名称，注意必须和定义的事件名完全相同。

（2）第二个参数是向父组件传递的数据。在父组件中就是定义的事件函数的第一个参数。

```vue
this.$emit('get-child-data','hello father')
```

## 3. 爷传孙（$attrs）

#### **一、前言**

1.在vue中我们可以使用`props`组件实现父子组件的单向数据传递。

2.然而当组件层级较多时，第一层级的组件要向第三层级的组件传递数据时，使用`props`一级一级传递，明显是不高效，且比较麻烦的。

3.vue中的`$attrs`可以实现组件的跨级数据传送。

#### **二、$attrs**

1.在`vue2`中，`$attrs`是组件的一个实例属性，包含了父作用域中 不作为 `prop` 被识别的属性。

```vue
// todoList定义了list,和name两个props属性，
const todoList={
	props:[list,name] 
	template:`<div></div>`
}

//当接受到这两个之外的属性，都会作为$attrs属性
<todo-list class='a' />

//组件实例的this.$attrs 会返回 {class:'a'}
```

2. 在数据结构上，$attrs是一个对象，键是所有绑定的非props属性，值是相应非props属性的值，

3. 我们可以将整个$attrs传递给组件内部的任一个组件，接受$attrs的组件一定要声明props来接收

```vue
<target v-bind="$attrs"></target> 

//对于target 组件，接受$attrs时需使用props声明$attrs里的每一个属性
```

4. 本质上爷传孙也是一级一级的传，使用`$attrs`可以避免中间组件的`props`声明

#### **三、注意事项**

1. **在`vue`中组件非`props`的属性会默认传递给绑定组件的根元素**。我们可以使用`inheritAttrs：false`来禁止这种行为。

## 4. 兄弟组件（事件总线）

#### **一、前言**

1. 当组件之间层级不相近时，或者层级涉及的较多的时候，普通的组件数据传递方式就会显得比较无力。

2. vue2中提供了事件总线的方式，通过事件的发送和事件的监听可以做到上下平行数据传送。

#### **二、事件总线使用**

1. 初始化
   （1）第一种方式新建一个.js文件导出Vue实例，作为桥梁，可以不需要有任何的配置项。

    ```vue
    export default const eventBus =new Vue()
    ```

	（2）第二种方式在vue入口文件全局初始化一个事件总线：`Vue.prototype.$EventBus = new Vue()`

2. 事件发送
   （1）`eventBus.$emit('eventName',data)` 通过实例方法`$emit`触发一个事件，一个参数是触发事件的名称，第二个是要发送的数据。

3. 事件监听
   （1）`eventBus.$on('eventName',(data)=>{})` 通过实例方法`$on`监听一个事件，一个参数是监听的事件 名称，第二个参数是一个回调函数，函数参数是事件发送的数据

4. 事件移除
   （1）`eventBus.off('eventName')` 使用$off实例方法可以想要移除的事件，当不传事件名称时，移除所有事件。

#### **三、注意事项**

1. 如果不通过全局方式注册，所有使用事件总线的js文件都需手动引入事件总线文件。

1. 当`$emit`一个事件时，所有监听的`$on`都会执行

#### **四、vue3中事件总线**

1. 在`vue3`中，创建方法改成了`createApp({})`，`prototype`属性也被取消了，因此无法使用之前`Vue.prototype.$bus = new Vue()`的方式使用事务总线。

2. 且在`vue3`中移除了`$on,$off`方法，因此官方推荐：事件总线模式可以被替换为使用外部的、实现了事件触发器接口的库，例如 `mitt` 或 `tiny-emitter`。

3. 首先安装`mitt：npm install --save mitt`

4. 在代码中使用：

   ```vue
   import mitt from "mitt"
   import {createApp} from "vue"
   
   const app = createApp({})//正常配置
   //挂载事务总线
   app.config.globalProperties.$bus = new mitt()
   
   //在组件A中使用事务总线触发某个动作
   this.$bus.emit("EVENTTYPE");
   
   //在组件B中监听动作的发生
   this.$bus.on("EVENTTYPE",()=>{
       console.log("EVENTTYPE发生了")})
   ```

   