## React Hooks

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee08e3abecda4bd181184c6b6690ee6c~tplv-k3u1fbpfcp-watermark.image)

#### useState原理

**使用状态：**
```
const [n,setN] = React.useState(0);
const [user,setUser] = React.useState({name:'F'})
```

**useState用法**

```
import React from "react";
import ReactDOM from "react-dom";
const rootElement = document.getElementById("root");

function App() {
  const [n, setN] = React.useState(0);
  return (
    <div className="App">
      <p>{n}</p>
      <p>
        <button onClick={() => setN(n + 1)}>+1</button>
      </p>
    </div>
  );
}

ReactDOM.render(<App />, rootElement);
```
点击Button之后发生了什么

首次渲染` render <App/>`,调用`App()`(每次调用App(),都会运行useState(0)) 得到虚拟div，创建真实div。

用户点击`button`,调用`setN()n+1) `再次`render<App/>`.

调用App(),得到虚拟DIV，DOM Diff.更新真实DIV。

**分析**

* setN
setN一定会修改数据x,将n+1存入x；
setN一定会触发`<App/>`重新渲染(re-render);

* useState
useState肯定会从x读取n的最新值；

* x
每个组件都有自己的数据x,我们将其命名为state;

**注意事项：**

如果state是一个对象，那么就不能部分setState;</br>
setState(obj)如果obj地址不变，那么React就会认为数据没有变化。

```
useState接收函数：
const [state,setState] = useState(()=>{
   return initialState
})
该函数返回初始化state,且只执行一次；

setState接受函数：
setN(i => i+1)}

```

#### useReducer

**实践操作Flux/Redux的思想：**

一：创建初始值initialState;</br>
二：创建所有操作reducer(state,action);</br>
三：传给useReducer,得到读和写API；</br>
四：调用写({type:'操作类型'})

```
举例：
import React, { useReducer } from "react";
import ReactDOM from "react-dom";

const initFormData = {
  name: "",
  age: 18,
  nationality: "汉族"
};

function reducer(state, action) {
  switch (action.type) {
    case "patch":
      return { ...state, ...action.formData };
    case "reset":
      return initFormData;
    default:
      throw new Error();
  }
}

function App() {
  const [formData, dispatch] = useReducer(reducer, initFormData);
  // const patch = (key, value)=>{
  //   dispatch({ type: "patch", formData: { [key]: value } })
  // }
  const onSubmit = () => {};
  const onReset = () => {
    dispatch({ type: "reset" });
  };
  return (
    <form onSubmit={onSubmit} onReset={onReset}>
      <div>
        <label>
          姓名
          <input
            value={formData.name}
            onChange={e =>
              dispatch({ type: "patch", formData: { name: e.target.value } })
            }
          />
        </label>
      </div>
      <div>
        <label>
          年龄
          <input
            value={formData.age}
            onChange={e =>
              dispatch({ type: "patch", formData: { age: e.target.value } })
            }
          />
        </label>
      </div>
      <div>
        <label>
          民族
          <input
            value={formData.nationality}
            onChange={e =>
              dispatch({
                type: "patch",
                formData: { nationality: e.target.value }
              })
            }
          />
        </label>
      </div>
      <div>
        <button type="submit">提交</button>
        <button type="reset">重置</button>
      </div>
      <hr />
      {JSON.stringify(formData)}
    </form>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/908158bd29d0481598eba82648f51af7~tplv-k3u1fbpfcp-watermark.image)

**如何代替Redux**

* 步骤：

一：将数据集中在一个store对象；</br>
二：将所有操作集中在reducer;</br>
三：创建一个Context;</br>
四：创建对数据的读写API；</br>
五：将第四步的内容放到第三步的Context</br>
六：用Ccontext.Provider将Context提供给所有组件；</br>
七：各个组件用useContext获取读写API</br>

```
import React, { useReducer, useContext, useEffect } from "react";
import ReactDOM from "react-dom";
import User from "./components/user";
import Context from "./Context";
import Books from "./components/books";
import Movies from "./components/movies";
import userReducer from "./reducers/user_reducer";
import booksReducer from "./reducers/books_reducer";
import moviesReducer from "./reducers/movies_reducer";

const store = {
  user: null,
  books: null,
  movies: null
};

const obj = {
  ...userReducer,
  ...booksReducer,
  ...moviesReducer
};

function reducer(state, action) {
  const fn = obj[action.type];
  if (fn) {
    return fn(state, action);
  } else {
    console.error("wrong type");
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, store);

  const api = { state, dispatch };
  return (
    <Context.Provider value={api}>
      <User />
      <hr />
      <Books />
      <Movies />
    </Context.Provider>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

// 帮助函数

```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5408224de0fd49bb88f4fa7a801b1685~tplv-k3u1fbpfcp-watermark.image)


#### useContext

**上下文：**

全局变量是全局的上下文；</br>
上下文是局部的全局变量；

**使用方法：**

一：使用C = createContext(initial);</br>
二：使用<C.provider>圈定作用域；</br>
三：在作用域内使用useContext(C)来使用上下人</br>

```
import React, { createContext, useState, useContext } from "react";
import ReactDOM from "react-dom";

import "./styles.css";

const C = createContext(null);

function App() {
  console.log("App 执行了");
  const [n, setN] = useState(0);
  return (
    <C.Provider value={{ n, setN }}>
      <div className="App">
        <Baba />
      </div>
    </C.Provider>
  );
}

function Baba() {
  const { n, setN } = useContext(C);
  return (
    <div>
      我是爸爸 n: {n} <Child />
    </div>
  );
}

function Child() {
  const { n, setN } = useContext(C);
  const onClick = () => {
    setN(i => i + 1);
  };
  return (
    <div>
      我是儿子 我得到的 n: {n}
      <button onClick={onClick}>+1</button>
    </div>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/382f805aa998421a9087346058fc447a~tplv-k3u1fbpfcp-watermark.image)

**useContext注意事项：**

* 不是响应式：

在一个模块将C里面的值改变，另一个模块无法感知到这个变化；

#### useEffect

**副作用：**

副作用指的是该API会对环境做出改变，例如修改document.title。但我们不一定要把副作用放在useEffect，实际上叫做afterRender更好，每次render后运行；

**用途：**

作为componentDidMount使用。[]作第二个参数;</br>
作为componentDidUpdate使用，可指定依赖;</br>
作为componentWillUnmount使用;通过return;</br>
以上三种用途可同时存在；</br>

**特点:**

如果同时存在多个useEffect,会按照出现次序执行；

#### useLayoutEffect

**布局副作用：**

useEffect在浏览器渲染完成后执行；</br>
useLayoutEffect在浏览器渲染前执行；</br>

**特点：**

useLayoutEffect总是比useEfffect先执行；</br>
useLayoutEffect里的任务最后影响了Layout；

#### useMemo

**特点**

第一个参数是()=>value;</br>
第二个参数是依赖[m,n];</br>
只有当依赖变化时，才会重新计算出新的value.如果依赖不变，那么就重用之前的value.</br>

#### useCallback

**用法：**

`useCallback(x => log(X),[m] `</br>
等价于</br>
`seMemo(() => x => log(x),[m])`

#### useRef

**目的**

如果需要一个值，在组件不断render时保持不变；</br>
初始化:const count = useRef(0);</br>
读取：count.current;</br>
为了保证两次useRef是同一个值，所以需要current;</br>

在函数组件中使用`String Ref、Callback Ref/Create Ref`会抛出以下错误：

`Uncaught Invariant Violation: Function components cannot have refs. Did you mean to use React.forwardRef()?`

这是因为函数组件没有实例，所以函数组件无法使用`String Ref、Callback Ref、Create Ref`,取而代之的是useRef.

**useRef的作用：**

获取DOM元素的节点；</br>
获取子组件的实例；</br>
渲染周期之间共享数据的存储(state不能存储跨渲染周期的数据，因为state的保存会触发组件重渲染)；

**获取DOM元素的节点**
```
import React, { useEffect, useRef } from 'react';
function App() {
  const h1Ref = useRef();
  useEffect(() => {
    console.log('useRef')
    console.log(h1Ref.current)
  }, [])
  return <h1 ref={h1Ref}>Hello World!</h1>
}
export default App;

//打印结果：
useRef
   <h1>Hello World!</h1>
```

**获取子组件的实例；**

**渲染周期之间共享数据的存储**

#### forwarsRef

**props无法传递ref属性：**

```
import React, { useRef } from "react";
import ReactDOM from "react-dom";

import "./styles.css";

function App() {
  const buttonRef = useRef(null);
  return (
    <div className="App">
      <Button2 ref={buttonRef}>按钮</Button2>
      {/* 看浏览器控制台的报错 */}
    </div>
  );
}

const Button2 = props => {
  return <button className="red" {...props} />;
};

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```

**实现ref的传递：**

```
import React, { useRef } from "react";
import ReactDOM from "react-dom";

import "./styles.css";

function App() {
  const buttonRef = useRef(null);
  return (
    <div className="App">
      <Button3 ref={buttonRef}>按钮</Button3>
    </div>
  );
}

const Button3 = React.forwardRef((props, ref) => {
  return <button className="red" ref={ref} {...props} />;
});

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```

**两次ref传递得到button的引用：**

```
import React, { useRef, useState, useEffect } from "react";
import ReactDOM from "react-dom";

import "./styles.css";

function App() {
  const MovableButton = movable(Button2);
  const buttonRef = useRef(null);
  useEffect(() => {
    console.log(buttonRef.curent);
  });
  return (
    <div className="App">
      <MovableButton name="email" ref={buttonRef}>
        按钮
      </MovableButton>
    </div>
  );
}

// function Button2(props) {
//   return <button {...props} />;
// }

const Button2 = React.forwardRef((props, ref) => {
  return <button ref={ref} {...props} />;
});

// 仅用于实验目的，不要在公司代码中使用
function movable(Component) {
  function Component2(props, ref) {
    console.log(props, ref);
    const [position, setPosition] = useState([0, 0]);
    const lastPosition = useRef(null);
    const onMouseDown = e => {
      lastPosition.current = [e.clientX, e.clientY];
    };
    const onMouseMove = e => {
      if (lastPosition.current) {
        const x = e.clientX - lastPosition.current[0];
        const y = e.clientY - lastPosition.current[1];
        setPosition([position[0] + x, position[1] + y]);
        lastPosition.current = [e.clientX, e.clientY];
      }
    };
    const onMouseUp = () => {
      lastPosition.current = null;
    };
    return (
      <div
        className="movable"
        onMouseDown={onMouseDown}
        onMouseMove={onMouseMove}
        onMouseUp={onMouseUp}
        style={{ left: position && position[0], top: position && position[1] }}
      >
        <Component {...props} ref={ref} />
      </div>
    );
  }
  return React.forwardRef(Component2);
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);

```

#### 尝试实现React.useState

```
import React from "react";
import ReactDOM from "react-dom";
const rootElement = document.getElementById("root");

let _state;

function myUseState(initialValue) {
  _state = _state === undefined ? initialValue : _state;
  function setState(newState) {
    _state = newState;
    render();
  }
  return [_state, setState];
}

// 教学需要，不用在意 render 的实现
const render = () => ReactDOM.render(<App />, rootElement);

function App() {
  const [n, setN] = myUseState(0);
  return (
    <div className="App">
      <p>{n}</p>
      <p>
        <button onClick={() => setN(n + 1)}>+1</button>
      </p>
    </div>
  );
}

ReactDOM.render(<App />, rootElement);

```

#### 总结

* 每个函数组件对应一个React节点
* 每个节点保存着state和index 
* useState会读取state[index]
* index由useState出现的顺序决定
* setState会修改state,并触发更新

每次重新渲染，组件函数就会执行；</br>
对应的所有state都会出现[分身]；</br>
如果不希望出现分身，可以用useRef/useContext。</br>



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c2ea7fa75474b639e9cd27fa4db2bf4~tplv-k3u1fbpfcp-watermark.image)
