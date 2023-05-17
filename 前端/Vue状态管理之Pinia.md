# vue之Pinia

### 一、前言

​    Pinia 是 Vue 的存储库，它允许跨组件/页面共享状态。实际上，pinia就是Vuex的升级版，官网也说过，为了尊重原作者，所以取名pinia，而没有取名Vuex，所以可以直接将pinia比作为[Vue3](https://so.csdn.net/so/search?q=Vue3&spm=1001.2101.3001.7020)的Vuex。

### 二、为什么要使用pinia？

- pinia中只有state、getter、action，抛弃了Vuex中的Mutation，Vuex中mutation一直都不太受小伙伴们的待见，pinia直接抛弃它了，这无疑减少了我们工作量。
- pinia中action支持同步和异步，Vuex不支持
- 良好的[Typescript](https://so.csdn.net/so/search?q=Typescript&spm=1001.2101.3001.7020)支持，毕竟Vue3都推荐使用TS来编写，这个时候使用pinia就非常合适了
- 无需再创建各个模块嵌套了，Vuex中如果数据过多，通常分模块来进行管理，稍显麻烦，而pinia中每个store都是独立的，互相不影响。
- 体积非常小，只有1KB左右。
- pinia支持插件来扩展自身功能。
- 支持服务端渲染。

### 三、Pinia 基础特性

  **1、State**

（1）默认情况下，通过 store 实例访问 state，可以直接读取和写入，如 @click="store.count++"

（2）通过 store.$reset() 方法可以将 state 重置为初始值。

（3）除了直接通过 store 修改 state，还可以通过 store.$patch() 方法提交多个更改。

（4）可以通过 store.$subscribe() 订阅 State 的变化，在 patches 修改之后订阅只会触发一次。默认情况下，订阅绑定到添加它的

组件，当组件卸载时，它们将自动删除，也可以配置将其保留。

**2、Getters**

（1）Getters 属性的值是一个函数，接受 state 作为第一个参数，目的是鼓励使用箭头函数

（2）非箭头函数会绑定 this，建议仅在需要获取整个 store 实例的场景使用，且需要显式定义函数返回类型

**3、Actions**

（1）与 Gettes 一样可以通过 this 访问整个 store 实例

（2）Actions 可以是异步的或同步的，不管怎样，都会返回一个 Promise

（3）Actions 可以自由的设置参数和返回的内容，一切将自动推断，不需要定义 TS 类型

（4）与 State 一样，可以通过 store.$onAction() 订阅 Actions，回调将在执行前触发，并可以通过参数 after() 和 onError() 允许在Action 决议后和拒绝后执行函数。同样的，订阅绑定的是当前组件。

### 四、Pinia 中关于 TypeScript 类型推断。Pinia 默认提供良好的 TypeScript 支持，但是要想获得完整的类型推断，需要遵循一些使用建议：

   **1、声明 `state` 建议使用箭头函数，会自动推断属性的类型。如果使用非箭头函数，在 `getters` 中使用形参 `state` 访问属性TypeScript 无法识别**

```javascript
const useStore = defineStore('main', {
  // 使用非箭头函数定义
  state() {
		return {
      count: 0
    }
  },
  getters: {
    // 使用 state 访问状态
    doubleCount(state) { 
      return state.count * 2
    }
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

​    **2、声明 Getters 属性建议使用 `state` 参数访问状态（箭头函数或非箭头函数），非箭头函数虽然可以使用 `this` ，但不会自动推断类型（当没有声明 `state` 形参，或 state 使用非箭头函数定义时）**

- 为了避免使用 `this`，官方建议使用箭头函数
- 建议仅当需要获取整个 `store` 时使用 `this`，但必须显式的定义函数返回类型，TypeScript 不会自动推断
- 建议使用 `this` 的时候不要声明 `state` 形参

```javascript
const useStore = defineStore('main', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount() {
      return this.count * 2
    },
    // 解决办法1
    doubleCount2(state) {
      return this.count * 2
    },
    // 解决办法2
    doubleCount3():number {
      return this.count * 2
    }
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

### 五、Pinia的函数

  1、createPinia(): Pinia。创建一个被应用所使用的 Pinia 实例。返回：Pinia

  2、defineStore(id, options): StoreDefinition。创建一个检索 `store` 实例的 `useStore` 函数。

参数

| 名称      | 类型                                                         | 描述                           |
| :-------- | :----------------------------------------------------------- | :----------------------------- |
| `id`      | `Id`                                                         | `store` 的 id （必须是唯一的） |
| `options` | `Omit`<[DefineStoreOptions](https://pinia.vuejs.org/api/interfaces/pinia.DefineStoreOptions.html)<`Id`, `S`, `G`, `A`>, `"id"`> | 定义 `store` 的选项            |

   3、mapActions(useStore, keyMapper): _MapActionsObjectReturn。通过生成要在组件的 `methods` 字段中铺开的对象，允许直接使用存储（`store`）中的操作（`action`），而不使用合成API (`setup()`)。对象的值是操作（`action`），而键是结果方法的名称。

参数

| 名称        | 类型                                                         | 描述                          |
| :---------- | :----------------------------------------------------------- | :---------------------------- |
| `useStore`  | [StoreDefinition](https://pinia.vuejs.org/api/interfaces/pinia.StoreDefinition.html)<`Id`, `S`, `G`, `A`> | 要映射的存储                  |
| `keyMapper` | `KeyMapper`                                                  | 为 `actions` 定义新名称的对象 |



 例如：

```javascript
export default {
  methods: {
    // 其他方法属性
    // useCounterStore 有两个操作（action），分别叫做 `increment` 和 `setCount`
    ...mapActions(useCounterStore, { moar: 'increment', setIt: 'setCount' })
  },
  created() {
    this.moar()
    this.setIt(2)
  }
}
```

   4、mapState(useStore, keyMapper): _MapStateObjectReturn。通过生成要在组件的 `computed` 字段中展开的对象，以允许从一个没有使用组合式 API (`setup()`) 的 `store` 使用 `state` 和 `getters` 对象的值是状态 properties/getters ，而键是结果计算属性的名称。您也可以传递一个自定义函数，该函数将接收 `store` 作为其第一个参数。请注意，虽然它可以通过这个访问组件实例，但它不会识别类型。

参数

| 名称        | 类型                                                         | 描述                                      |
| :---------- | :----------------------------------------------------------- | :---------------------------------------- |
| `useStore`  | [StoreDefinition](https://pinia.vuejs.org/api/interfaces/pinia.StoreDefinition.html) | 要映射的存储                              |
| `keyMapper` | `KeyMapper`                                                  | 状态（state）属性或访问器（getter）的对象 |

例如：

```javascript
export default {
  computed: {
    // 其它计算属性
    // useCounterStore 有一个 state 属性叫做 `count` 和一个 getter 叫做 `double`
    ...mapState(useCounterStore, {
      n: 'count',
      triple: store => store.n * 3,
      // 注意如果想使用 `this` 则不能使用箭头函数
      custom(store) {
        return this.someComponentValue + store.n
      },
      doubleN: 'double'
    })
  },
  created() {
    this.n // 2
    this.doubleN // 4
  }
}
```

   5、mapStores(…stores): _Spread。通过生成要在组件的 `computed` 字段中展开的对象，允许使用没有组合式API (`setup()`)的存储。它接受 store 定义的列表。

参数

| Name        | Type        | 描述                              |
| :---------- | :---------- | :-------------------------------- |
| `...stores` | […Stores[]] | 要映射到对象的存储（`store`）列表 |

例如：

```javascript
export default {
 computed: {
   // 其它计算属性
   ...mapStores(useUserStore, useCartStore)
 },
 created() {
   this.userStore // id为"user"的存储
   this.cartStore // id为"cart"的存储
 }
}
```

   6、 storeToRefs(store): ToRefs。用 store 的所有`state`、`getters`和 插件添加的（plugin-added）`state`、 属性（properties）创建一个引用对象。类似于`toRefs()`，但专门为 Pinia store 设计，

参数

| 名称    | 类型 | 描述                   |
| :------ | :--- | :--------------------- |
| `store` | `SS` | 用以提取`refs`的 store |