### 安装组件库

```cmd
cnpm i ant-design-vue moment -S
```

### 引入并全局注册ant-design-vue

````js
// main.js
import Antd from "ant-design-vue";

Vue.use(Antd);
````

### 引入less样式并配置vue.config.js

````js
// main.js
import "ant-design-vue/dist/antd.less";

// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      less: {
        modifyVars: {
          // antd主题色配置
          "primary-color": "#FA541C"
        },
        javascriptEnabled: true
      }
    }
  }
};
````

### 高扩展性路由设计

```js
const routes = [
  {
    path: "/user",
    // component: { render: h => h("router-view") },
    component: () =>
      import(/* webpackChunkName: "layout" */ "./layouts/UserLayout"),
    children: [
      {
        path: "/user",
        redirect: "/user/login"
      },
      {
        path: "/user/login",
        name: "login",
        component: () =>
          import(/* webpackChunkName: "user" */ "./views/User/Login")
      },
      {
        path: "/user/register",
        name: "register",
        component: () =>
          import(/* webpackChunkName: "user" */ "./views/User/Register")
      }
    ]
  },
  {
    path: "/",
    component: () =>
      import(/* webpackChunkName: "layout" */ "./layouts/BasicLayout"),
    children: [
      // dashboard
      {
        path: "/",
        redirect: "/dashboard/analysis"
      },
      {
        path: "/dashboard",
        name: "dashboard",
        component: { render: h => h("router-view") },
        children: [
          {
            path: "/dashboard/analysis",
            name: "analysis",
            meta: { title: "分析页" },
            component: () =>
              import(
                /* webpackChunkName: "dashboard" */ "./views/Dashboard/Analysis"
              )
          }
        ]
      },
      // form
      {
        path: "/form",
        name: "form",
        component: { render: h => h("router-view") },
        children: [
          {
            path: "/form/basic-form",
            name: "basicform",
            component: () =>
              import(/* webpackChunkName: "form" */ "./views/Forms/BasicForm")
          },
          {
            path: "/form/step-form",
            name: "stepform",
            component: () =>
              import(/* webpackChunkName: "form" */ "./views/Forms/StepForm"),
            children: [
              {
                path: "/form/step-form",
                redirect: "/form/step-form/info"
              },
              {
                path: "/form/step-form/info",
                name: "info",
                component: () =>
                  import(
                    /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step1"
                  )
              },
              {
                path: "/form/step-form/confirm",
                name: "confirm",
                component: () =>
                  import(
                    /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step2"
                  )
              },
              {
                path: "/form/step-form/result",
                name: "result",
                component: () =>
                  import(
                    /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step3"
                  )
              }
            ]
          }
        ]
      }
    ]
  },
  {
    path: "*",
    name: "404",
    component: NotFound
  }
];
```

### 安装路由动画 Nprogress

```js
// 使用导航守卫实现动画效果
import NProgress from "nprogress";
import "nprogress/nprogress.css";

NProgress.configure({ showSpinner: false }); //不要转圈，丑

router.beforeEach((to, from, next) => {
  NProgress.start();
  next();
});

router.afterEach(() => {
  NProgress.done();
});
```

### 基本页面布局

````vue
<template>
  <div>
    <a-layout id="components-layout-demo-side" style="min-height: 100vh">
      <a-layout-sider v-model="collapsed" collapsible>
        <div class="logo" />
        <SiderMenu />
      </a-layout-sider>
      <a-layout>
        <a-layout-header style="background: #fff; padding: 0">
          <Header />
        </a-layout-header>
        <a-layout-content style="margin: 0 16px">
          <router-view></router-view>
        </a-layout-content>
        <a-layout-footer style="text-align: center">
          <Footer />
        </a-layout-footer>
      </a-layout>
    </a-layout>
  </div>
</template>

<script>
import Header from "./Header";
import Footer from "./Footer";
import SiderMenu from "./SiderMenu";
export default {
  data() {
    return {
      collapsed: false
    };
  },
  components: {
    Header,
    Footer,
    SiderMenu
  }
};
</script>

<style></style>
````

### 隐藏菜单栏默认trigger并添加自定义trigger

````vue
<!-- 隐藏trigger-->
<a-layout-sider :trigger="null" v-model="collapsed" collapsible>
    <div class="logo" />
    <SiderMenu />
</a-layout-sider>

<!-- 添加自定义trigger-->
 <a-layout-header style="background: #fff; padding: 0">
    <a-icon
        class="trigger"
        :type="collapsed ? 'menu-unfold' : 'menu-fold'"
        @click="collapsed = !collapsed"
        ></a-icon>
    <Header />
</a-layout-header>

<style scoped>
.trigger {
  padding: 0 20px;
  line-height: 64px;
  font-size: 20px;
}
.trigger:hover {
  background-color: #eeeeee;
}
</style>

````

### 添加Drawer组件用于主题配置，通过路由传值

````vue
<!-- SettingDrawer.vue-->
<template>
  <div>
    <a-drawer
      placement="right"
      :closable="false"
      :visible="visible"
      @close="onClose"
      width="300px"
    >
      <template v-slot:handle>
        <div class="handle" @click="visible = !visible">
          <a-icon :type="visible ? 'close' : 'setting'"></a-icon>
        </div>
      </template>

      <div>
        <h3>整体风格定制</h3>
        <a-radio-group
          :value="$route.query.navTheme || 'dark'"
          @change="e => handleSettingChange('navTheme', e.target.value)"
        >
          <a-radio value="dark">
            黑色
          </a-radio>
          <a-radio value="light">
            白色
          </a-radio>
        </a-radio-group>
        <h3>导航模式</h3>
        <a-radio-group
          :value="$route.query.navLayout || 'left'"
          @change="e => handleSettingChange('navLayout', e.target.value)"
        >
          <a-radio value="left">
            左侧
          </a-radio>
          <a-radio value="top">
            顶部
          </a-radio>
        </a-radio-group>
      </div>
    </a-drawer>
  </div>
</template>
<script>
export default {
  data() {
    return {
      visible: false
    };
  },
  methods: {
    handleSettingChange(type, value) {
      this.$router.push({ query: { ...this.$route.query, [type]: value } });
    },
    onClose() {
      this.visible = false;
    }
  }
};
</script>
<style scoped>
.handle {
  position: absolute;
  top: 240px;
  right: 300px;
  width: 48px;
  height: 48px;
  background-color: #fa541c;
  color: #fff;
  font-size: 20px;
  text-align: center;
  line-height: 48px;
  border-radius: 4px 0 0 4px;
}
</style>

````

