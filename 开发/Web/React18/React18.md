# React 18

- React基础
  - 工程化环境创建
  - JSX
  - 组件
  - useState
  - useEffect
  - useRef
  - 自定义Hook
  - 智能组件和UI组件
- Redux
  - RTK
  - 同步状态
  - 异步状态
  - 状态和视图分离
  - 美团购物车案例
- Router
  - 基础使用
  - 嵌套路由
  - 路由模式
  - 声明式导航
  - 编程式导航
  - 记账本案例

## 1 React基础

### 1.1 开发环境搭建

```bash
npx create-react-app <project name>
```

```bash
PS E:\my-project\react> npx create-react-app my-react-01

Creating a new React app in E:\my-project\react\my-react-01.

Installing packages. This might take a couple of minutes.
Installing react, react-dom, and react-scripts with cra-template...


added 1480 packages in 22s

251 packages are looking for funding
  run `npm fund` for details

Initialized a git repository.

Installing template dependencies using npm...

added 63 packages, and changed 1 package in 3s

251 packages are looking for funding
  run `npm fund` for details
Removing template package using npm...


removed 1 package in 1s

251 packages are looking for funding
  run `npm fund` for details

Created git commit.

Success! Created my-react-01 at E:\my-project\react\my-react-01
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm run eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd my-react-01
  npm start

Happy hacking!
npm notice
npm notice New major version of npm available! 9.6.6 -> 10.8.2
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.8.2
npm notice Run npm install -g npm@10.8.2 to update!
npm notice
```



