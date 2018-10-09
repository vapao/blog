---
title: CSS Modules in React
tags:
- css modules
- react
---

想创建一个美观的React应用不可避免的要使用CSS，但CSS在近几年前端飞速发展的过程中并没有什么改变。在小型的项目中并没有什么问题，但当项目足够大之后会就浮现了其弊端，让我们先来看看需要解决哪些问题。

### CSS的问题

CSS本身语法是非常简单且灵活的，但这些特性在大项目中反而会成为问题。

#### 全局作用域

在大多数语言中全局变量在使用时都是需要慎重的，而且我们React应用中的CSS样式通常也是针对具体的组件设置的，所以CSS的全局作用域不是一个好的特性。

受CSS的全局作用域影响，很容易通过`class`来覆盖样式，在开发中通常没有人希望发生某个`class`中做的改变最终可能会默默影响多个组件的UI。

#### 代码冗余

很多时候我们的应用都会有一些冗余的或不再使用的样式。虽然最初他们可能很小，但随着应用的更新我们会到处看到不用的CSS样式，但又不能直接删除，应用会为这些不必要的数据而增加体积。

通过使用`CSS Modules`可以只导入自己使用的样式，这样就很容易的分辨哪些是无用的。

#### 条件语句

有时候我们希望一个`class`应用在不同地方有不同的样式，这是如果可以有个CSS的`class`变量能在组件里修改就好了。

### 使用CSS Modules

在React里模块化使用CSS有两个方法：

- 使用javascript对象
- 使用CSS Modules

#### JavaScript对象实现

在React中把CSS写成javascript对象很容易，类似正常的css只是把命名变为驼峰格式。

例如：

```jsx
// lets have some CSS in js: style.css.js
const button = {
    backgoundColo: '#202020',
    padding: '12px',
    borderRadius: '2px',
    width: '100%'
}
export default {button}

// importing and using
import styles from './style.css.js'

const button = () => {
    return <button style={styles.button}>A Button</button>
}
```

#### 使用CSS Modules

如果使用的`create-react-app`来创建的项目，可以通过安装`react-app-rewired`来覆盖默认配置来启用，或者通过`npm eject`来自己配置。这里给出通过安装`react-app-rewired`来覆盖配置的方法：

```js
// create file at project root path named config-overrides.js
// source: node_modules/react-scripts/config/webpack.config.dev.js
module.exports = function override(config, env) {
  config['module']['rules'][1]['oneOf'][2]['use'][1]['options'] = {
    importLoaders: 1,
    modules: true,
    localIdentName: '[name]__[local]__[hash:base64:5]'
  };
  return config
};
```

修改配置需要重新运行`npm start`生效。

我们来看下启用CSS Modules之后的代码：

```jsx
// lets have some CSS: style.css
.button {
    background-color: #202020;
    padding: 12px;
    border-radius: 2px;
    width: 100px;
}

// importing and using in a jsx
import styles from './style.css'
import React from 'react';

class Button extends React.Component {
    render() {
        return (
            <button className={styles.button}>A Button</button>
        )
    }
}
export default Button;
```

> 参考来源：https://blog.pusher.com/css-modules-react/



