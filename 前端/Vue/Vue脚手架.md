# Vue脚手架

## 安装

---



## 属性

---

### ref属性

1. 被用来给元素或子组件引用信息（id的替代者）
2. 应用在html标签上获取真实DOM元素，应用在组件标签上是组件实例对象
3. 使用方式：
  1. `<h1 ref="xxx">.....<h1>`
  2. `this.$refs.xxx`


### props属性

功能：让组件接收外部传来的数据
1. 传递数据： `<Student name="xx",age="18",sex="male">`
2. 接收数据有以下三种

**Tips:** `props`是只读的，Vue底层会检测对`props`的修改，如果进行了修改，就会发出警告，若业务需求确实需要修改，请复制`props`的内容到data中，再去修改data中的数据
  
    简单声明接收
    props:['name','age','sex']

    接收的同时对数据进行类型限制
    props:{
        name:String,
        age:Number,
        sex:String
    }

    增加默认值的指定及必要性限制
    props:{
        name:{
            type:String,
            required:true
        },
        age:{
            type:Number,
            default:18
        },
        sex:{
            type:String,
            required:true
        }
    }

### mixin属性

功能：可以把多个组件共用的配置提取成一个混入对象  

```html
<script>
    import {mix} from '../mix'

    export default {
        name:'Student',
        data(){
            name:xx
        },
        mixins:[mix]
    }
</script>
```

```javascript
export const mix{
    methods:{
        showName(){
            alert(this.name)
        }
    }
}
```

全局混合  
在`main.js`中添加
```javascript
import {mix} from './mix'

Vue.minin(mix)
```

## 插件

---

功能：用于增强Vue  
本质：包含install方法的一个对象，install的第一个参数是Vue，第二个以后的参数是插件使用者传递的数据  
定义插件:
```javascript
对象.install = function(Vue,Options){
    // 1. 添加全局过滤器
    Vue.filter(...)

    //2. 添加全局指令
    Vue.directive(...)

    //3. 配置全局混入
    Vue.mixin(...)

    //4. 添加实例方法
    Vue.prototype.$myMethod = function(){...}
    Vue.prototype.$myProperty = xxxx
}
```
使用插件：`Vue.use()`

## 全局事件总线(GlobalEventBus)

---

1. 一种组件间通信的方式，适用于任意组件间通信
2. 安装全局事件总线
   ```javascript
    new Vue({
        .....
        beforCreate(){
            // 安装全局事件总线，$bus就是当前应用的vm
            Vue.prototype.$bus = this
        }
        .....
    })
   ```
3. 使用事件总线
   1. 接收数据：A组件想接收数据，则在A组件中给`$bus`绑定自定义事件，事件的*回调留在A组件自身*
   ```javascript
    methods(){
        demo(data){....}
        .....
        mounted(){
            this.$bus.on('xxx',this.demo)
        }
    }
   ```
   2. 提供数据：`this.$bus.$emit('xxx',数据)`
4. 最后在`beforeDestroy`钩子中，用`$off`去解绑*当前组件所用到的*事件

## nextTick

---

1. 语法：`this.$nextTick`（回调函数）
2. 作用：在下一次DOM更新结束后执行其指定的回调
3. 使用：当数据改变后，要基于更新后的新DOM进行某些操作时，要在`nextTick`所指定的回调函数中执行