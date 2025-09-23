# RastarGame项目

## 特色亮点

1. **多租户架构** - 一套代码支持多个游戏官网
2. **自动设备适配系统** - 自动适配移动端和PC端
4. **性能优化** - 图片压缩、懒加载、网络请求优化、资源加载优化
4. c
5. **游戏化功能** - 集成抽奖、预约等游戏运营功能

```
src/
├── api/          # API接口层
├── assets/       # 全局资源
├── components/   # 通用组件
├── pages/        # 各游戏独立页面
│   ├── dpcq/     # 斗破苍穹
│   ├── cdjz/     # 超电竞战
│   ├── dmbj/     # 大明宝鉴
│   ├── qyn/      # 青云年
│   └── ro/       # RO仙境传说
└── util/         # 工具函数
```

## 技术实现

### 1. 多租户架构

#### 环境变量定义

```javascript
define: {
  'process.env.VITE_APP_ID': JSON.stringify(process.env.VITE_APP_ID),
}
```

- 将环境变量注入到前端代码中，供运行时使用

```json
"dpcq": "set VITE_APP_ID=dpcq && vite serve src/pages/dpcq/",
"cdjz": "set VITE_APP_ID=cdjz && vite serve src/pages/cdjz/",
"dmbj": "set VITE_APP_ID=dmbj && vite serve src/pages/dmbj/"
```

- 通过 `VITE_APP_ID` 环境变量区分不同租户（游戏）
- 每个租户有独立的启动命令

#### HTML 模板插件

```js
createHtmlPlugin({
  template: `src/pages/${process.env.VITE_APP_ID}/index.html`,
  filename: () => {
    return path.resolve(__dirname, `${process.env.VITE_APP_ID}/index.html`)
  },
})
```

**关键设计**：根据 `VITE_APP_ID` 动态选择不同游戏的 HTML 模板

- 例如：`VITE_APP_ID=dpcq` 时使用 `src/pages/dpcq/index.html`

#### 构建输出目录隔离

```javascript
build: {
  outDir: `dist/${process.env.VITE_APP_ID}`,
}
```

#### 文件复制插件

```js
copy({
  targets: [{
    src: `src/pages/${process.env.VITE_APP_ID}/check/*.txt`,
    dest: `dist/${process.env.VITE_APP_ID}`,
  }],
  hook: 'writeBundle',
})
```

- 复制每个游戏特有的验证文件（通常用于域名验证）
- 在构建完成后执行复制操作

#### 多租户优点

1. **代码复用**：共享核心功能和组件
2. **独立部署**：每个租户可以独立构建和部署
3. **配置隔离**：每个租户有独立的配置和路由
4. **资源隔离**：静态资源和构建产物完全分离
5. **开发效率**：一套代码维护多个产品

### 2. 自动设备适配系统

#### 设备检测机制

- 用户代理检测

- **位置：** `src/util/base.js` 中的 `client()` 函数

```js
export const client = () => {
  if (
    uaParser.device.type === 'mobile' ||
    uaParser.device.type === 'tablet' ||
    uaParser.device.type === 'wearable' ||
    uaParser.device.type === 'xr'
  ) {
    return '2'  // 移动端
  } else {
    return '1'  // PC端
  }
}
```

- 路由层面的设备检测

- **位置：** `src/pages/dpcq/router.js` 的路由守卫

  ```javascript
  router.beforeEach((to, from, next) => {
    const userAgent = navigator.userAgent.toLowerCase();
    const isMobile = /iphone|ipad|ipod|android|blackberry|mini|windows\sce|palm/i.test(userAgent);
    
    if (isMobile) {
      // 移动端访问PC路径时，自动跳转到移动端路径
      if (to.path == "/") {
        next({ path: "/m", query: to.query });
      } else if (to.path == "/home") {
        next({ path: "/home/m", query: to.query });
      }
    } else {
      // PC端访问移动端路径时，自动跳转到PC路径
      if (to.path == "/m") {
        next({ path: "/", query: to.query });
      } else if (to.path == "/home/m") {
        next({ path: "/home", query: to.query });
      }
    }
  });
  ```

#### 响应式布局

- REM 自适应方案

**位置：** `src/util/rem.js`

```javascript
function setFont() {
  var html = document.documentElement
  var k = ''
  if (client() === '2') {  // 移动端
    k = 750   // 基于750px设计稿
  } else {    // PC端
    k = 1920  // 基于1920px设计稿
  }
  html.style.fontSize = (html.clientWidth / k) * 100 + 'px'
}

doc.addEventListener('DOMContentLoaded', setFont, false)  // 页面加载完成
win.addEventListener('resize', setFont, false)           // 窗口大小改变
win.addEventListener('load', setFont, false)             // 资源加载完成

```

**工作原理：**

- **移动端**：以750px为基准，动态计算根字体大小
- **PC端**：以1920px为基准，动态计算根字体大小
- 页面元素使用rem单位，自动跟随根字体大小缩放

### 3. 性能优化

#### 图片压缩和格式转换

```js
// vite.config.js
imagemin({
  mode: 'sharp',
  compress: {
    jpeg: {
      // 0 ~ 100
      quality: 75,
    },
    png: {
      // 0 ~ 100
      quality: 75,
    },
    webp: {
      // 0 ~ 100
      quality: 75,
    },
  },
  conversion: [
    { from: 'png', to: 'webp' },
    { from: 'jpg', to: 'webp' },
  ],
  cache: false,
})
```

- 自动将 PNG/JPG 转换为更高效的 WebP 格式
- 压缩质量设置为 75%，平衡文件大小和图片质量
- 使用 Sharp 引擎进行高性能图片处理

#### 懒加载实现

#### 网络请求优化

- 请求重试和恢复机制

```javascript
// src/api/request.js
import { RequestRecovery } from 'xhwebtool'

// 请求拦截器 - URL重试机制
axios.interceptors.request.use(
  async (config) => {
    config.url = RequestRecovery.replaceUrl2RetryApi(config.url)
    // ... 其他处理
    return config
  }
)

// 响应拦截器 - 自动重试失败请求
axios.interceptors.response.use(
  async (response) => {
    const res = await RequestRecovery.fetch(response, axios)
    if (res.status === 200) {
      return res
    } else {
      return Promise.reject(res)
    }
  },
  async (error) => {
    const err = await RequestRecovery.fetch(error, axios)
    return Promise.reject(err)
  },
)
```

- 请求配置优化

```javascript
// 设置请求超时时间
axios.defaults.timeout = 5000
axios.defaults.withCredentials = true

// 封装带重试机制的请求
export const postFetch = async (url, params = {}, config = {}) => {
  const res = await axios.post(url, params, { 
    ...config, 
    recoveryType: 4  // 启用4次重试
  })
  // ... 处理响应
}
```

#### 资源加载优化

- 资源内联优化

```javascript
// vite.config.js
build: {
  // 小于 4KB 的资源自动内联为 base64，减少 HTTP 请求
  assetsInlineLimit: 4096,
  sourcemap: true,
}
```

- SDK 异步加载优化

```javascript
// src/util/loadSdk.js
export const loadSdk = (type, locale, appId) => {
  // 删除旧脚本避免重复加载
  const existingScripts = document.querySelectorAll('script[src*="connect.facebook.net"]')
  existingScripts.forEach((script) => script.parentNode.removeChild(script))

  // 添加时间戳防止缓存
  const timestamp = Date.now()
  const url = `https://connect.facebook.net/${locale}/sdk.js#xfbml=true&version=v11.0&appId=${appId}&autoLogAppEvents=1&_=${timestamp}`

  // 异步加载脚本
  ;(function (d) {
    var js, id = 'facebook-jssdk'
    js = d.createElement('script')
    js.id = id
    js.async = true  // 异步加载
    js.src = url
    d.getElementsByTagName('head')[0].appendChild(js)
  })(document)
}
```

#### 防抖优化

```javascript
// src/pages/ro/views/yygw/com/navWap.vue
// 滚动高亮显示业务
let scrollTimer = null
const handleScroll = () => {
  if (scrollTimer) return  // 防抖处理
  scrollTimer = setTimeout(() => {
    updateActiveMenuOnScroll()
    scrollTimer = null
  }, 100)
}
```