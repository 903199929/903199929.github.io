# Vuex

概念：专门在Vue中实现集中式状态（数据）管理的一个Vue插件，对Vue应用中多个组件的共享状态进行集中式的管理（读/写），也是一种组件间的通信方式，且适用于任意组件间的通信

什么时候使用？
- 多个组件依赖于同一状态
- 来自不同组件的行为需要变更同一状态

## 使用

1. 安装vuex插件，`npm i vuex` 默认为4版本用于支持vue3.x，若为vue2.x则输入命令`npm i vuex@3`
2. 在`src`目录下新建`store`目录，并创建新文件命名为`index.js`
   ```javascript
    // index.js
    // 该文件用于创建Vuex最为核心的store

    // 引入Vue
    import Vue from 'vue'
    // 引入Vuex
    import Vuex from 'vuex'
    Vue.use(Vuex)

    // 准备actions用于响应组件中的动作
    const actions = {}
    // 准备mutations用于操作数据（state）
    const mutations = {}
    // 准备state用于存储数据
    const state = {}

    // 创建并暴露store
    export default new Vuex.Store({
        actions,
        mutations,
        state
    })
   ```

在`main.js`中引入
```javascript
// 引入store
import store from './store'

// 注册
new Vue({
  render: h => h(App),
  store
}).$mount('#app')
```

## 四个map方法的使用

---

1. mapState:用于帮助我们映射`state`中的数据为计算属性
   ```javascript
    computed:{
        // 借助mapState生成计算属性（对象写法）
        ...mapState({school:'school',name:'name'}),
        // 借助mapState生成计算属性（数组写法）
        ...mapState(['school','name'])
    }
   ```
2. mapGetters:用于帮助我们映射`getters`中的数据为计算属性
    ```javascript
    computed:{
        // 借助mapGetters生成计算属性（对象写法）
        ...mapGetters({school:'school',name:'name'}),
        // 借助mapState生成计算属性（数组写法）
        ...mapGetters(['school','name'])
    }
   ```
3. mapActions:用于帮助我们生成`actions`对话的方法，即：包含`$store.dispatch(xxx)`的函数
    ```javascript
    methods:{
        // 靠mapActions生成：increment,decrement（对象形式）
        ...mapActions({increment:'increment',decrement:'decrement'})
        // 靠mapActions生成：increment,decrement（数组形式）
        ...mapActions(['increment','decrement'])
    }
    ```
4. mapMutations:用于帮助我们生成与`mutations`对话的方法，即：包含`$store.commit(xxx)`
    ```javascript
    // 靠mapMutations生成：increment,decrement（对象形式）
    ...mapMutations({increment:'increment',decrement:'decrement'})
    // 靠mapMutations生成：increment,decrement（数组形式）
    ...mapMutations(['increment','decrement'])
    ```

**Tips:** mapActions和mapMutations使用时，若需要传递参数，在模版中绑定事件时传递好参数，否则参数是事件对象

## Vuex模块化

---

目的：让代码更好维护，让多种数据分类更加明确

步骤：
1. 修改`store.js`
    ```javascript
    const demo1 = {
        namespaced:true, //开启命名空间
        state:{x:1},
        mutations:{},
        actions:{},
        getters:{},
    }

    const demo2 = {
        namespaced:true, //开启命名空间
        state:{y:1},
        mutations:{},
        actions:{},
        getters:{},
    }

    const store = new Vuex.Store({
        modules:{
            demo1,
            demo2
        }
    })
    ```
2. 组件中读取`state`数据
    ```javascript
    // 直接读取
    this.$store.state.demo1.list
    // 借助mapState读取
    ...mapState('demo1',['school','name'])
    ```
3. 组件中读取`getters`数据
    ```javascript
    // 直接读取
    this.$store.getters['demo1/getName']
    // 借助mapState读取
    ...mapGetters('demo1',['getName'])
    ```
4. 组件中调用`dispatch`
    ```javascript
    // 直接读取
    this.$store.getters['demo1/getName',name]
    // 借助mapState读取
    ...mapActions('demo1',['getName'])
    ```
5. 组件中调用`commit`
    ```javascript
    // 直接读取
    this.$store.commit['demo1/getName',name]
    // 借助mapState读取
    ...mapMutations('demo1',['getName'])
    ```