# Promise

一个 Promise 对象代表一个在这个 promise 被创建出来时不一定已知的值。它让您能够把异步操作最终的成功返回值或者失败原因和相应的处理程序关联起来。 这样使得异步方法可以像同步方法那样返回值：异步方法并不会立即返回最终的值，而是会返回一个 promise，以便在未来某个时候把值交给使用者。  

一个 Promise 必然处于以下几种状态之一：
- 待定（pending）: 初始状态，既没有被兑现，也没有被拒绝。
- 已兑现（fulfilled）: 意味着操作成功完成。
- 已拒绝（rejected）: 意味着操作失败。

待定状态的 Promise 对象要么会通过一个值被兑现（fulfilled），要么会通过一个原因（错误）被拒绝（rejected）。当这些情况之一发生时，我们用 promise 的 then 方法排列起来的相关处理程序就会被调用。如果 promise 在一个相应的处理程序被绑定时就已经被兑现或被拒绝了，那么这个处理程序就会被调用，因此在完成异步操作和绑定处理方法之间不会存在竞争状态。

Promise的格式代码如下：
```html
<body>
    <div class="container">
        <h2 class="page-header">Promise</h2>
        <button class="btn btn-primary" id="btn">点击抽奖</button>
    </div>
    <script>
        //生成随机数
        function rand(m, n) {
            return Math.ceil(Math.random() * (n - m + 1)) + m - 1;
        }

        const btn = document.querySelector('#btn');
        //const btn = document.getElementById('btn')
        //const btn = document.getElementsByTagName('button')[0]
        btn.addEventListener('click', function () {
            // 原始写法
            // setTimeout(()=>{
            //     let num = rand(1,100)
            //     if(num<=30){
            //         alert("中奖了")
            //     }else{
            //         alert("再接再厉")
            //     }
            // },500)

            // Promise写法
            const p = new Promise((resolve, reject) => {
                setTimeout(() => {
                    let num = rand(1, 100)
                    if (num <= 30) {
                        resolve(num)
                    } else {
                        reject(num)
                    }
                }, 1000)
            })

            p.then(
                (value) => {
                    alert("中奖了,中奖号码为"+value)
                },
                (reason) => {
                    alert("再接再厉，未中奖号码为"+reason)
                })
        })
    </script>
</body>
```

## All方法

```html
<script>
    let p1 = Promise.resolve('1')
    let p2 = Promise.resolve('2')
    let p3 = Promise.resolve('3')

    const result = Promise.all([p1,p2,p3])
    console.log(result)
</script>
```

只有all方法里面的都成功才为成功

## race方法

```html
<script>
    let p1 = Promise.resolve('1')
    let p2 = Promise.resolve('2')
    let p3 = Promise.resolve('3')

    const result = Promise.race([p1,p2,p3])
    console.log(result)
</script>
```

返回一个新的Promise，第一个完成Promise的结果状态就是最终的结果状态

## 自定义封装

---

### Promise

```javascript
function Promise(executor) {

    // 添加属性
    this.PromiseState = 'pending';
    this.PromiseResult = null

    // 保存实例对象的this的值
    const _this = this

    // resolve函数
    function resolve(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'fulfilled';
        // 设置对象结果值
        _this.PromiseResult = data;
    }

    // reject函数
    function reject(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'rejected';
        // 设置对象结果值
        _this.PromiseResult = data;
    }
}
```

### Catch

```javascript
try {
    // 同步调用 执行器函数
    executor(resolve, reject)
}
catch (e) {
    // 修改Promise状态为失败
    reject(e)
}
```

### then

```javascript
// 添加then方法
Promise.prototype.then = function(onResolved,onRejected){

    // 调用回调函数
    if(this.PromiseState==='fulfilled'){
        onResolved(this.PromiseResult)
    }
    if(this.PromiseState==='rejected'){
        onRejected(this.PromiseResult)
    }
}
```

## 异步任务回调的执行

修改为如下

```javascript
function Promise(executor) {

    // 添加属性
    this.PromiseState = 'pending';
    this.PromiseResult = null

    // 声明属性
    this.callback = {};

    // 保存实例对象的this的值
    const _this = this

    // resolve函数
    function resolve(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'fulfilled';
        // 设置对象结果值
        _this.PromiseResult = data;

        if(_this.callback.onResolved){
            _this.callback.onResolved(data);
        }
    }

    // reject函数
    function reject(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'rejected';
        // 设置对象结果值
        _this.PromiseResult = data;

        if(_this.callback.onRejected){
            _this.callback.onRejected(data);
        }

    }

    try {
        // 同步调用 执行器函数
        executor(resolve, reject)
    }
    catch (e) {
        // 修改Promise状态为失败
        reject(e)
    }
}

// 添加then方法
Promise.prototype.then = function (onResolved, onRejected) {

    // 调用回调函数
    if (this.PromiseState === 'fulfilled') {
        onResolved(this.PromiseResult)
    }
    if (this.PromiseState === 'rejected') {
        onRejected(this.PromiseResult)
    }

    // 判断pending状态
    if (this.PromiseState === 'pending') {
        // 保存回调函数
        this.callback = {
            onResolved: onResolved,
            onRejected: onRejected
        }
    }
}
```

### 指定多个回调的实现

```javascript
function Promise(executor) {

    // 添加属性
    this.PromiseState = 'pending';
    this.PromiseResult = null

    // 声明属性
    this.callbacks = [];

    // 保存实例对象的this的值
    const _this = this

    // resolve函数
    function resolve(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'fulfilled';
        // 设置对象结果值
        _this.PromiseResult = data;

        _this.callbacks.forEach(item=>{
            item.onResolved(data);
        })
    }

    // reject函数
    function reject(data) {
        //判断状态
        if (_this.PromiseState !== 'pending') return;
        // 修改对象的状态
        _this.PromiseState = 'rejected';
        // 设置对象结果值
        _this.PromiseResult = data;

        _this.callbacks.forEach(item=>{
            item.onRejected(data);
        })
    }

    try {
        // 同步调用 执行器函数
        executor(resolve, reject)
    }
    catch (e) {
        // 修改Promise状态为失败
        reject(e)
    }
}

// 添加then方法
Promise.prototype.then = function (onResolved, onRejected) {

    // 调用回调函数
    if (this.PromiseState === 'fulfilled') {
        onResolved(this.PromiseResult)
    }
    if (this.PromiseState === 'rejected') {
        onRejected(this.PromiseResult)
    }

    // 判断pending状态
    if (this.PromiseState === 'pending') {
        // 保存回调函数
        this.callbacks.push({
            onResolved: onResolved,
            onRejected: onRejected
        });
    }
}
```

### then方法的优化，以及同步和异步修改状态
```javascript
Promise.prototype.then = function (onResolved, onRejected) {
    const _this = this;

    return new Promise((resolve, reject) => {

        // 封装函数
        function callback(type){
            try {
                // 获取回调函数的执行结果
                let result = type(_this.PromiseResult)

                // 判断是否为Promise
                if (result instanceof Promise) {
                    // 如果是Promise类型的对象
                    result.then(v => {
                        resolve(v)
                    }, r => {
                        reject(r)
                    })
                } else {
                    // 结果的对象状态为成功
                    resolve(result)
                }
            } catch (e) {
                reject(e);
            }
        }

        // 调用回调函数
        if (this.PromiseState === 'fulfilled') {
            callback(onResolved);
        }
        if (this.PromiseState === 'rejected') {
            callback(onRejected);
        }

        // 判断pending状态
        if (this.PromiseState === 'pending') {
            // 保存回调函数
            this.callbacks.push({
                onResolved: function () {
                    callback(onResolved);
                },
                onRejected: function () {
                    callback(onRejected);
                }
            });
        }
    })
}
```

### catch方法

```javascript
// then方法中加入
// 判断回调函数参数
if (typeof onRejected !== 'function') {
    onRejected = reason => {
        throw reason;
    }
}

// 添加catch 方法
Promise.prototype.catch = function (onRejected) {
    return this.then(undefined, onRejected);
}
```

### 添加resolve 方法

```javascript
Promise.resolve = function (value) {
    return new Promise((resolve, reject) => {
        if (value instanceof Promise) {
            value.then(v => {
                resolve(v);
            }, r => {
                reject(r);
            })
        } else {
            resolve(value)
        }
    })
}
```

### 添加reject 方法

```javascript
Promise.reject = function (reason) {
    return new Promise((resolve, reject) => {
        reject(reason);
    })
}
```

### 添加all 方法

```javascript
Promise.all = function (promises) {
    return new Promise((resolve, reject) => {
        // 声明变量
        let count = 0;
        let arr = [];

        // 遍历
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(v => {
                count++;
                arr[i] = v;
                if (count === promises.length) {
                    // 修改状态
                    resolve(arr);
                }
            }, r => {
                reject(r);
            })
        }
    })
}
```

### 添加race 方法

```javascript
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(v => {
                resolve(v);
            }, r => {
                reject(r);
            })
        }
    })
}
```
