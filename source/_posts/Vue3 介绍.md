---
title: Vue3 介绍（一）
categories: web前端
tags:
- javascript
- Vue
---

### 1、Vue-cli 和 Vue3 环境搭建

#### 安装 vue-cli

> Vue-cli ：Vue.js 开发的标准工具 ， 新版的 vue-cli 还提供了图形界面，如果你对命令行陌生，也可以使用图形界面。

**安装**：

```basic
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

查看版本命令

```basic
vue --version
```

#### 使用 vue-cli 创建项目

```basic
vue create vue3-1 //后面是创建的文件夹名称
```

输入完成后，会有一句这样的询问

```basic
Your connection to the default yarn registry seems to be slow.
   Use https://registry.npm.taobao.org for faster installation? (Y/n)
```

意思是你不能科学上网，建议你使用淘宝源，这时候你需要选择`Y`,也就是使用淘宝源进行创建。如果你已经配置淘宝源不会显示这个选项。

当你选择`Y`之后，就会跳出三个菜单让你选择。

```basic
? Please pick a preset: (Use arrow keys)            //请选择预选项
> Default ([Vue 2] babel, eslint)                   //使用 Vue2 默认模板进行创建
Default (Vue 3 Preview) ([Vue 3] babel, eslint)     //使用 Vue3 默认模板进行创建
  Manually select features                          //手动选择(自定义)的意思
```

成功创建后，会出现以下内容

```basic
one in 10.33s.


 $ cd vue3-1
 $ yarn serve
```

根据提示在命令行输入`cd vue3-1`进入项目，然后再输入`yarn serve`开启项目预览。这时候就会给出两个地址，都可以访问到现在的项目

```basic
 App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.0.118:8080/

  Note that the development build is not optimized.
  To create a production build, run yarn build.
```



## 2、setup() 和 ref()

用 `setup()` 新语法，写了这个就不需要再写 `data` 这样的东西了。然后在 `setup` 中加入一个 `girls` 变量，为了能让模板中进行使用，还需要返回。（有的小伙伴此时可能会感觉有点复杂，没有 Vue2 直接写 data 那么直观，但这正式 Vue3 的先进之处，不用暴漏在界面中的变量就可以不进行返回了。精确控制那些方法和变量被导出和使用）。

```javascript
<script lang="ts">
import { defineComponent, ref } from "vue";
export default defineComponent({
  name: "App",
  setup() {
    const girls = ref(["大脚", "刘英", "晓红"]);
    return {
      girls
    };
  },
});
</script>
```

写完后，这里使用 `v-for` 语法。代码如下

```html
<button v-for="(item, index) in girls" v-bind:key="index">
  {{ index }} : {{ item }}
</button>
```

现在要实现的就是当我们点击按钮的时候，触发一个方法 `selectGirlFun`，方法绑定一个选定值 `selectGirl`,具体实现代码如下。

```javascript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <div>
    <button
      v-for="(item, index) in girls"
      v-bind:key="index"
      @click="selectGirlFun(index)"
    >
      {{ index }} : {{ item }}
    </button>
  </div>
  <div>你选择了【{{ selectGirl }}】为你服务</div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
export default defineComponent({
  name: "App",
  setup() {
    const girls = ref(["大脚", "刘英", "晓红"]);
    const selectGirl = ref("");
    const selectGirlFun = (index: number) => {
      selectGirl.value = girls.value[index];
    };
    //因为在模板中这些变量和方法都需要条用，所以需要return出去。
    return {
      girls,
      selectGirl,
      selectGirlFun,
    };
  },
});
</script>
```

- `setup` 函数的用法，可以代替 Vue2 中的 date 和 methods 属性，直接把逻辑卸载 setup 里就可以。
- `ref` 函数的使用，它是一个神奇的函数，我们这节只是初次相遇，要在 `template` 中使用的变量，必须用`ref`包装一下。
- `return`出去的数据和方法，在模板中才可以使用，这样可以精准的控制暴漏的变量和方法。



## 3、reactive()

在 `setup` 中要改变和读取一个值的时候，还要加上 `value`。这种代码一定是可以优化的，需要引入一个新的 API `reactive`。它也是一个函数(方法)，只不过里边接受的参数是一个 Object (对象)。

然后无论是变量和方法，都可以作为 Object 中的一个属性，而且在改变`selectGirl` 值的时候也不用再加讨厌的 `value` 属性了。再 `return` 返回的时候也不用一个个返回了，只需要返回一个 `data`,就可以了。

```javascript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <div>
    <button
      v-for="(item, index) in data.girls"
      v-bind:key="index"
      @click="data.selectGirlFun(index)"
    >
      {{ index }} : {{ item }}
    </button>
  </div>
  <div>你选择了【{{ data.selectGirl }}】为你服务</div>
</template>


<script lang="ts">
import { ref, reactive } from "vue";
export default {
  name: "App",
  setup() {
    const data = reactive({
      girls: ["大脚", "刘英", "晓红"],
      selectGirl: "",
      selectGirlFun: (index: number) => {
        data.selectGirl = data.girls[index];
      },
    });

    return {
      data,
    };
  },
};
</script>

```

修改完成`<script>`部分的代码后，还需要修改`template`部分的代码，因为我们这时候返回的只有`data`,所以模板部分的字面量前要加入`data`。

##### 用 toRefs() 继续优化

现在 `template` 中，每次输出变量前面都要加一个 `data` ，这是可以优化的。但是不能用 `...` 扩展运算符。因为结构后就变成了普通变量，不再具有响应式的能力。为了解决这个问题，需要使用 Vue3 的一个新函数 `toRefs()`，使用前需要先进行引入。

```basic
import { reactive, toRefs } from "vue";
```

引入后就可以对 `data` 进行包装，把 data 变成 `refData`，这样就可以使用扩展运算符的方式了。具体代码如下：

```vue
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <div>
    <h2>欢迎光临红浪漫洗浴中心</h2>
    <div>请选择一位美女为你服务</div>
  </div>
  <div>
    <button
      v-for="(item, index) in girls"
      v-bind:key="index"
      @click="selectGirlFun(index)"
    >
      {{ index }} : {{ item }}
    </button>
  </div>
  <div>你选择了【{{ selectGirl }}】为你服务</div>
</template>

<script lang="ts">
export default {
  name: "App",
  setup() {
    // const girls = ref(["大脚", "刘英", "晓红"]);
    // const selectGirl = ref("");
    // const selectGirlFun = (index: number) => {
    //   selectGirl.value = girls.value[index];
    // };
    const data: DataProps = reactive({
      girls: ["大脚", "刘英", "晓红"],
      selectGirl: "",
      selectGirlFun: (index: number) => {
        data.selectGirl = data.girls[index];
      },
    });
    const refData = toRefs(data);

    return {
      ...refData,
    };
  },
};
</script>
```



### Vue3.x 的生命周期和钩子函数

- setup() :开始创建组件之前，在 `beforeCreate` 和 `created` 之前执行。创建的是 `data` 和 `method`
- onBeforeMount() : 组件挂载到节点上之前执行的函数。
- onMounted() : 组件挂载完成后执行的函数。
- onBeforeUpdate(): 组件更新之前执行的函数。
- onUpdated(): 组件更新完成之后执行的函数。
- onBeforeUnmount(): 组件卸载之前执行的函数。
- onUnmounted(): 组件卸载完成后执行的函数
- onActivated(): 被包含在`<keep-alive>`中的组件，会多出两个生命周期钩子函数。被激活时执行。
- onDeactivated(): 比如从 A 组件，切换到 B 组件，A 组件消失时执行。
- onErrorCaptured(): 当捕获一个来自子孙组件的异常时激活钩子函数（以后用到再讲，不好展现。

Vue3.x 生命周期在调用前需要先进行引入

```javascript
import {
  reactive,
  toRefs,
  onMounted,
  onBeforeMount,
  onBeforeUpdate,
  onUpdated,
} from "vue";

```

```javascript

<script lang="ts">

//....
const app = {
  name: "App",
  setup() {
    console.log("1-开始创建组件-----setup()");
    const data: DataProps = reactive({
      girls: ["大脚", "刘英", "晓红"],
      selectGirl: "",
      selectGirlFun: (index: number) => {
        data.selectGirl = data.girls[index];
      },
    });
    onBeforeMount(() => {
      console.log("2-组件挂载到页面之前执行-----onBeforeMount()");
    });

    onMounted(() => {
      console.log("3-组件挂载到页面之后执行-----onMounted()");
    });
    onBeforeUpdate(() => {
      console.log("4-组件更新之前-----onBeforeUpdate()");
    });

    onUpdated(() => {
      console.log("5-组件更新之后-----onUpdated()");
    });

    const refData = toRefs(data);

    return {
      ...refData,
    };
  },
};
export default app;
</script>
```

写完后可以到浏览器看一下效果，效果和你想象的应该是一样的。

```javascript
1 - 开始创建组件---- - setup();
2 - 组件挂载到页面之前执行---- - onBeforeMount();
3 - 组件挂载到页面之后执行---- - onMounted();
4 - 组件更新之前---- - onBeforeUpdate();
5 - 组件更新之后---- - onUpdated();

```

##### Vue2.x 和 Vue3.x 生命周期对比

```javascript
Vue2--------------vue3
beforeCreate  -> setup()
created       -> setup()
beforeMount   -> onBeforeMount
mounted       -> onMounted
beforeUpdate  -> onBeforeUpdate
updated       -> onUpdated
beforeDestroy -> onBeforeUnmount
destroyed     -> onUnmounted
activated     -> onActivated
deactivated   -> onDeactivated
errorCaptured -> onErrorCaptured

```

