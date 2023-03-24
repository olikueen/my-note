### 1. 创建vite项目

```shell
# 设置淘宝源
npm config set registry https://registry.npm.taobao.org

# 创建空项目
npm init vite-app 项目名称

# 安装依赖
cd 项目名称
npm install

# 运行项目
npm run dev
```

### 2. 目录结构

```shell
node_modules			第三方依赖包
pubilc					公共静态资源
	favicon.ico
src						项目源代码
	assets					存放项目中所有的静态资源文件(css fonts)
	commponents				存放项目中所有自定义组件
    App.vue					项目根组件
    index.css				全局样式表文件
    main.js					项目打包
.gitigonre
index.html				单页面应用入口
package.json			项目包管理配置文件
package-lock.json
```

