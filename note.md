
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
function render(element,container) {
    const dom = 
        element.type == "TEXT_ELEMENT"
    ? document.createTextNode('')
            : document.createElement(element.type)
    //排除 children 属性
    const isProperty = key => key!== "children"
    
    //将元素一一 写入 dom节点上面
    Object.keys(element.props)
        .filter(isProperty)
        .forEach(name => {
            dom[name] = element.props[name]
        })
    //遍历递归,将子元素一个一个都附到 真实的 dom 节点上面
    element.props.children.forEach(child =>
     render(child,dom)
    )
    
    //最后挂载到 指定的 dom 节点上面
    container.appendChild(dom)
}
```














