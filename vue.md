* <font size=red>[vue]</font>问：
```
this.data.abc[qwe].wq = index
```
为什么没有用？
* 答：在vue中有时候会出现监听失效的情况，这种时候一般要使用$set去做，这种情况一般存在于数组的时候，因为在此vue只对整个数组对象监听，没法对字符串的数组元素个别监听
*   由于 JavaScript 的限制， Vue 不能检测以下变动的数组：
*   当你利用索引直接设置一个项时，例如： vm.items[indexOfItem] = newValue
*   当你修改数组的长度时，例如： vm.items.length = newLength
*   为了解决第一类问题，以下两种方式都可以实现和 vm.items[indexOfItem] = newValue 相同的效果， 同时也将触发状态更新：
*   1.Vue.set(example1.items, indexOfItem, newValue)
*   2.example1.items.splice(indexOfItem, 1, newValue)
*   为了解决第二类问题，你也同样可以使用 splice：
*   example1.items.splice(newLength)