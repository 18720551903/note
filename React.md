# React Hooks



> 为什么类组件用的好好，react 官方又搞了一个什么 hooks，并且还想让函数组件替代类组件呢？



**写法对比**

Class 版本

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



React Hooks 版本

```javascript
import { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

可以让代码变得更简单



**复用性对比**

render props

```javascript
import Child from 'Child'
class PlacementProvider extends React.Component {
  constructor(props) {
    super(props);
    this.state = { placement: 'top' };
  }
  
  change = (dir) => {
    this.setState({
      placement: dir,
    })
  }

  render() {
    return (
      <div>
        {this.props.content({placement: this.state.placement, handleChange: change})}
      </div>
    )
  }
}

function Child(props) {
  const { placement, handleChange } = props;
  return (
  	<select value={placement} onChange={(e) => { handleChange(e.currentTarget.value) }}>
    	<option value="top">top</option>
      <option value="bottom">bottom</option>
      <option value="left">left</option>
      <option value="right">right</option>
    </select>
  )
}

<PlacementProvider content={data => (
  <Child placement={data.placement} handleChange={data.handleChange} />
)}/>

```



高阶组件

```javascript
import React from 'react';

const withPlacement = WrappedComponent => {
  return (
    class W extends React.Component {
      constructor(props) {
        super(props);
        this.state = { placement: 'top' };
      }
      
      change = (dir) => {
        this.setState({
          placement: dir,
        })
      }
      
      render() {
        return <WrappedComponent placement={this.state.placement} handleChange={this.change} />
      }
    }
  )
};

export default withPlacement(Child);
```



Hooks

```javascript
import React, { useState } from 'react';

function usePlacement() {
  const [placement, setPlacement] = useState('top');
  
  return {
    placement,
    setPlacement,
  };
}

function Child() {
  const {placement, setPlacement} = usePlacement();
  return (
  	<select value={placement} onChange={(e) => { setPlacement(e.currentTarget.value) }}>
    	<option value="top">top</option>
      <option value="bottom">bottom</option>
      <option value="left">left</option>
      <option value="right">right</option>
    </select>
  )
}

export default Child;
```

可以减少代码层级关系，没有多余的层级嵌套



**副作用处理对比**

Class 版本

```javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    // 改变 title
    document.title = `You clicked ${this.state.count} times`;
    // 发送 ajax
    request('/api/get')
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
    request('/api/get')
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



Hooks 版本

```javascript
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似于componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 改变 title
    document.title = `You clicked ${count} times`;
  });
  
   useEffect(() => {
     // 发送 ajax
    request('/api/get');
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

没有生命周期函数，并且可以让每个 effect 的职责单一



总结起来，React Hooks 出现的原因主要就两种：

1. 解决了类组件经常被诟病的一些问题
2. 拥抱函数式编程，让我们的代码变得复用性更高，更加简洁



##3. 基础Hook

###3.1 State Hook

```javascript
// 返回一个 state，以及更新 state 的函数。
const [state, setState] = useState(initialState); //初始值

// 初始化值
const [count, setCount] = useState(0); 

// 它不仅可以是一个值，也可以是一个函数
const [state, setState] = useState(() => { return initialState + 1 });

// 连续声明
const [count, setCount] = useState(0);
const [text, setText] = useState('my text'); 
const [xxx, setXxx] = useState([{ age: 18 }]); 

// 函数式更新
setCount(prevCount => prevCount + 2);

// 跳过更新
setCount(2);
setCount(2);
```



注意：

1. 声明的名称叫做 xxx,那么必定有一个值叫做 setXxx()
2. 初始值只在第一次加载起作用，在后续的渲染中，useState 返回的值将始终是更新后最新的 state
3. useState 必须在函数组件的最外层使用，不可以加在任何条件或者嵌套内
4. React 使用 Object.is 来比较 state，如果 state 没有变化，则不会触发更新



###3.2 Effect Hook

```javascript
// 该 Hook 接收一个包含副作用函数，componentDidMount 与 componentDidUpdate 的集合
useEffect(() => {
  document.title = `You clicked ${count} times`;
});

// 清除 effect, componentWillUnmount
useEffect(() => {
  const timer = setInterval(() => { console.log('timer') }, 1000);
  return () => {
    clearInterval(timer);    
  };
});

// effect 的条件执行 当count发生改变时执行
useEffect(() => {
  document.title = `You clicked ${count} times`;

useEffect(() => {   //componentDidMount 和 componentWillUnmount时执行
  document.title = `You clicked ${count} times`;
     return () => {
        console.log('离开')
      };
}, []);
```



注意：

1. useEffect 是异步的，将在每轮渲染结束后执行
2. 清除 effect 会在每次重新渲染执行
3. useEffect 要么不返回，要么返回一个方法， componentWillUnmount时执行
4. 可以通过设置第二个参数优化性能， 为空时componentDidMount和componentWillUnmount才执行， 传参表示参数改变即执行， 不传表示state改变即执行


###3.3 useContext

#### 了解React.createContext

```js
//context.js
import React from 'react'
const ThemeContext = React.createContext() //可传参为默认值
export const ThemeProvider = ThemeContext.Provider
export const ThemeConsumer = ThemeContext.Consumer

//index.js 是需要位于Consumer组件的上层
import { ThemeProvider } from './context'
render() {
  const { theme } = this.state
  return (
  <ThemeProvider value={{ themeColor: theme }}>
    <Page/>
  </ThemeProvider>
  )

// title.js 
import React from 'react'
import { ThemeConsumer } from './context'
class Title extends React.Component {
  render() {
    return <ThemeConsumer>
      {
        theme => <h1 style={{ color: theme.themeColor }}>
          title
        </h1>
      }
    </ThemeConsumer>
  }
```

#### useContext的用法

```js
onst themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);   //初始值

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>          //初始值被覆盖
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

### 3.4 useReducer

1. 语法： const [state, dispatch] = useReducer(reducer, initialArg, init);

2.  initialArg和init参数

   - 你可以选择惰性地创建初始 state。为此，需要将 `init` 函数作为 `useReducer` 的第三个参数传入，这样初始 state 将被设置为 `init(initialArg)`。
   - 这么做可以将用于计算 state 的逻辑提取到 reducer 外部，这也为将来对重置 state 的 action 做处理提供了便利：

   ```js
   function init(initialCount) {
     return {count: initialCount};
   }

   function reducer(state, action) {
     switch (action.type) {
       case 'increment':
         return {count: state.count + 1};
       case 'decrement':
         return {count: state.count - 1};
       case 'reset':
         return init(action.payload);
       default:
         throw new Error();
     }
   }

   function Counter({initialCount}) {
     const [state, dispatch] = useReducer(reducer, initialCount, init);
     return (
       <>
         Count: {state.count}
         <button
           onClick={() => dispatch({type: 'reset', payload: initialCount})}>
           Reset
         </button>
         <button onClick={() => dispatch({type: 'decrement'})}>-</button>
         <button onClick={() => dispatch({type: 'increment'})}>+</button>
       </>
     );
   }
   ```

3. 用例

   ```js
   const initialState = {count: 0};
   
   function reducer(state, action) {
     switch (action.type) {
       case 'increment':
         return {count: state.count + 1};
       case 'decrement':
         return {count: state.count - 1};
       default:
         throw new Error();
     }
   }
   
   function Counter() {
     const [state, dispatch] = useReducer(reducer, initialState);
     return (
       <>
         Count: {state.count}
         <button onClick={() => dispatch({type: 'decrement'})}>-</button>
         <button onClick={() => dispatch({type: 'increment'})}>+</button>
       </>
     );
   }
   ```



### 3.5 useCallback

- 语法：useCallback(fn, deps) 


-  该回调函数仅在某个依赖项改变时才会执行

- 返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 回调函数

- 用例

  ```js
  const memoizedCallback = useCallback(
    () => {
      doSomething(a, b);
    },
    [a, b],
  );
  ```

- useCallback(fn, deps) 相当于 useMemo(() => fn, deps)。

  - 共同： 仅仅 `依赖数据` 发生变化, 才会重新计算结果，也就是起到缓存的作用。
  - 区别：
    1.`useMemo` 计算结果是 `return` 回来的值, 主要用于 缓存计算结果的值 ，应用场景如： 需要 计算的状态
    2.`useCallback` 计算结果是 `函数`, 主要用于 缓存函数，应用场景如: 需要缓存的函数，因为函数式组件每次任何一个 state 的变化 整个组件 都会被重新刷新，一些函数是没有必要被重新刷新的，此时就应该缓存起来，提高性能，和减少资源浪费。

### 3.6 useMemo

- 返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 值。


- 把“创建”函数和依赖项数组作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

  ```
  const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  ```

- 用例

  不使用memo

  ```JS
  import React from 'react';
   
  export default function WithoutMemo() {
      const [count, setCount] = useState(1);
      const [val, setValue] = useState('');
   
      function expensive() {
          console.log('compute');
          let sum = 0;
          for (let i = 0; i < count * 100; i++) {
              sum += i;
          }
          return sum;
      }
   
      return <div>
          <h4>{count}-{val}-{expensive()}</h4>
          <div>
              <button onClick={() => setCount(count + 1)}>+c1</button>
              <input value={val} onChange={event => setValue(event.target.value)}/>
          </div>
      </div>;
  }
  ```

  使用memo

  ```JS
  
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
          <h4>{count}-{expensive}</h4>
          {val}
          <div>
              <button onClick={() => setCount(count + 1)}>+c1</button>
              <input value={val} onChange={event => setValue(event.target.value)}/>
          </div>
      </div>;
  }
  ```

  上面我们可以看到，使用useMemo来执行昂贵的计算，然后将计算值返回，并且将count作为依赖值传递进去。这样，就只会在count改变的时候触发expensive执行，在修改val的时候，返回上一次缓存的值。


##4. 创建自己的 Hooks

自定义 Hook 更像是一种约定，而不是一种功能。如果函数的名字以 **use** 开头，并且调用了其他的 Hook，则就称其为一个自定义 Hook。



提取自定义 Hooks  

- 函数名以use开头, 并且调用了hook(useState或者useEffect)

```javascript
const useFetchData = (filmId) => {
  const [loading, setLoading] = useState(true);
  const [data, setData] = useState({});
  
  useEffect(() => {
    setLoading(true);
    fetch(`https://swapi.co/api/films/${filmId}`)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [filmId]);

  return [loading, data];
};
```



使用自定义 Hook

```javascript
function App({ filmId }) {
  const [loading, data] = useFetchData(filmId);

  if (loading === true) {
    return <p>Loading ...</p>;
  }

  return (
    <div>
      <p>电影名称: {data.title}</p>
      <p>导演: {data.producer}</p>
      <p>发布日期: {data.release_date}</p>
    </div>
  );
}
```



**Hook 是一种复用状态逻辑的方式（是即高阶组件 和 render props 后的另外一种增加复用性的方式）**

优势：自定义 Hook 可以让我们在不增加组件的情况下达到同样的目的。



##5. 额外的 Hooks

**useReducer**

```javascript
const initialState = { number: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {number: state.number + 1};
    case 'decrement':
      return {number: state.number - 1};
    case 'awesome':
      const number = state.number * state.number + Math.random();
      return { number };
    default:
      throw new Error();
  }
}

function Counter(){
    const [state, dispatch] = useReducer(reducer, initialState);
    return (
        <>
          Count: {state.number}
					<button onClick={() => { setState({ number: state.number + 1 }) }}>+</button>
					<button onClick={() => { setState({ number: state.number - 1 }) }}>-</button>
 					<button onClick={() => dispatch({type: 'increment'})}>+</button>
          <button onClick={() => dispatch({type: 'decrement'})}>-</button>
        </>
    )
}

```

在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。



注意：

**他们的作用都是帮我们声明一个状态，useState 适合声明单一状态，而 useReducer 适合处理多状态的场景。**



**useContext**

作用：接受 React.createContext 返回的值，并返回该 Context 的 value。

```javascript
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```

注意：

useContext 的参数必须是 **context 对象本身**

只要 Context.Provider 的 value 发生变化，使用到 useContext 的组件就会触发更新



**useRef**

使用 ref 访问组件的 DOM

```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```



使用 uesRef 创建函数组件内的私有变量（类似于 class 里的 this 对象上的属性）

```javascript
function Example() {
  const intervalRef = useRef();

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(timer);
    });
    intervalRef.current = timer;
    return () => {
      clearInterval(intervalRef.current);
    };
  });
}
```

注意：

useRef 的值需要放到 current 上

useRef 的 current 值改变，不会引发组件更新



[React Hooks 文档](https://zh-hans.reactjs.org/docs/hooks-reference.html)





## 6. 使用 Hooks 的注意事项

**只在最顶层使用 Hook**



**不要在普通的 JavaScript 函数中调用 Hook

# React-redux

1. 安装相关包文件

   ```
   npm i redux 
   npm i react-redux
   ```

2. 创建action.js文件, 存放action

   ```js
   //此处action只是为了举例，是最简单的方式，在后面会记录更多方式创建
   //此处的Test便是一个action生成函数，返回的是一个对象，具有type属性与其他附带的值
   export const Name= (payload) => ({
        type: 'NAME',
        payload   //此处为调用action生成函数时候传的参数
   })
   export const Rank = () =>({
       type: 'RANK'
   })
   ```

3. 创建reducer.js文件, 存放reducer

   ```javascript

   //引入action
   import action from './action.js'
   import {combineReducers} from 'redux'
   //tip  此处的函数形参必须要有默认值，否则会报错
   function getName(state = '',action){
       //此处的形参不是完整的state了，而是我们单独操作的那个状态值，也就是后面state对象中的name
       //因为combineReducers的作用，它会生成一个state对象，包含每一个reducer处理后返回的值
       switch(action.type){
           case 'NAME':
               state = action.payload
               return state
           default:
               return state
       }
   }
    
   function changeRank(state = 0,action){
       switch(action.type){
           case 'RANK':
               state++
               return state
            default:
               return state
      }
   }

   const reducer = combineReducers({
       name: getName,
       rank: changeRank
   })

   export default reducer
   ```

4. 创建store.js, 生成store对象

   ```js
   import {createStore} from 'redux'
   import reducer from './reducer'
   const store = createStore(reducer)
   export default store
   ```

5. 在主入口js中引入相关store，这里已index.js为例

   ```javascript
   import React from 'react'
   import ReactDOM from 'react-dom'
   import App from './index' //组件app
   import {Provider} from 'react-redux'  //用来装载store，使store渗入各个组件
   import store from './store' //创建的store的位置

   ReactDOM.render(
       <Provider store={store}>
           <App/>
       </Provider>,
       document.getElementById('root')
   );
   ```

6. 在组件中使用redux

   ```javascript
   import { connect } from 'react-redux'
   import {Name} from '../../redux/actions'
   
   let Content = (props) => {
     const {value, getName} = props;
     return(
       <div>
         redux 结合 hook
         显示{value}
         <div onClick={() => {getName)}}></div>
       </div>
     )
   }
   const mapStateToProps = state => {
       return {
           value:state
       }
   }
   mapDispatchToProps => dispatch => {
       return {
           getName: () => dispatch({type:'RANK'});
       }
   }
   export connect()(Content)
   ```

# React中的路由

## 路由配置

1. jsx

```html
import { Router, Route, Redirect } from 'react-router'
 <Router>
    <Route path="/" component={App}>
      <IndexRoute component={Dashboard} />   //设置默认页面
      <Route path="about" component={About} />
      <Redirect from="messages/:id" to="/messages/:id" /> //重定向
      <Route path="inbox" component={Inbox}> //嵌套路由
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
```

2. 配置

   ```js
   const routeConfig = [
     { path: '/',
       component: App,
       indexRoute: { component: Dashboard },
       childRoutes: [
         { path: 'about', component: About },
         { path: 'inbox',
           component: Inbox,
           childRoutes: [
             { path: '/messages/:id', component: Message },
             { path: 'messages/:id',
               onEnter: function (nextState, replaceState) {
                 replaceState(null, '/messages/' + nextState.params.id)
               }
             }
           ]
         }
       ]
     }
   ]
   
   React.render(<Router routes={routeConfig} />, document.body)
   ```

## 路由跳转

1. 标签跳转

   ```
   import {Link} from 'react-router-dom'
   <Link to="/home"></Link>
   ```

2. js跳转

   ```
   class MyComponent () {
   	jump(){
   		this.props.history.push('/home)									this.props.history.push({path:'/home',query:{name:'zs})
   	}
   	render()=> <p onClick={this.jump}>跳转</p>
   }
   export default MyComponent;
   ```

