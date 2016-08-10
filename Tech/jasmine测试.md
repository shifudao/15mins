## 单元测试目的
- 验证功能逻辑性
- 帮助设计灵活、解耦合的类

## 什么是jasmine
> Jasmine is a behavior-driven development framework for testing JavaScript code. It does not depend on any other JavaScript frameworks. It does not require a DOM. And it has a clean, obvious syntax so that you can easily write tests. 

- 用于JS代码测试的行为驱动(BDD)开发的框架
- 独立
- 清晰简洁

## jasmine语法
1.四个核心概念：
- 分组
  - describe(string,function)﻿​任何一个测试用例以describe函数来定义,表示分组，即一组的测试用例
- 用例
  - it(string,function)﻿​表示测试用例,定义单个具体测试任务
- 期望
  - expect(expression)﻿​表示期望
- 匹配
  - to***(arg)﻿​表示匹配

2.以一个最简单的例子说明：
```js
describe("Group", function() {
  var a;
  it("sample", function() {
    a = true;
    expect(a).toBe(true);
  });
});
```
> describe和it都有两个参数，一个string和function。string为辅，描述大体内容，便于阅读；function真是的测试代码和方法。
> expect获取当前测试值，利用matcher与expected value进行匹配。

3.内置匹配函数举例
- expect(a).toBe(true);   //期望变量a的值为true
- expect(a).toBeDefined();   //期望a已被定义
- expect(a).toEqual(1);  //期望a的值为1
- expect(a).toBeNull();   //期望变量a为null
- expect(ionicLoading.show).toHaveBeenCalled();   //期望这个方法已经被调用过
- expect(user.goDetail).toHaveBeenCalledWith(1);  //期望在调用方法时传递了参数1

更多详情参考官方文档：http://jasmine.github.io/

## jasmine配合karma
使用jasmine在Karma上实现自动化。

1.安装Karma
- npm install -g karma

2.下载所需插件，以师傅到为例
- karma-jasmine
- angular-mocks
- karma
- karma-chrome-launcher
- karma-coverage
- karma-phantomjs-launcher
- phantomjs-prebilt

3.配置karma.config.js文件
- 利用指令初步配置  karma init
- 根据项目进行修改

```js
module.exports = function(config) {
    config.set({
        frameworks: ['jasmine'],                    //应用的测试框架
        files: [...],                               //测试环境需要加载的js信息
        preprocessors: {
        'www/airCare/**/*.js': ['coverage']
        }
        reporters: ['progress', 'coverage'],        //代码覆盖率配置
        coverageReporter: {
          type : 'html',
          dir : 'tests/coverage/'
       },
       autoWatch: true,                             //自动监听
       browsers: ['PhantomJS']                      //代码测试环境
       ...
    })
} 
```

## ng单元测试举例
需要测试的controller.js
```js
(function () {
    'use strict';
    angular
        .module('weeklyModule')
        .controller('WeeklyCtrl',WeeklyCtrl);

    WeeklyCtrl.$inject = ['$scope','weeklyService','$ionicLoading','$timeout'];
    function WeeklyCtrl($scope,weeklyService,$ionicLoading, $timeout){
        var total  = -1;
        var offset = 0;
        var vm = this;
        vm.weeklyListItems = [];
        vm.showOrHide = true;
        vm.doRefresh = doRefresh;
        
        function doRefresh() {
            vm.weeklyListItems = [];
            offset = 0;
            total  = -1;
            requestWeeklyListByUrl("scroll.refreshComplete", true);
        }
        function requestWeeklyListByUrl(broadMsg, isDoRefreshOrLoadMoreData){
            weeklyService.getWeeklyList(offset).then(function (data) {
                var records = data.records;
                if(isDoRefreshOrLoadMoreData){
                    vm.weeklyListItems = records.weeklyList;
                    total = parseInt(records.weeklyCount);
                    vm.showOrHide = (total > 0);
                }else{
                    addItems(records.weeklyList);
                }
                offset += records.weeklyList.length;
                $scope.$broadcast(broadMsg);
            }, function () {
                $scope.$broadcast(broadMsg);
            });
        }
        ...
    }
})();
```
测试代码编写

1.导入模块：
- 利用官方提供的测试工具类angular-mock.js，进行模块的定义、加载、注入等。
```js
describe('weeklyControllerSpec',function () {
    var controller;
    beforeEach(function () {
        angular.mock.module('weeklyModule');
    });
});
```
2.注入对应依赖：
- 在controller.js中，注入了$scope,weeklyService,$ionicLoading,$timeout。那么在测试中，同样需要把这些依赖给mock进来。
- 原因：要测试的doRefresh方法由WeeklyCtrl控制器定义在controller($scope)上，为了保证测试的有效性，需要创建新的WeeklyCtrl控制器、$scope、service等。实质就是手动做 ng-controller指令的工作。

3.关联被测对象：
- 将注入的依赖关联到本测试单元的controller上。
```js
describe('weeklyControllerSpec',function () {
    var controller, $httpBackend,$scope,ionicLoadingMock,weeklyService, $timeout;
    
    beforeEach(function () {
        angular.mock.module('weeklyModule');
        ionicLoadingMock = jasmine.createSpyObj('ionicLoadingMock spy',['show','hide']);

        angular.mock.module(function ($provide) {
            $provide.factory('weeklyService',function ($http,$cordovaToast,$q) {
                return {
                    getWeeklyList:     getWeeklyList
                };
                function getWeeklyList(offset) {
                    return $http.get('http://192.168.1.36:8080/aircare/user/myWeekly?max=10&offset=1')
                        .then(function (response) {
                            var data = response.data;
                            if(data.result == "success"){
                                return data;
                            }else{
                                $cordovaToast.showShortBottom(data.message);
                                return $q.reject();
                            }
                        });
                }
            });

            $provide.factory('$cordovaToast', function () {
                var showMsg = '';
                return {
                    showShortBottom: showShortBottom,
                    getShowMsg: getShowMsg
                };
                function showShortBottom(msg) {
                    showMsg = msg;
                    return msg;
                }
                function getShowMsg() {
                    return showMsg;
                }
            });
        });
        
        inject(function (_$rootScope_,_$controller_,_$injector_, _$httpBackend_,_$timeout_) {
            $httpBackend = _$httpBackend_;
            $scope = _$rootScope_.$new();
            $timeout = _$timeout_;
            weeklyService = _$injector_.get('weeklyService');
            controller = _$controller_('WeeklyCtrl',{
                $scope:$scope,
                $ionicLoading:ionicLoadingMock,
                weeklyService:weeklyService,
                $timeout:$timeout
            });
        });
    });
});
```
The injector unwraps the underscores (_) from around the parameter names when matching

4.编写对应的测试用例：
```js
describe('WeeklyCtrl',function () {

        it('检查初始化数据',function () {
            expect(controller.weeklyListItems).toEqual([]);
            expect(controller.showOrHide).toEqual(true);
        });
        it('刷新数据',function () {
            $httpBackend.whenGET('http://192.168.1.36:8080/aircare/user/myWeekly?max=10&offset=1').respond(
                {result: "success", records:
                {
                    weeklyList:
                        [
                            {id: 1,beginTime: "2016-07-06",endTime: "2016-07-06"},
                            {id: 2,beginTime: "2016-07-07",endTime: "2016-07-07"}
                        ],
                    weeklyCount: 10
                }
                }
            );
            controller.doRefresh();
            $httpBackend.flush();
            expect(controller.weeklyListItems.length).toEqual(2);
            expect(controller.weeklyListItems).toEqual([
                {id: 1,beginTime: "2016-07-06",endTime: "2016-07-06"}, 
                {id: 2,beginTime: "2016-07-07",endTime: "2016-07-07"}
                ]);
            expect(controller.showOrHide).toEqual(true);
        });
})
```
5.运行：
- karma start

## 补充
在项目中我们经常会用到这些事件，$ionicView.enter、$ionicView.loaded等等，那这些事件怎么测?
```js
$scope.$on('$ionicView.enter', function () {
    screen.unlockOrientation();
});
```
对应的测试用例
```js
it('当进入页面后,调用$ionicView.enter',function () {
    $scope.$emit('$ionicView.enter');
    $scope.$digest();            
    $scope.$on('$ionicView.enter');
    expect(screen.unlockOrientation).toHaveBeenCalled();
});
```
## 总结 
- 在编写用例之前需要把注入的依赖mock进来，实质就是将用的这些方法在测试中全部模拟好。


