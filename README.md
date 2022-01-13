
接入步骤
主引用
一. npm install qiankun -S
二. main.js 引入
import {
  registerMicroApps,
  addGlobalUncaughtErrorHandler,
  start
} from 'qiankun'
1. 注册子应用
registerMicroApps([
{ 
        name: 'sonVue',  // 微应用名称
        entry: '//localhost:10200', // 加载入口
        container: '#son-client', // 挂在节点，会把微应用挂在到当前节点
        activeRule: '/client'  // 微应用匹配的路由匹配规则，凡是包括/client都会开启微应用页面
      }
], 
{
// qiankun 生命周期钩子 - 加载前
  beforeLoad: (app) => {
    // 加载子应用前，加载进度条
    console.log('before load', app.name)
    return Promise.resolve()
  },
  // qiankun 生命周期钩子 - 挂载后
  afterMount: (app) => {
    // 加载子应用前，进度条加载完成
    console.log('after mount', app.name)
    return Promise.resolve()
  }
})
2. 全局异常捕捉
addGlobalUncaughtErrorHandler((event) => {
  console.error(event)
  const { message: msg } = event
  // 加载失败时提示
  if (msg && msg.includes('died in status LOADING_SOURCE_CODE')) {
    console.log('子应用加载失败，请检查应用是否可运行')
  }
})
3.启动函数
start()
四. 设置挂在显示点,
Appmain.vue中加入
<div id="son-client"></div>    // 这里的id 就是 之前注册微应用的挂在点

五. 路由配置
{
    path: '/client/form',  // 匹配 /client
    component: Layout,
    children: [
      {
        path: 'index',
        name: 'Profile',
        meta: { title: '客户端-表单', icon: 'dashboard', affix: true }
      }
    ]
  },

微应用
一. vue.config.js文件中添加允许跨域
headers: {
      "Access-Control-Allow-Origin": "*",
    },
二. 打包输出
output: {
      // 微应用的包名，这里与主应用中注册的微应用名称一致
      library: "sonVue",
      // 将你的 library 暴露为所有的模块定义下都可运行的方式
      libraryTarget: "umd",
      // 按需加载相关，设置为 webpackJsonp_VueMicroApp 即可
      jsonpFunction: `webpackJsonp_sonVue`,
    },
三. main.js
1. 导出微应用出口
/**
 * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
 * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
 */
export async function bootstrap() {
}

/**
 * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
 */
export async function mount(props) {
  render(props)
}

/**
 * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
 */
export async function unmount() {
  instance.$destroy()
  instance.$el.innerHTML = ''
  instance = null
  router = null
}

2. 改造 创建vue
function render(props) {

  router = new VueRouter({
    mode: 'history',
    routes: constantRoutes,
// 设置路由基础路径  当是微应用形式的话 加上/client, 正常打开的项目还是默认的
    base: window.__POWERED_BY_QIANKUN__ ? '/client' : '/' 
  })

  instance = new Vue({
    el: '#app',
    router,
    store,
    render: h => h(App)
  })
}

3. // 独立运行时，直接挂载应用
if (!window.__POWERED_BY_QIANKUN__) {
  render()
}

4. 
if (window.__POWERED_BY_QIANKUN__) {
  // 动态设置 webpack publicPath，防止资源加载出错
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__
}