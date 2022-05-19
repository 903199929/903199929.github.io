# webStorage

1. 存储内容大小一般为5M左右（不同浏览器可能不同）
2. 浏览器端通过`Window.sessionStorage`和`Window.localStorage`属性来实现本地存储机制
3. 相关API：
   1. `xxxxStorage.setItem('key','value');` 该方法接受一个键和值作为参数，会把键值对添加到存储中，如果键名存在，则更新对应的值
   2. `xxxxStorage.getItem('key');` 该方法接受一个键名作为参数，返回键名对应的值
   3. `xxxxStorage.removeItem('key');` 该方法接受一个键名作为参数，并把该键名从存储中删除
   4. `xxxxStorage.clear()` 该方法会清空存储中所有数据
4. **Tips:** 
   1. `SessionStorage`存储的内容会随着浏览器窗口关闭而消失
   2. `LocalStorage`存储的内容，需要手动清除才会消失
   3. `xxxxStorage.getItem('key');`如果key对应的value获取不到，那么`getItem`的返回值是null
   4. `JSON.parse(null)`的结果依然是null

对象在获取或拿缓存时需要进行转换
1. `JSON.stringify(object)`
2. `JSON.parse(object)`