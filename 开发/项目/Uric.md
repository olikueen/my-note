# 前端初始化

### 安装vue-cli

```bash
npm install -g @vue/cli
```

### 创建项目

```bash
vue create <project_name>

选 babel, vuex, router
选 Vue3
选 In package.json

cd <project_name>
npm install
npm run serve
```

### 使用router

src/router/index.js

```js
const routes = [
    {
        path: '/show_center',
        name: 'show_center',
        component: () => import(/* webpackChunkName: "about" */ '../views/ShowCenter.vue')
    }
]
```

src/App.vue

```vue
<template>
  <nav>
    <router-link to="/show_center">展示中心</router-link>
  </nav>
  <router-view/>
</template>
```

src/views/ShowCenter.vue

```vue
<template>
  <h3>展示中心</h3>
</template>

<script>
import axios from "axios";
export default {
  name: "ShowCenter",
  mounted() {
    alert(this.$settings.host)
  }
}
</script>

<style scoped>

</style>
```

### 引入axios

```bash
npm install axios
```

### 引入全局settings

1. 创建 src/settings.js

   ```js
   export default {
       host: "127.0.0.1:8000"
   }
   ```

2. 在 src/main.js 中注册settings.js

   ```js
   // createApp(App).use(store).use(router).mount('#app')
   
   import settings from "@/settings"
   // @ 代表src
   
   const app = createApp(App)
   app.use(store).use(router).mount('#app')
   app.config.globalProperties.$settings = settings
   ```

3. 使用settings.js, 在src/views/ShowCenter.vue中, 使用 `this.$settings.变量` 调用

   ```js
   <script>
   import axios from "axios";
   export default {
     name: "ShowCenter",
     mounted() {
       alert(this.$settings.host)
     }
   }
   </script>
   ```

### 引入ant-design-vue

```bash
npm install ant-design-vue@next
```

**注册 ant-design-vue**, src/main.js

```js
import Antd from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'
import settings from "@/settings"

const app = createApp(App)
app.use(store).use(router).use(Antd).mount('#app')
app.config.globalProperties.$settings = settings
```

**使用**, src/views/ShowCenter.vue

```vue
<template>
  <h3>展示中心</h3>
  <a-button type="primary">Primary Button</a-button>
</template>

<script>
import axios from "axios";
export default {
  name: "ShowCenter",
  mounted() {
    // alert(this.$settings.host)
  }
}
</script>

<style scoped>

</style>
```

### 引入echarts

```bash
npm install echarts
```

注册, src/main.js

```js
let echarts = require('echarts');
app.config.globalProperties.$echarts = echarts;
