## React函数组件

react一直推荐使用函数组件，这肯定是因为类组件有不足的地方，由于自身react开发经验不是很多，所以只能依靠谷歌了。

### 类组件的不足

- 状态逻辑难以复用： 在组件之间复用状态逻辑很难，可能要用到 `渲染属性` 或者 `高阶组件`，但无论是渲染属性，还是高阶组件，都会在原先的组件外包裹一层div，导致层级冗余
- 趋向复杂难以维护：
  - 在生命周期函数中混杂与组件本身功能不相关的逻辑，影响阅读
  - 类组件中到处都是对状态的访问和处理，导致组件难以拆分成更小的组件
- **this指向问题：** 根据谷歌到的内容，这一块好像才是重中之重，[例如这篇译文所述](https://zhuanlan.zhihu.com/p/62767474)，在React生态里，props是不可变数据，永远不会改变。但是，this却始终是可变的 。

### 函数组件的优势

- 能在无需修改组件结构的情况下复用状态逻辑（自定义hook）
- 能够很方便的拆分组件，即拆分成更小的函数
- 不需要对this进行操作

而函数组件之所以能够替代类组件最关键的地方就是hook，因为函数组件本身没有状态，没有生命周期，所以这些都靠hook来完成。

### React hook

#### state hook

state hook是一个在函数组件中使用的函数（useState），用于在函数组件中使用状态

useState

```javascript
const [state, setState] = useState(initialState);
```

- 函数有一个参数，这个参数的值表示状态的默认值
- 函数的返回值是一个数组，该数组一定包含两项
  - 第一项是状态的值
  - 第二项是函数，该函数用于改变状态

一个函数组件中可以有多个状态，有利于拆分不同功能的不同状态。

**注意的细节**

1. useState最好写到函数的起始位置，便于阅读
2. useState严禁出现在判断、循环中
3. 如果使用函数改变数据，若数据和之前的数据完全相等，不会导致重新渲染
4. 使用函数改变数据，传入的值不会和原来的数据进行合并，而是直接替换
5. **如果某些状态之间没有必然的联系，应该分化为不同的状态，而不要合并成一个对象**
6. 和类组件的状态一样，函数组件中改变状态可能是异步的（在Dom事件中），多个状态变化会合并以提高效率。此时，不能信任之前的状态，而应该使用回调函数的方式改变状态。
7. hook内部使用Object.is来比较新/旧state是否相等

#### state hook原理

当运行一个函数组件时（调用该函数），
1. 第N次调用useState
2. 检查该节点的状态数组是否存在下标N
3. 不存在
   1. 使用默认值创建一个状态
   2. 将该状态加入到状态数组中，下标为N
4. 存在
   1. 忽略掉默认值
   2. 直接得到默认值

#### effect hook

effect hook：用于在函数组件中处理副作用

```javascript
import React,{useState,useEffect} from 'react';
import ReactDOM from 'react-dom';
function Counter(){
    const [number,setNumber] = useState(0);
    useEffect(() => {
        document.title = `你点击了${number}次`;
    });
    return (
        <>
            <p>{number}</p>
            <button onClick={()=>setNumber(number+1)}>+</button>
        </>
    )
}
ReactDOM.render(<Counter />, document.getElementById('root'));
```

副作用：

1. ajax请求
2. 计时器
3. 其他异步操作
4. 更改真实Dom对象
5. 本地存储
6. 其他会对外部产生影响的操作

函数：useEffect，该函数接收一个函数作为参数，接收的函数就是需要进行副作用操作的函数

**细节**

1. 副作用函数的运行时间点，是在页面完成真实的UI渲染之后。因此它的执行是异步的，并且不会阻塞浏览器。
   1. 与类组件中的componentDidMount和componentDidUpdate的区别
   2. componentDidMount和componentDidUpdate，更改了真实Dom，但是用户还没有看到UI更新
   3. useEffect中的副作用函数，更改了真实Dom，并且用户已经看到了UI更新，异步的
2. 每个函数组件中，可以多次使用useEffect，但不要放入判断或循环等代码块中
3. useEffect中的副作用函数，可以有返回值，返回值必须是一个函数，该函数叫做清理函数
   1. 该函数运行时间点，在每次运行副作用函数之前
   2. 首次渲染组件不会运行
   3. 组件被销毁时一定会运行
4. useEffect函数，可以传递第二个参数
   1. 第二个参数是一个数组
   2. 数组中记录该副作用的依赖数据
   3. 当组件重新渲染后，只有依赖数据与上一次不一样的时，才会执行副作用
   4. 所以，当传递了依赖数据之后，如果数据没有发生变化
      1. 副作用函数仅在第一次渲染后运行
      2. 清理函数仅在卸载组件后运行
5. 副作用函数中，如果使用了函数上下文中的变量，则由于闭包的影响，会导致副作用函数中变量不会实时变化

#### 自定义hook

- 自定义hook更像是一种约定，而不是一种功能。如果函数的名字以`use`开头，并且调用了其他的hook，则就称其为一个自定义hook
- 自定义hook可以在不增加组件的情况下复用一些逻辑，达到高阶组件或者渲染属性的效果。
- hook是一种复用状态逻辑的方式，它不复用state本身
-  hook的每次调用都有一个完全独立的state

```javascript 
import React, { useEffect, useState } from 'react';
import ReactDOM from 'react-dom';

function useNumber(){
  let [number,setNumber] = useState(0);
  useEffect(()=>{
    setInterval(()=>{
        setNumber(number=>number+1);
    },1000);
  },[]);
  return [number,setNumber];
}
// 每个组件调用同一个 hook，只是复用 hook 的状态逻辑，并不会共用一个状态
function Counter1(){
    let [number,setNumber] = useNumber();
    return (
        <div><button onClick={()=>{
            setNumber(number+1)
        }}>{number}</button></div>
    )
}
function Counter2(){
    let [number,setNumber] = useNumber();
    return (
        <div><button  onClick={()=>{
            setNumber(number+1)
        }}>{number}</button></div>
    )
}
ReactDOM.render(<><Counter1 /><Counter2 /></>, document.getElementById('root'));
```
自定义hook必须以use开头。不遵循的话就无法判断某个函数是否包含对其内部hook的调用，React将无法自动检查自定义hook是否违反了hook的规则。

1.为什么需要函数式组件？它解决了什么问题？React底层又是如何区分两者？  
这一点可以由上面写的类组件的不足所得出，当类组件本身没有状态和其他副作用方法时，在render执行的时候，为了保证访问到的props就是render执行时的那份数据，会利用闭包来访问props。此时类组件中所有逻辑部分都定义在render内部，而不是作为类的实例方法，所以就没有必要浪费性能去定义一个需要实例化的类组件。在无状态组件中，函数组件的性能比类组件的性能要高，因为类组件使用的时候要实例化，而函数组件直接执行函数取返回结果即可。

2.为什么有了函数式组件还推出了React Hooks？它又解决了什么问题？  
因为函数组件是没有实例化的，所以本身没有状态和生命周期，react hook就是为了解决函数组件的状态和生命周期问题。

3.划分组件时出于什么考虑来选择不同的组件类型？  
本身应根据组件是纯功能组件还是有状态组件来划分，但是react hook完善之后应该都能用函数组件替代类组件。