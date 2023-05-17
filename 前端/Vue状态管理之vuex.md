

# vue之vuex

## 一、前言

1.`Vuex`是专门为`Vue`设计的状态管理库，**集中管理所有组件的状态**。**并与相应的规则保证状态以一种可预测的方式**发生变化。

2.`Vuex`解决了多个组件共享状态时，传参方式繁琐，代码维护困难的问题。

## 二、`Vuex`入口

1.**`Vuex`的入口是`store`**，`store`是`Vuex`核心概念，像一个容器仓库集中管理了所有组件的`state`。

2.在`Vuex4`中，**我们使用`createStore` 初始化一个`store`**

3.`createStore`接受一个对象作为参数，对象中定义了组件的`state`和改变`state`的规则

```javascript
import { createApp } from "vue";
import { createStore } from 'vuex';
const store = createStore({...option})
const app = createApp(/*单文件组件*/)
app.use(store )
```

4.`Vuex` 使用单一状态树，用一个对象就包含了全部的应用层级状，这也意味着，**每个应用将仅仅包含一个 `store` 实例**

```javascript
import { createApp } from "vue";
import {createStore} from 'vuex';
const store = createStore({...option})
const store1 = createStore({...option})
const app = createApp(/*单文件组件*/)
app.use(store)
app.use(store1) //当我们初始化两个store时，会根据代码执行顺序保留最后的，其余的无效
```

## 三、`store`介绍

1.一个完整的`store`包含

（1）状态：`state`
（2）计算属性：`getters`
（3）同步修改：`mutations`
（4）异步修改：`actions`
（5）模块化：`modules`

2.当我们全局注册了`store`后

```javascript
app.use(store)
```

（1）在`vue2`中，我们通过组件实例的`$store`属性访问`store`仓库

```javascript
this.$store
```

（2）在`vue3`中，使用组合式`api`

```javascript
import { useStore } from 'vuex'
const store = useStore()   //useStore()等同于vue2中的$store
```

## 四、状态：`state`

1.`state`用于声明组件的状态。

2.我们使用函数形式来定义`state`

```javascript
const store = createStore({
	state(){
		return {
			name:'Mr.poto',
			age:'20',
		}
	},
})
```

3.`Vuex` 的状态存储是[响应式](https://so.csdn.net/so/search?q=响应式&spm=1001.2101.3001.7020)的，从 `store` 实例中读取状态最简单的方法就是在计算属性中返回某个状态。

（1）在组件中使用`vuex`中`state`数据（`vue3`）

```html
<script setup>
	import { computed } from 'vue';
	import { useStore } from 'vuex';
	const store = useStore()
	const age =computed(()=>store.state.age)
	const name=computed(()=>store.state.name)
</script>
```

（2）在组件中使用`vuex`中`state`数据（`vue2`）

```html
<script>
	export default {
		computed:{
			age:()=>this.$store.state.name
		}
	}
</script>
```

4.虽然通常我们使用`computed`来处理这个响应式，但是直接将状态写进视图也是可以的。

5.在`vue2`中当使用状态较多时可以使用辅助函数`mapState`简写，

```html
<script>
	import { mapState } from 'vuex'
	export default {
		computed:{
			age:()=>this.$store.state.name，
			...mapState({
				count: state => state.count,
				aliasCount:'count',
			    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    			countPlusLocalState (state) {
     			 return state.count + this.localCount
   			 }
			})
		}
	}
</script>
```

6.在`vue3`中暂不支持`vue2`中辅助函数的写法

## 四、派生状态：`getters`

1.有时我们需要使用状态的派生状态，当只有一个组件需要使用时，我们可以使用计算属性，但如果有多个组件需要用到此属性，如果还使用组件的计算属性，我们就要一直复制这个函数。`getter`好像`store`的计算属性，我们只需定义一次即可。

2.**定义`getter`**:

```javascript
const store = createStore({
	getters:{
		info:(state)=>state.age + state.name //接受 state 作为其第一个参数
		moreinfo:(state,getters)=>state.age + state.name + getters.info//接受 getter 作为其第二个参数
		setName:(state)=>((p)=>state.name + p) //可以通过返回一个函数来接受自定义参数
	},
})
```

3.**使用`getters`**:

（1）使用属性的形式

```html
<script>
	export default {
		computed:{
			info:()=>this.$store.getters.info
		}
	}
</script>
```

（2）使用方法的形式

```html
<script>
	export default {
		computed:{
			name:()=>this.$store.getters.setName('zjl')
		}
	}
</script>
```

4.当使用的`getters`较多时可以使用辅助函数`mapGetters`简写，

```html
<script>
	import { mapGetters } from 'vuex'
	export default {
		computed:{
			age:()=>this.$store.state.name，
			...mapState({
				count: state => state.count,
				aliasCount:'count',
			    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    			countPlusLocalState (state) {
     			 return state.count + this.localCount
   			 }
			}),
			...mapGetters (["info","setName"])
		}
	}
</script>
```

## 五、修改状态：`mutations`

1.**`mutations`是用于定义修改`state`的函数，且是唯一能修改`state`的地方**，

2.`mutations`中的函数必须是同步的。

3.`mutations`函数第一个参数是`state`，第二个参数是载荷`payload`，无返回值，修改`state`时，`state`地址不能变。

```javascript
const store = createStore({
	state:{
		count:1,
	},
	mutations:{
		add(state){ //定义count自增的规则
			state.count=count+1 //修改时state地址不能变，否则无法相应，例如state={...state,a:111}不行
		}
		addByParams(state,payload){ //定义count根据传入参数增加的规则
			state.count =state.count+payload.num
		}
	},
})
```

4.我们使用`store`实例的`commit`方法来触发`mutations`

5.在`commit`里我们需要通过`type`指明触发那个`mutations`，通过`payload`传递参数，有如下两种形式

```javascript
store.commit("add",{count:1})
store.commit({type:"add",payload:{num:2}) 
```

6.在`vue2`中，我们可以使用`mapMutations` 辅助函数将组件中的 `methods` 映射为 `store.commit` 调用

```javascript
import { mapMutations } from 'vuex'

export default {
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

## 六、异步操作：`actions`

1.**`actions`支持异步操作**。

2.**在`actions`里，我们以函数形式定义需要异步操作的业务**，如需要在异步操作后修改 `state`、执行下一个`actions`等等

3.`actions`函数接受两个参数

（1）第一个参数接受一个与 `store` 实例具有相同方法和属性的 `context` 对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/573ba0103dc84e05837ba5565feab437.png)
（2）第二个参数是提交荷载。

```javascript
const store = createStore({
	state:{
		count:1,
	},
	actions:{
		add(context,payload){ 
			setTimeout(()=>{
				context.commit(...)
			},100)
		},
		add({commit,dispatch,state},payload){ //可以将context解构出来
				setTimeout(()=>{
				commit(...)
			},100)
		}
	},
})
```

4.**`actions`里的函数调用需要使用`store`实例的`dispatch`方法**，`type`表明要使用的具体`actions`，`payload`是传递的参数

```javascript
this.$store.dispatch({type:"add",payload:{num:1}})
```

5.在`vue2`中，我们可以使用`mapActions` 辅助函数将组件中的 `methods` 映射为 `store.dispatch` 调用

```javascript
import { mapActions} from 'vuex'

export default {
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

6.当我们在`actions`里使用`async`函数或者返回`promise`对象时，就可以组合`actions`

```javascript
actions: {
  async actionA ({ commit }) {
    commit('gotData',)
  },

this.$store.dispatch("actionA").then()
```

## 六、模块化：`module`

1.由于`store`是单例的，当项目较大时，所有`state`写在一起会不够清晰又庞大，为了解决以上问题，`Vuex` 允许我们将 `store` 分割成模块。

```javascript
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

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

2.模块内部的 `action` 和 `mutation` 仍然是注册在全局命名空间的——这样使得多个模块能够对同一个 `action` 或 `mutation` 作出响应。`Getter` 同样也默认注册在全局命名空间

3.如果希望你的模块具有更高的封装度和复用性，你可以通过添加 `namespaced: true` 的方式使其成为带命名空间的模块,**同时在访问时也要加上相应的路径**。

4.注意，不管以任何方式分割了多个模块，都只有一个`store`实例，且各个模块一定可以相互访问的

## 七、完整数据流

1.一个完整的数据流如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/7daa31ea86a444dbb211929e23047799.png)