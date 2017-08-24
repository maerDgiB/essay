### React单元测试（Jest  + Enzyme）

### 前提：

使用`create-react-app`生成一个项目。使用默认的`Jest`的单元测试的工具，但是引入`Enzyme`帮助之后更容易去断言、操作`React`组件的输出。

**注意：**如果仅引入`enzyme`在`npm test`会报错，需要安装依赖`react-addons-test-utils`，再次run的时候会出现一个问题，bash里会有一个warning，需要更新组件，

```bash
 Warning: ReactTestUtils has been moved to react-dom/test-utils. Update references to remove this warning.
```

查过`issue`发现需要安装同样版本的`react-test-renderer`，这样就可以消除warning了。这些依赖都是安装在devDependencies`里面。同样版本号的意思是：`package.json`的版本号有大致相同如：`^15.6.x`。

### 单元测试

#### 浅渲染 [shallow()](https://github.com/airbnb/enzyme/blob/master/docs/api/shallow.md)   shallow(node[, options]) => ShallowWrapper

> Shallow rendering is useful to constrain yourself to testing a component as a unit, and to ensure that your tests aren't indirectly asserting on behavior of child components.
>
> 浅渲染在将一个组件作为一个单元进行测试的时候非常有用，可以确保你的测试不会去间接断言子组件的行为。shallow 方法只会渲染出组件的第一层 DOM 结构，其嵌套的子组件不会被渲染出来，从而使得渲染的效率更高，单元测试的速度也会更快。

```JavaScript
import {shallow} from 'enzyme'

describe('Enzyme Shallow', () => {
  it("Home the first p text",()=>{
        // const wrapper = shallow(<Home />)
        const title = "11111111"
        // 测试Home组件下的第一个<p> 是不是 title
        // expect 对象是 Jest的，API找官方文档
        expect(wrapper.find("p").first().text()).toEqual(title);
    })
}
```

#### 深度渲染 [mount()](https://github.com/airbnb/enzyme/blob/master/docs/api/mount.md)   mount(node[, options]) => ReactWrapper

> mount 方法则会将React组件渲染为真是的DOM节点，特别是在你需要必须依赖真实的DOM结构的情况下，或是测试组件的全部的生命周期。
>
> 完全的DOM渲染需要全局提供全部的DOM API。这就意味着需要跑在一个至少看起来像浏览器的环境下。如果你不想在浏览器中测试，推荐一个使用mount的方法是依赖一个jsdom的库，它本质上是一个完全在 JavaScript 中实现的 headless 浏览器。

```javascript
import { mount } from 'enzyme'

describe('Enzyme Mount', () => {
  it('should delete Todo when click button', () => {
    const app = mount(<App />)
    const todoLength = app.find('li').length
    app.find('button.delete').at(0).simulate('click')
    expect(app.find('li').length).to.equal(todoLength - 1)
  })
})
```

#### 静态渲染 [render()](https://github.com/airbnb/enzyme/blob/master/docs/api/render.md)   render(node[, options]) => CheerioWrapper

> render 方法则会将 React 组件渲染成静态的 HTML 字符串，返回的是一个 Cheerio 实例对象，采用的是一个第三方的 HTML 解析库 Cheerio，官方的解释是「我们相信 Cheerio 可以非常好地处理 HTML 的解析和遍历，再重复造轮子只能算是一种损失」。这个 CheerioWrapper 可以用于分析最终结果的 HTML 代码结构，它的 API 跟 shallow 和 mount 方法的 API 都保持基本一致。

```javascript
import { render } from 'enzyme'
describe('Enzyme Render', () => {
  it('Todo item should not have todo-done class', () => {
    const app = render(<App />)
    expect(app.find('.todo-done').length).to.equal(0)
    expect(app.contains(<div className="todo" />)).to.equal(true)
  })
})
```

### 使用react-redux & immutable

在这样的情况下，如果单元测试的组件中含有一个`connect`函数包裹的组件，建议使用浅渲染的模式去测试，当然可以使用深度渲染，但是一定会报错，查后可得你一定要包裹一层的`<Provider>`还要引入`store`

```javascript
// 这个结构才能经得住三种渲染

	import Home from './Home/index';
	const Connectednode = <Provider store={store}><Home /></Provider>
    const Connectedwrapper = shallow(
        Connectednode
    )

    it('Home component render 12', () => {
        const div = document.createElement('div');
        ReactDOM.render(Connectednode, div);
        // logs <Connect(Home) />
        //console.log(Connectedwrapper.debug())
    });
```

这样的方式是可是渲染connect函数返回的组件，但是我只想测试**原本的Home**组件，只需要将Home组件一同`export`出来，方法如下：

```javascript
// ./Home/index.js
export class Home extends React.component{}
export default connect()(Home)
// test.js
import  ConnectedHome, {Home} from './Home/index';
```

当然，可能测试的Home组件会依赖store的部分的state以及dispatch方法。如果单元测试中没有提供对应的数据和方法则会报错，第一种方法就是原封不动的将数据和方法传入给组件，就像这样：

```javascript
	const state = store.getState()
    const props = {
           addResult:state.getIn(['HomeReducer']),
           dispatch:store.dispatch
    }
    const node = <Home {...props} />
    const wrapper = shallow(node)
```

第二种方式，是根据对单元测试概念测出来的，因为是单元测试，而且是对每一个组件进行测试，所以只要提供相对应的数据和方法就好了，仅仅是将这个组件看为一个模块测试其的功能是否走的通便好，所以这样处理`props`对象，如下：

```javascript
 import {Map} form "immutable"
 const props = {
           // 如果你的项目里用了immutable 
           addResult:Map({data:999999}),
           dispatch:()={}
    }
```

这样就可以测试了。

### 三种测试

#### 浅渲染（建议使用）

对测试的组件只要提供对应的数据方法就行。

#### 深度渲染（明白坑点在使用）

因为是DOM结构的渲染，连同子组件一同的渲染出来（各个渲染的概念要多看几遍），所以一定要提供所有的依赖。

#### 静态渲染（不建议使用）

因为返回的是Cheerio对象，能够分析静态HTML结构但是怎么样分析自行google，因为emzyme这里用的就是第三方库，而且渲染完的`wrapper`能使用的API并不多，因为已经是第三方的东西，目前最大的BUG。

#### 建议

可以分别把同样的组件三种不同的形式渲染一下

```javascript
const Connectednode = <Provider store={store}><ConnectedHome /></Provider>
const Shallowwrapper = shallow(Connectednode)
const  mountWrapper = mount(Connectednode)
const renderWrapper = render(Connectednode)
console.log("shallow",Shallowwrapper.debug())
console.log("mount",mountWrapper.debug())
console.log("render",renderWrapper)
```




### 参考文章

[「技术雷达」之使用 Enzyme 测试 React（Native）组件](https://zhuanlan.zhihu.com/p/24494839)

[React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)

[Enzyme笔记](http://blog.leanote.com/post/haitang.reg@qq.com/Enzyme%E7%AC%94%E8%AE%B0)

[Writing Tests Redux](http://redux.js.org/docs/recipes/WritingTests.html)

[Emzyme](http://airbnb.io/enzyme/index.html)

