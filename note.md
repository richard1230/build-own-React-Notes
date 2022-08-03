
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
