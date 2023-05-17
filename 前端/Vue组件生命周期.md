# Vue组件生命周期

## 一、生命周期

初次加载：
1.`beforeCreate` **创建前**：组件创建执行
2.`created` **创建后**：组件创建后执行
2.`beforeMount` **挂载前**：组件挂载前执行
3.`mounted` **挂载后**：组件挂载后执行

更新：
1.`beforeUpdated` 更新前执行
2.`updated` 更新完成后执行

卸载：
1.`beforeDestory` 卸载前执行
2.`destoryed` 卸载后执行

异常：
1.`errorCaptured` 异常捕捉

`keepalive`专属
1.`actived` 组件激活时执行，第一加载也会执行
2.`deactived` 组件失活时执行，组件卸载时也会执行

## 二、父子组件生命周期执行顺序

1.挂载过程：父`beforeCreate`->父`created`->父`beforeMount`->子`beforeCreate`->子`created`->子`beforeMount`->子`mounted`->父`mounted`

2.卸载过程：父`beforeDestroy`->子`beforeDestroy`->子`destroyed`->父`destroyed`

3.从父元素出发，等所有子元素闭合，在从父元素接结束

## 三、注意事项

1.`vue`的[响应式](https://so.csdn.net/so/search?q=响应式&spm=1001.2101.3001.7020)思想是，只有**应用到模板上**的响应式数据（除了`props`）发生了修改，才会触发更新生命周期，对于没有应用到模板上的响应式数据，即使发生修改，也不会触发更新周期。

2.对于组件的`props`，要分两种情况讨论

（1）当是基础数据时，只要发生改变，不管是否应用到组件上，都会执行更新生命周期

（2）当是对象时，又可分为两种情况

①替换整个对象（引用地址发生改变），会执行更新生命周期，
②更新对象的某个属性时，当属性被应用到模板上，会执行更新生命周期，没有被引用到模板上，不会执行更新生命周期。