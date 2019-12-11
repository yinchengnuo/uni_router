# 欢迎使用 uni_router.js

对 uni-app 中的路由 api 进行了简单封装，定义了全局导航守卫方法、路由状态保存、错误捕获等。使用起来及其方便，无需配置路由表， 也无需进行二次封装。代码也不复杂，只有不到100行，比较适合中小型项目快速开发。很实用。如有BUG，还请不吝指出，非常感谢。

----

## 使用

```javascript
// 在 main.js 中注册
import Vue from 'vue'
import App from '@/App'
import $router, { $route } from '@/common/js/uni_router.js'

$router.beforeEach = (to, next) => { // 注册全局前置守卫
    console.log('全局前置守卫', to)
    if (to.path.includes('/test')) {
        // 可以通过传一个回调给 next 来访问 $router 实例, 会返回一个 reject('在全局前置守卫 next 中重定向路由')
        next(vm => {
            vm.push('/redirect')
        })
    } else if (to.path.includes('/redirect')) {
        next(false) //  中断当前的导航,会返回一个 reject('在全局前置守卫 next 中取消路由')
    } else {
        next() // 一定要调用该方法来 resolve 这个钩子
    }
}

$router.afterEach = to => { // 注册一个全局后置守卫
 console.log('全局后置守卫', to)
}

Vue.prototype.$route = $route // 当前路由对象，保存路由当前信息
Vue.prototype.$router = $router // 路由对象，保存了实例方法

// 在组件中使用
methods: {
    to() {
        this.$router.push('/test', {
            id: 2333,
            test: {
                a: 1,
                b: 2
            }
        }).then(e => {
            console.log('路由结束，当前路由对象为', e)
        }).catch(e => {
            console.warn(e)
        })
    }
}

// test 页面中
onLoad() {
    console.log(this.$route) // 可以通过 $route 获取当前路由对象、参数、路径信息等
    // {
    //     fullPath: "/pages/test/test"
    //     path: "/test"
    //     query: {
    //         id: 2333,
    //         test: {
    //             a: 1,
    //             b: 2
    //         }
    //     }
    // }
}

// this.$router 上一共有 5 个方法， 分别是
this.$router.pop()
this.$router.push()
this.$router.replace()
this.$router.reLaunch()
this.$router.switchTab()

// 除了 pop 方法只接受一个参数 delta 外，其余四个方法接受的参数分别是
fun(
    path, // path 路由名
    query = {}, // query 路由传参
    notBeforeEach, // isBeforeEach 是否要被全局前置守卫拦截,默认false，拦截，设置为 true 禁止拦截
    notAfterEach // isAfterEach 是否要被全局后置守卫拦截，默认false，拦截，设置为 true 禁止拦截
).then(...)

```
# 注意事项

1 . uni_router 的核心实现是利用了
```
require.context('@/pages', true, /\.vue$/)
```
是利用 require.context 将 pages 文件夹中的所有 .vue 页面组件统统导入，然后放在一个数组里，在使用的时候动态匹配，因为才可以不用配置路由。这样的好处和坏处都是显而易见的，用起来方便的同时。你不可以将页面路由放在 pages 以外的地方，不然读取不到。同时 pages 除了页面最好不要放组件。你应该在根目录下建一个 componetns 文件夹管理组件。

2 .  全局导航守卫并不能也没有对 pop 方法或者说所有的返回行为做监听。因为目前个人用 uni-app 开发几乎都是小程序方面，小程序没办法对返回行为做监听，所有就没做

3 . 可能不支持 nvue 。没测试过。具体原因看第二条。

----
