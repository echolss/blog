# 在create-react-app中配置按需加载
虽然此文档针对的是按需加载antd组件的样式，但此方法同样适用于antd-mobile。另外，配置了按需加载，你可以在写react代码时，将所有react组件的样式都写在一个css文件里。
# 配置步骤（在项目目录下）
1. 运行yarn add antd，安装antd
2. 运行 yarn run eject 命令将所有脚手架内建的配置暴露出来
3. 运行yarn add babel-plugin-import --dev，安装这个用于按需加载组件代码和样式的 babel 插件
4. 将package.json文件的如下代码

```
"babel": {
  "presets": [
    "react-app"
  ]
},
```
替换成：

```
"babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd",
          "style": "css"
        }
      ]
    ]
  },
```
5. 在react组件代码中引入antd组件：

```
import React, { Component } from 'react';
import { Button } from 'antd';

class App extends Component {
  render() {
    return (
      <div>
        <Button type="primary">Button</Button>
      </div>
    );
  }
}

export default App;
```
6. 重新运行yarn start就可以看到效果了