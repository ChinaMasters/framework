# 二开框架说明文档
为提高开发效率，约束开发规范，公司提供基于vue-cli3的二开框架以便于快速开发。因框架正迭代维护中，如有不当之处，欢迎指正，谢谢！

### 安装epgis脚手架

`epgis`脚手架是基于公司内网开发的，必须先设置内网npm镜像源：

> npm set registry http://172.20.20.57:4873

接着全局安装脚手架

> npm i -g epgis

### 下载二开框架

执行如下命令创建二开项目， `project-name` 为要创建的项目名称，在项目创建过程中会自动执行 `npm install` 安装项目所需依赖

> epgis create project-name
 
注意，如果在 `powershell` 执行命令提示 `提示在此系统禁止运行脚本`，直接执行 `epgis.cmd create project-name`, 或者切换控制台直接执行 ` epgis create project-name`

### 二开项目启动发布

启动
> npm run dev

发布
> npm run build

### 二开项目使用

**项目目录结构**

具体的使用规范，在每个目录都有对应的规范说明文档，请参照文档进行开发。如下是项目的主要目录树状结构，及其功能说明，对一些vue脚手架自带功能不做过多描述。

```
├─public
|  ├─favicon.ico
|  └index.html     // 模版页面，提供全局变量config配置keys秘钥、地图参数等
├─src
|  ├─api           // api模块，案例请参考user.js
|  |  ├─axios.js 
|  |  ├─pis.js    
|  |  └user.js
|  ├─components
|  |  ├─mixin
|  |  ├─map                          // 这边是地图核心功能，主要封装了gis2.0拓扑图和pis网架
|  |  |  ├─legend                    // 图例
|  |  |  ├─compass                   // 指北针
|  |  |  ├─baseLayerController       // 图层控制器
|  |  |  |  ├─common.less            // pis和gis公共样式
|  |  |  |  ├─index.vue              // 功能页面入口
|  |  |  |  ├─pis.vue                // pis独立模块（后期删除）
|  |  |  |  ├─tool                   // 地图js工具操作
|  |  |  |  |  ├─config.js           // 地图渲染要素配置
|  |  |  |  |  ├─custom.js           // 自定义图层
|  |  |  |  |  ├─MapBaseServer.js    // gis2.0请求封装
|  |  |  |  |  ├─mapModel.js         // 地图模版
|  |  |  |  |  ├─note.js             // gis2.0注释标注
|  |  |  |  |  ├─singleRack.js       // pis简图网架js
|  |  |  |  |  └topolopyRack.js      // gis2.0拓扑图js
|  |  ├─login-form
|  |  ├─dv          // dv为常用边框盒子组件，具体效果可以查看demo页面
|  |  ├─count-to    // count-to为数字滚动器，具体效果可以查看demo页面
|  ├─libs           // 常用公共函数封装模块 
|  |  ├─contants.js
|  |  ├─epgisUtil.js
|  |  ├─excel.js
|  |  ├─MapManager.js // 此处封装地图的公共操作方法，并提供图层容器，以便进行统一图层管理
|  |  ├─README.md
|  |  ├─render-dom.js
|  |  └util.js
|  ├─locale   // 多语言
|  ├─mock     // mock模拟数据
|  ├─plugin   // vue插件
|  ├─router   // 路由
|  ├─static   // 静态资源
|  ├─store    // vuex分模块仓库
|  ├─style    // 公共样式
|  ├─view     // 页面
|  |  ├─module
|  |  |  └page.vue  // 通用页面，可以把该页面作为模版替换开发
|  |  ├─login       // 登陆页面
|  |  ├─example     // 案例
|  ├─App.vue
|  ├─index.less
|  ├─main.js
├─.babelrc
├─.editorconfig
├─.eslintignore
├─.eslintrc.js
├─.gitignore
├─.postcssrc.js
├─.travis.yml
├─cypress.json
├─LICENSE
├─package.json
├─README.md
├─vue.config.js  // vue通用配置，主要注意地图请求公共地址配置和proxy代理配置

```

**思极地图使用注意事项**

- keys秘钥配置在 `public/index.html` 文件下的config配置中，默认设置为全局变量，以便于上线之后能在控制台中实时操作地图

`epgisKeys` 为思极地图秘钥，`geoglKeys` 为三维库秘钥

- 思极地图内外网地址配置在 `vue.config.js` 文件下

`EPGIS_URL` 为思极地图地址，`GOGL_URL` 为三维库地址

- 在`app.vue`中实例初始化`MapManager`类

- `src/lib/MapManager.js`文件提供了全局变量mapCenter，统一管理思极地图和geogl三维库，建议详细阅读该源码。
注意这边的地图配置将覆盖 `public/index.html` 中的config配置。

`app.vue` 中对地图进行初始化。

```
import MapManager from './libs/MapManager'

// 为了方便调用将mapCenter挂载全局
window.mapCenter = new MapManager({
	zoom: 16,
	pitch: 0,
	center: [119.33187240031535, 26.080261865846516]
})

// 其中window.mapCenter.map就是我们要获取的全局map地图对象
console.log(mapCenter.map)

```

- 地图异步操作

```
// 为了确保地图加载完，请将地图相关内容写在defer回调函数里面
window.mapCenter.defer(() => {
  // 地图相关内容写在这个回调函数里面
})
```
- 地图图层操作

具体案例可以查看`view/example/demo/line.vue`页面, 通过`mapCenter.addLayer`将所有图层必须添加到`mapCenter._layers`容器中，便于统一对图层进行显隐删除等操作，`mapCenter.addLayers`会自动添加对应图层id。 

```
// 图层添加, 如果onAdd属性为空时，单纯添加{id，renderType}对象到图层队列。
mapCenter.addLayer({
	id: 'line_demos',
	renderType: 'EPGIS'
	onAdd: fn
})

// 更新图层数据
mapCenter.map.getSource('id').setData({
	type: 'FeatureCollection',
	features: Feature
})

// 销毁图层
mapCenter.removeByKeys('id')

```

- 网架

`gis2.0`拓扑图和`pis`网架简图的代码位于 `src/components/map/index`
具体案例在 `http://127.0.0.1:8080/#/rack` 
