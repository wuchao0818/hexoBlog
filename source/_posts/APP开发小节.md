---
title: App 开发小结
categories: web前端
tags:
- app
- vue
---

### 1、css单位 px、 em 和 rem 的区别

##### px 是固定的像素，一旦设置了就无法因为适应页面大小而改变。应用于 Web 端。

##### em :

- 子元素字体大小的 em 是相对于父元素字体大小
- em 的值并不是固定的

##### rem :

- rem 相对单位，相对于根元素 `<html>` 
- 修改根元素就成比例地调整所有字体大小



### 2、手机自适应大小

在 `index.html`  文件中添加以下代码

```html
<meta content="width=device-width, initial-scale=1.0" name="viewport">
```

`width` : 把布局视口 = 理想视口，也即是设置成设备的实际宽度

`initial-scale=1.0`： 作用是改变视觉视口的，以及缩放系数



#### js 动态设置 rem 来实现移动端字体的自适应

```javascript
(function(doc, win) {
    var docEl = doc.documentElement,
    resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
    recalc = function() {
        // 获取当前浏览器窗口的宽度
        var clientWidth = docEl.clientWidth;
        if (!clientWidth) return;
        //设置html 根元素字体初始化大小值，页面在根据html font-size 设置 rem 进行自适应 
        docEl.style.fontSize = 100 * (clientWidth / 750) + 'px';
    };

    if (!doc.addEventListener) return;
    win.addEventListener(resizeEvt, recalc, false);
    doc.addEventListener('DOMContentLoaded', recalc, false);
})(document, window);
```



### 3、缓存组件

> 在 app 项目中,难免会有列表页面，点击进入详情页面，返回回来时，如果不对结果页面进行缓存,那么返回列表页面的时候会回到初始状态,但是我们想要的结果是返回时这个页面还是之前搜索的结果列表,这时候就需要用到 vue的 keep-alive 技术了.

#### keep - alive 简介

`keep-alive` 是 Vue 内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染。
用法也很简单：

在 App.vue 中添加如下代码

```javascript
<keep-alive>
    <router-view if="$router.meta.keepAlive"></router-view>
</keep-alive>
<router-view if="!$router.meta.keepAlive"></router-view>
```

以上路径匹配到的视图组件都会被缓存。如果只想 `router-view` 里面某个组件被缓存，怎么办？

##### 增加 router.meta 属性

```javascript
// routes 配置
export default [
  {
    path: 'course',
    name: 'yunxue-course',
    component: () => import {'@/compontents/yunxue/counrseList'},
    meta: {
      keepAlive: true // 需要被缓存
    }
  }
]
```

##### 清除缓存

```javascript
beforeRouteLeave(to,from,next){
     if(to.name === 'stydy-detail') {
         if(!from.meta.keepAlive) {
             from.meta.keepAlive = true
         }
     } else {
         from.meta.keepAlive = false
     }
      next()
},
```

在当前缓存页面利用 beforeRouteLeave 路由离开时进行缓存销毁。

##### activated deactivated

注意一点：activated,deactivated 这两个生命周期函数一定是要在使用了 keep-alive 组件后才会有的，否则则不存在当引入 keep-alive 的时候，页面第一次进入，钩子的触发顺序 created-> mounted-> activated，退出时触发 deactivated。当再次进入（前进或者后退）时，只触发activated。

事件挂载的方法等，只执行一次的放在 mounted 中；组件每次进去执行的方法放在 activated 中， activated 中的不管是否需要缓存多会执行。



## VueX 的使用

五个核心：state、getters、mutations、actions、modules。

1、state：vuex 的基本数据，用来存储变量

2、getters：从基本数据(state)派生的数据，相当于state的计算属性

3、mutation：提交更新数据的方法，必须是同步的(如果需要异步使用 action。每个 mutation 都有一个字符串的事件类型(type)和一个回调函数(handler)。

回调函数就是我们实际进行状态更改的地方，并且它会接受state作为第一个参数，提交载荷作为第二个参数。

4、action：和 mutation 的功能大致相同，不同之处在于==》1.Action 提交的是 mutation，而不是直接变更状态。2. Action 可以包含任意异步操作。

5、modules：模块化 vuex，可以让每一个模块拥有自己的 state、mutation、action、getters, 使得结构非常清晰，方便管理。

##### state

###### **单一状态树**

Vuex使用单一状态树，即用一个对象就包含了全部的状态数据。state 作为构造器选项，定义了所有我们需要的基本状态参数。

**在 Vue 组件中获得 Vuex 属性**

```javascript
const store = new Vuex.Store({
    state: {
        count:0
    }
})
const app = new Vue({
    //..
    store,
    computed: {
        count: function(){
            return this.$store.state.count
        }
    },
    //..
})
```

每当 store.state.count 变化的时候, 都会重新求取计算属性，并且触发更新相关联的 DOM。



##### **mutations**

提交 mutation 是更改 Vuex 中的 store 中的状态的唯一方法。

mutation 必须是同步的，如果要异步需要使用 action。

每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数，提交载荷作为第二个参数。

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    //无提交荷载
    increment(state) {
        state.count++
    }
    //提交荷载
    incrementN(state, obj) {
      state.count += obj.n
    }
  }
})
```



```javascript
//无提交荷载
store.commit('increment')
//提交荷载
store.commit('incrementN', {
    n: 100
})
```



##### **actions**

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

Mutation 通过 `store.commit` 触发，那么 Action 则通过 `store.dispatch` 方法触发。

```javascript
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

**调用**

```javascript
store.dispatch('actionA').then(() => {
  // ...
})
```



```javascript
  mutations: {
        //没有所谓的大写字母的Type了，就是一个一个函数
        add (state, n ) {
            state.count += n;
        },
        minus (state) {
            state.count--;
        }
    },
    actions : {
        add(context){
            $.get("api.txt",function(data){
                context.commit('add',Number(data));
            });
        }
    },　
```

action 在 vuex 中用于异步 commit 的发送 。

add 函数会自动获得一个默认参数 context, 它是一个 store 实例，通过它可以获取到 store 实例的属性和方法,如 context.state 就会获取到 state 属性， context.commit 就会执行commit命令。