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



### 参考文章

[「技术雷达」之使用 Enzyme 测试 React（Native）组件](https://zhuanlan.zhihu.com/p/24494839)

[React 测试入门教程](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)

[Enzyme笔记](http://blog.leanote.com/post/haitang.reg@qq.com/Enzyme%E7%AC%94%E8%AE%B0)

