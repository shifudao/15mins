## 什么是Ionic

> Ionic提供了一个免费且开源的移动优化HTML，CSS和JS组件库，来构建高交互性应用。基于Sass构建和AngularJS 优化。

### Ionic的特点
- 追求性能
  - 操作最少的 DOM，非 jQuery，和硬件加速过渡
- AngularJS 和 Ionic
  - 与AngularJS完美融合
- 专注原生
  - 以流行的原生移动开发SDK为蓝本, 通过Cordova build
- 漂亮的设计
  - 众多流行移动组件，结构，交互规范，以及华丽的（且可扩展）的主题
- 强大的命令行
  - 创建，构建，测试，部署
- 简单易学
  - html,css,js,基本的angularjs语法

## Ionic与原生应用有何不同
- Native App
  - 优点
    - 用户体验度高
    - 流畅的交互性
    - 性能稳定
    - 访问本地资源
    - 访问硬件方便
  - 缺点
    - 分发成本高
    - 维护成本高
    - 更新缓慢
- Web App
  - 优点
    - 开发成本低
    - 迭代更新快
    - 适配多种设备
    - 无需安装
  - 缺点
    - 无法调用系统功能
    - 用户体验度差
- Hybrid App
  - 通过WebView实现Native App与Web App的一种混合
  
![](http://image.uisdc.com/wp-content/uploads/2014/12/duibi.png)


## 为何要选用Ionic
- 跨平台
  - 支持ios android windowsPhone
- 开发效率高, 开发成本低
- 面向用户特殊

## Ionic安装及使用
先安装cordova
> npm install -g cordova

其次安装ionic
> npm install -g ionic

配置android ios环境
> android需要java 和 android SDK

> ios需要Mac, 并安装xcode

创建ionic项目
> ionic start myApp blank(tabs sidemenu)

添加平台
> ionic platform add ios/android

编译
> ionic build android/ios

运行（真机，虚拟机）
> ionic run android / ionic emulate ios

### app.js路由配置
```js
var nameApp = angular.module('starter', ['ionic', 'ui.router']);

nameApp.config(function($stateProvider, $urlRouterProvider) {
  $stateProvider
    .state('login', {
      url: '/login',
      templateUrl: 'login.html',
      controller: 'LoginCtrl'
    })
    .state('view', {
      url: '/view',
      templateUrl: 'view.html',
      controller: 'ViewCtrl'
    });
  $urlRouterProvider.otherwise("/login");
});
```
### 页面
```html
<ion-view view-title="First page">  
    <ion-content class="padding">
      <div class="list">
        <label class="item item-input">
          <input type="text" placeholder="Enter your first name" ng-model="input.firstName">
        </label>
        <label class="item item-input">
          <input type="text" placeholder="Enter your last name" ng-model="input.lastName">
        </label>
      </div>        
      <a href="#/view" style="text-decoration: none;"><button class="button button-full button-positive">
        登录
      </button></a>      
    </ion-content>
</ion-view>
```

### controller
```js
nameApp.controller('LoginCtrl', function($scope, Authorization) {
  $scope.input = Authorization;  
});

nameApp.controller('ViewCtrl', function($scope, Authorization) {
  $scope.input = Authorization;
});
```

### service
```js
nameApp.factory('Authorization', function() {
  authorization = {};
  authorization.firstName = "";
  authorization.lastName = "";  
  return authorization;
});
```

## Cordova插件系统
> Cordova提供了一系列插件进行辅助开发, 如照相机,相册,通讯录,GPS,BLE.

### 什么是ngCordova
> ngCordova是在Cordova Api基础上封装的一系列开源的AngularJs服务和扩展，让开发者可以方便的在HybridApp开发中调用设备能力，即可以在AngularJs代码中访问设备能力Api。

ngCordova安装
> bower install ngCordova

将ng-cordova.js添加到index.html中的cordova.js之前
```js
<script src="lib/ngCordova/dist/ng-cordova.js"></script>
<script src="cordova.js"></script>
```
通过Ionic添加插件
> ionic plugin add ...

### cordova插件自定义开发
[开发插件](https://cordova.apache.org/docs/en/latest/guide/hybrid/plugins/)

## Ionic2
> ionic 2基于angular 2（由于angular2处于beta阶段需要typescript的支持，ionic 2目前仍然处于测试版）。ionic 2几乎修改了第一版中的每一个模块，开发者上手将更加简单

- 语法更为简洁
- 样式分别针对ios android windowsPhone进行优化

[Ionic2](http://ionicframework.com/docs/v2/)
