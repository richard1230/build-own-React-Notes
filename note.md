## JSX是如何被解析的

```jsx
const element = <h1 title="foo">Hello</h1>;
```

会被 babel 解析为如下代码:

```jsx
const element = React.createElement("h1", {title: "foo"}, "Hello");
```

接下来实现一个简单的 createElement,用于生成虚拟 DOM:

```jsx

function createElement(type, props, ...children) {

    return {
//元素类型
        type,
//元素的属性
        props: {
            ...props,
            //子元素
            children: children.map(
                child =>
                    typeof child === "object"
                        ? child
                        : createTextElement(child)
            ),
        },
    }
}


function createTextElement(text) {

    return {
        type: "TEXT_ELEMENT",
        props: {
            nodeValue: text,
            children: []
        }
    }
}



```

## 渲染到真实的dom节点上面(React16之前的逻辑:用的是递归遍历的方式)

```js
function render(element, container) {
    const dom =
        element.type == "TEXT_ELEMENT"
            ? document.createTextNode('')
            : document.createElement(element.type)
    //排除 children 属性
    const isProperty = key => key !== "children"

    //将元素一一 写入 dom节点上面
    Object.keys(element.props)
        .filter(isProperty)
        .forEach(name => {
            dom[name] = element.props[name]
        })
    //遍历递归,将子元素一个一个都附到 真实的 dom 节点上面
    element.props.children.forEach(child =>
        render(child, dom)
    )

    //最后挂载到 指定的 dom 节点上面
    container.appendChild(dom)
}
```

## 并发模式（向React16诞生的缘由）

到目前为止,已经实现一个比较简单的React了，可以把JSX渲染到dom上面,但是有一个问题,
就是Render函数是使用递归来实现将子元素一一附着到dom上的,如果节点的层级比较多,节点很多的话,就有可能长时间占用浏览器进程,造成阻塞，

影响浏览器更高优先级别的事物处理（用户的输入和UI交互）.

因此,需要将这个大的任务切割分为多个小的工作单元,这样的话,如果浏览器拥有更高优先级别的事物处理,我们就会中断React元素的渲染;我们称之为“并发模式”；

```js
let nextUnitofWork = null;

function workloop(deadline) {
    //是否要暂停
    let shouldYield = false;

    while (nextUnitofWork && !shouldYield) {
        //执行下一个工作单元  并返回下一个工作单元
        nextUnitofWork = performUniteofWork(nextUnitofWork)
        //判断空闲时间是否足够,如果剩余时间小于1ms(几乎没有剩余时间了)，块钱跳出循环
        shouldYield = deadline.timeRemaining() < 1
    }

    requestIdleCallback(workloop)
}

requestIdleCallback(workloop)

function performUnitofWork() {
    // TODO
}

```



















