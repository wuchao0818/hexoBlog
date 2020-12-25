---
title: React Hooks 详解
categories: web前端
date: 2020-05-29 13:42:39
tags:
- javascript
- React
---

### 一、React Hooks 介绍

> 2018年底 FaceBook 的 React 小组推出 Hooks 以来，`React Hooks `就是用函数的形式代替原来的继承类的形式，并且使用预函数的形式管理 `state`，有 Hooks 可以不再使用类的形式定义组件了。这时候你的认知也要发生变化了，原来把组件分为有状态组件和无状态组件，有状态组件用类的形式声明，无状态组件用函数的形式声明。那现在所有的组件都可以用函数来声明了。

**原始写法：**

```javascript
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
        this.state = { count:0 }
    }
    render() { 
        return (
            <div>
                <p>You clicked {this.state.count} times</p>
                <button onClick={this.addCount.bind(this)}>Chlick me</button>
            </div>
        );
    }
    addCount(){
        this.setState({count:this.state.count+1})
    }
}

export default Example;

```

**React Hooks 写法：**

```javascript
import React, { useState } from 'react';
function Example(){
    const [ count , setCount ] = useState(0);
    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>
        </div>
    )
}
export default Example;

```

hooks 的目的就是让你不再写 `class`，让 `function` 一统江湖。



### 二、useState 的介绍和多状态声明

>`useState`是 react 自带的一个 hook 函数，它的作用是用来声明状态变量。

```javascript
const [ count , setCount ] = useState(0); //单个状态声明

const [data, setData] = useState({   //多个状态声明
    msg: '',
    count: 0,
    list: [],
});
```

`useState` 这个函数接收的参数是状态的初始值 (Initial state)，它返回一个数组，这个数组的第 0 位是当前的状态值，第 1 位是可以改变状态值的方法函数。 所以上面的代码的意思就是声明了一个状态变量为 count，并把它的初始值设为 0，同时提供了一个可以改变 `count` 的状态值的方法函数。

```javascript
<p>You clicked {count} times</p>
```

你可以发现，我们读取是很简单的。只要使用 `{count}` 就可以，因为这时候的 count 就是 JS 里的一个变量，想在 `JSX` 中使用，值用加上 `{}` 就可以。

如果改变 `State` 中的值,看下面的代码:

```javascript
<button onClick={()=>{setCount(count+1)}}>click me</button>
```

**React Hooks 不能出现在条件判断语句中，因为它必须有完全一样的渲染顺序**



### 三、useEffect 代替常用生命周期

```javascript
import React, { useState , useEffect } from 'react';
function Example(){
    const [ count , setCount ] = useState(0);
    //---关键代码---------start-------
    useEffect(()=>{
        console.log(`useEffect=>You clicked ${count} times`)
    }) //替代 class 类中的 componentDidMount 和 componentDidUpdate
    //---关键代码---------end-------

    return (
        <div>
            <p>You clicked {count} times</p>
            <button onClick={()=>{setCount(count+1)}}>click me</button>
        </div>
    )
}
export default Example;
```

这代表第一次组件渲染和每次组件更新都会执行 `useEffect`。

> **useEffect 两个注意点：**

1. React 首次渲染和之后的每次渲染都会调用一遍 `useEffect` 函数，而之前我们要用两个生命周期函数分别表示首次渲染(componentDidMonut) 和更新导致的重新渲染 (componentDidUpdate)。
2. useEffect 中定义的函数的执行不会阻碍浏览器更新视图，也就是说这些函数时异步执行的，而 `componentDidMonut` 和 `componentDidUpdate` 中的代码都是同步执行的。

#### useEffect 实现 componentWillUnmount 生命周期函数

```javascript
 useEffect(()=>{
        /* 执行初始化操作,需要注意的是，如果你只是想在渲染的时候初始化一次数据，那么第二个参数必须传空数组。
        这样它就等价于只在componentDidMount的时候执行。
        如果不传第二个参数的话，它就等价于componentDidMount和componentDidUpdate */

        pollingQueryingStatus();
        console.log('name')
        return stopPollingQueryStatus; //在组件unmount的时候通过return清理副作用
     
    },[])  //用户传入了第二个参数，那么只有在第二个参数的值发生变化(以及首次渲染)的时候，才会触发effects
```



### 四、useContext 传值

>**`useContext` 主要是用来实现父子组件之间的传值 如下代码实现**

```javascript
import React,{useState ,useContext, createContext } from 'react'
const CountContext = createContext()

// 定义子组件
const Coounter = () =>{
   //子组件一句话就可以得到父组件传递过来的count
const count = useContext(CountContext)
return (<h2>{count}</h2>)
}

// 父组件
const IncreasedHooks2= () => {
  const [ count , setCount ] =useState(0)
  return (
    <div>
      <div>使用React Hooks</div>
       <p>总数:{count}</p>
        <button onClick={()=>setCount(count+1)}>增加</button>
        {/* 父组件向组件提供值 */}
        <CountContext.Provider value={count} >
          <Coounter/>
        </CountContext.Provider>
    </div>
  )
}
export default IncreasedHooks2

```



### 五、useMemo 优化 React Hooks 程序性能

>`useMemo` 主要用来解决使用 React hooks 产生的无用渲染的性能问题。使用 function 的形式来声明组件，失去了 `shouldCompnentUpdate`（在组件更新之前）这个生命周期，也就是说我们没有办法通过组件更新前条件来决定组件是否更新。而且在函数组件中，也不再区分 `mount` 和 `update` 两个状态，这意味着函数组件的每一次调用都会执行内部的所有逻辑，就带来了非常大的性能损耗。`useMemo` 和`useCallback `都是解决上述性能问题的。

**性能展示案例：**

```javascript

import React , {useState,useMemo} from 'react';

function Example7(){
    const [xiaohong , setXiaohong] = useState('小红待客状态')
    const [zhiling , setZhiling] = useState('志玲待客状态')
    return (
        <>
            <button onClick={()=>{setXiaohong(new Date().getTime())}}>小红</button>
            <button onClick={()=>{setZhiling(new Date().getTime()+',志玲向我们走来了')}}>志玲</button>
            <ChildComponent name={xiaohong}>{zhiling}</ChildComponent>
        </>
    )
}
```

父组件调用了子组件，子组件我们输出两个姑娘的状态，显示在界面上。代码如下：

```javascript
function ChildComponent({name,children}){
    function changeXiaohong(name){
        console.log('她来了，她来了。小红向我们走来了')
        return name+',小红向我们走来了'
    }

    const actionXiaohong = changeXiaohong(name)
    return (
        <>
            <div>{actionXiaohong}</div>
            <div>{children}</div>
        </>
    )
}
```

这时候你会发现在浏览器中点击`志玲`按钮，小红对应的方法都会执行，结果虽然没变，但是每次都执行，这就是性能的损耗。目前只有子组件，业务逻辑也非常简单，如果是一个后台查询，这将产生严重的后果。所以这个问题必须解决。当我们点击`志玲`按钮时，小红对应的 `changeXiaohong` 方法不能执行，只有在点击`小红`按钮时才能执行。

>**useMemo 优化性能**

**其实只要使用 `useMemo` ，然后给她传递第二个参数，参数匹配成功，才会执行。代码如下：**

```javascript
function ChildComponent({name,children}){
    function changeXiaohong(name){
        console.log('她来了，她来了。小红向我们走来了')
        return name+',小红向我们走来了'
    }
  
   /* 优化性能 */
    const actionXiaohong = useMemo(()=>changeXiaohong(name),[name]) //第2个参数为父组件发生变化时， 子组件只发生对应的state进行渲染，注意为数组格式。其他的state不进行改变。相当于 shouldCompnentUpdate 是否对其他的 state 进行页面渲染
    return (
        <>
            <div>{actionXiaohong}</div>
            <div>{children}</div>
        </>
    )
}

```

```javascript
export default function WithMemo() {
    const [count, setCount] = useState(1);
    const [val, setValue] = useState('');
    const expensive = useMemo(() => {
        console.log('compute');
        let sum = 0;
        for (let i = 0; i < count * 100; i++) {
            sum += i;
        }
        return sum;
    }, [count]);
 
    return <div>
        <h4>{count}-{val}-{expensive()}</h4>
        {val}
        <div>
            <button onClick={() => setCount(count + 1)}>+c1</button>
            <input value={val} onChange={event => setValue(event.target.value)}/>
        </div>
    </div>;


```

这里创建了两个state，然后通过expensive函数，执行一次昂贵的计算，拿到count对应的某个值。我们可以看到：无论是修改count还是val，由于组件的重新渲染，都会触发expensive的执行(能够在控制台看到，即使修改val，也会打印)；但是这里的昂贵计算只依赖于count的值，在val修改的时候，是没有必要再次计算的。在这种情况下，我们就可以使用useMemo，只在count的值修改时，执行expensive计算。这样，就只会在count改变的时候触发expensive执行，在修改val的时候，返回上一次缓存的值。



### 六、useCallback 缓存函数

>**问题引发：**
>
>- 子组件onChange调用了父组件的handleOnChange
>- 父组件handleOnChange内部会执行setText(e.target.value)引起父组件更新
>- 父组件更新会得到新的handleOnChange，传递给子组件，对于子组件来说接收到一个新的props
>- 子组件进行不必要更新
>- 使用场景是：有一个父组件，其中包含子组件，子组件接收一个函数作为 props；通常而言，如果父组件更新了，子组件也会执行更新；但是大多数场景下，更新是没有必要的，我们可以借助useCallback来返回函数，然后把这个函数作为 props传递给子组件；这样，子组件就能避免不必要的更新

**使用 useCallback 解决：**

```javascript
import React, { useState, memo, useMemo, useCallback } from 'react'

const Child = memo((props) => {
  console.log(props);

  return (
    <div>
      <input type="text" onChange={props.onChange}/>
    </div>
  )
})

const Parent = () => {
  const [count, setCount] = useState(0)
  const [text, setText] = useState('')

  const handleOnChange = useCallback((e) => {
    setText(e.target.value)
  },[]) //第二个参数为 text 的时候  子组件会重新render

  return (
    <div>
      <div>count: {count}</div>
      <div>text: {text}</div>
      <button onClick={() => {
        setCount(count + 1)
      }}>+1</button>
      <Child onChange={handleOnChange} />
    </div>
  )
}

function App() {
  return <div><Parent /></div>
}

export default App
```



1. handleOnChange 被缓存了下来，尽管父组件更新了，但是拿到的 handleOnChange 还是同一个
2. 对比 useMemo，useMemo 缓存的是一个值，useCallback 缓存的是一个函数，是对一个单独的 props 值进行缓存
3. memo 缓存的是组件本身，是站在全局的角度进行优化
4. 子组件接受父组件 props 传过来的方法执行后，改变父组件的 UI 视图，但同时子组件不希望执行本身的渲染，以免浪费性能。







