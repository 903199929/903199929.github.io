# Vue3.x

## 常用`Composition API`

---

### setup

Vue3.x中一个新的配置项，值为一个函数  
组件中用到的数据、方法等均配置在`setup`中

`setup`函数都两种返回值
1. 若返回一个对象， 则对象中的属性、方法在模版中均可直接使用（注意！！！）
2. 若返回一个渲染函数，则可以自定义渲染内容(了解)

**Tips:** 
1. 尽量不要与Vue2.x配置混用
2. `setup`不能是一个async函数，因为返回值不能再是return的对象，而是promise，模版看不到return对象中的属性

执行的时机：在`beforeCreate`之前执行一次，this是undefined

参数：
- props: 值为对象，包含组件外传递过来，且组件内部声明接收了的属性
- context: 上下文对象
  - attrs: 值为对象，包含：组件外部传递过来，但是没有在props配置中声明的属性，相当于`this.$attrs`
  - slots: 收到的插槽内容，相当于`this.$slots`
  - emit: 分发自定义事件的函数，相当于`this.$emit`

### ref函数

用于定义一个响应式数据  
语法：`const xxx = ref(initValue)`  
- 创建一个包含响应式数据的**引用对象**
- JS中操作数据：`xxx.value`
- 模版中读取数据：`<div>{{xxx}}</dic>`

**Tips:**
- 接收的数据可以是基本类型，也可以是对象类型
- 基本类型的数据：响应式依然是靠`Object.defineProperty()`的get和set完成的
- 对象类型的数据：内部用了Vue3.x的一个新函数`reactive`函数

### reactive函数

用于定义一个对象类型的响应式数据（基本类型用`ref`）  
语法：`const 代理对象 = reactive(源对象)`接收一个对象（或数组），返回一个代理对象  
reactive定义的响应式数据是深层次的  
内部基于ES6的Proxy实现，通过代理对象操作源对象内部数据进行操作

### watchEffect函数

watch：即要指明监视的属性，也要指明监视的回调
watchEffect函数：不用指明监视哪个属性，监视的回调中用到哪个属性，就监视哪个属性

```javascript
// 指定的回调中要到的数据只要发生了变化则直接重新执行回调
watchEffect(()=>{
    const x1 = sum.value
    const x2 = person.age
})
```

### 自定义hook函数

---

本质是一个函数，把setup中使用的Composition API进行了封装  
类似于vue2.x中的mixin  
用于复用代码，让setup中的逻辑更清晰

### toRef

用于创建一个ref对象，其中value值指向另一个对象中的某个属性

语法：`const name = toRef(person,'name')`  
应用：要将响应式对象中的某个属性单独提供给外部使用时  
扩展：`toRefs`与`toRef`功能一致，但可以批量创建多个`ref`对象,语法`toRefs(person)`

### shallowReactive 与 shallowRef

shallowReactive: 只处理对象最外层属性的响应式
shallowRef: 只处理基本数据类型的响应式，不进行对象的响应式处理

- 当一个对象数据，结构比较深，但变化时只是外层属性变化时用`shallowReactive`
- 当一个对象数据，后续功能不会修改该对象中的属性，而是生成新的对象来替换时用`shallowRef`

### readonly 与 shallowReadonly

readonly: 让一个响应式数据变为只读的（深只读）  
shallowReadonly: 让一个响应式数据变为只读的（浅只读）  
当不希望数据被修改时使用

### toRaw 与 markRaw

toRaw: 将一个由reactive生成的响应式对象转为普通对象，用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新

markRaw: 标记一个对象，使其永远不会再成为响应式对象，用于将不应该被设置为响应式的值（例如复杂的第三方类库等）或当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能

