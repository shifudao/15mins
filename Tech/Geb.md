
***
#Geb测试分享
###Geb简介
Geb是浏览器自动化(browser automation)的解决方案, 以强大的Selenium WebDriver作为基础，直接控制浏览器进行网站操作，可以用来做Web的自动化测试 ，功能测试（Functional Testing）和验收测试(User Acceptance Testing)并生成网站截图的快照 ，它具有以下优点：
* 易学易懂易用,
* 高度整合现有的测试框架如：Junit,Spock,TestNG等等，
* 可以搭配 Grails,Gradle,Maven,Jenkins等等使用,
* Geb支持很多核心浏览器如：Firefox,PhantomJS,Chrome,Internet Explorer,等
* 更方便的获取网页组件 


###认识Geb与WebDriver
Geb是构建在WebDriver之上，Geb测试也可以参考WebDriver API。WebDriver架构及其原理如下：
* webdriver是按照server – client的经典设计模式设计的.
* server端就是remote server，可以是任意的浏览器。当我们的脚本启动浏览器后，该浏览器就是remote server，它的职责就是等待client发送请求并做出响应.
* client端简单说来就是我们的测试代码，我们测试代码中的一些行为，比如打开浏览器，转跳到特定的url等操作是以http请求的方式发送给被测试浏览器，也就是remote server；remote server接受请求，并执行相应操作，并在response中返回执行状态、返回值等信息。


###Geb测试完整案例与解析
 在这里以Grails框架和Firefox做Geb测试，以用户登录功能为例，要求：
 * JDK7以上
 * Groovy2.3以上
 * Firefox3.3以上
 
测试前的配置

* Grails中对BuildConfig.groovy的配置
* GebConfig的配置
 
#####Grails的BuildConfig.groovy配置
```
def gebVersion = "0.13.1"
def seleniumVersion = "2.52.0"
dependencies {       
        test "org.grails:grails-datastore-test-support:1.0.2-grails-2.4"
        test "org.seleniumhq.selenium:selenium-support:2.52.0"
        test("org.seleniumhq.selenium:selenium-firefox-driver:$seleniumVersion")        
        test "org.gebish:geb-spock:$gebVersion"       
        
    }
 plugins {      
        test ":geb:$gebVersion"
    }
```
#####Geb中GebConfig.groovy的配置
```
import org.openqa.selenium.firefox.FirefoxDriver
import org.openqa.selenium.firefox.FirefoxProfile
driver = {
        FirdfoxProfile profile = new FirefoxProfile()
        profile.setPreference("intl.accept_languages", "en-us")
        def driverInstance = new FirefoxDriver(profile)
        driverInstance.manage().window().maximize()
        driverInstance
}
baseUrl = 'http://127.0.0.1:8080/aircare/'
baseNavigatorWaiting = true
atCheckWaiting = true
autoClearCookies = true
```
#####测试代码
```
//page页面  
 class LoginPage extends Page {
    static url="auth/login"
    static at={
         waitFor{title=="用户登录"}
     }
     
    static content = {
        loginName(required:false){$("#username",0)}
        loginPwd(required:false){$("#password")}
        loginButton(required:false){$("#loginBtn")}
 
     }
     
    def login(String phoneNum ,String pwd){
         loginName.value(phoneNum)
         loginPwd.value(pwd)
         loginButton.click()
         Thread.sleep(3000)   
       }
     }
 
 ```
 **范例说明：**先定义一个LoginPage页面，用来在测试环境启动好后，浏览器根据测试代码打开这个LoginPage页面，而在这个页面上是用jQuery对涉及到测试的页面元素进行获取，也可以自定义一些登录方法去便于复用（以上，login(String phoneNum,String pwd)是自定义登录方法）。Geb在page页面定义了三个很重要的静态属性，如下:
   1. url: 表示页面要跳转的地址或者网址（如果配置了baseUrl则page页面要写成相对路径）
   2. at: 判断跳转的页面是否在自己所希望的这个页面上
   3. content: 表示page包含的内容（获取的页面元素）


```    
      //测试代码 
      def "用户可以正常登录"(){
        when:
        to LoginPage
        loginName.click()
        loginName.value("13366665442")
        loginPwd.value("admin1")
        loginButton.click()
        Thread.sleep(3000)

        then :
        waitFor{at LandingPage}
     }
```
**范例说明：**when then 是geb测试的基本格式要配对使用，when里面可以写一些操作比如点击按钮，填写表单，获取元素的值等，then 里面可以写一些条件判断，对于when里一些操作之后期望达到的结果作出断言，也有一些很重要的语法要了解如下：

1. to: 表示让浏览器去到哪个page页面
2. go: 和to表达的意思相同，只是go 要接收一个字符行式的url 比如：go 'www.baidu.com'
3. value: 将数据填入到表单
4. text: 获取页面元素包含的文字
5. waitFor: 等待一个条件被满足
6. click: 模拟点击页面元素事件

#####运行测试
```
grails test-app functional:
如果Geb测试有单独的运行环境要进行切换:
grails -Dgrails.env=geb test-app functional:
```

###Geb测试异常及解决思路
以下是自己在开发过程中出现的部分异常，并进行了解决与总结

 ``` 
 geb.error.SingleElementNavigatorOnlyMethodException: Method click()
 can only be called on single element navigators but it was called on a
 navigator with size 2. Please use the spread operator to call this
 method on all elements of this navigator or change the selector used
 to create this navigator to only match a single element. 
 ```

 
 **The solution**

 1. page页面获取的元素不唯一，geb获取到多个元素会报此错，重新获取元素，保证元素唯一
 2. 具体的指明要获取多个元素中的第几个元素如： $("div", 0)或者$("div", 0, title: "section")

----------


```
 org.openqa.selenium.InvalidElementStateException:   
 {"errorMessage":"Element is not currently interactable and may not be 
 manipulated","request":{"headers":{"Accept-Encoding":"gzip,deflate","Connection":"Keep-Alive","Content-Length":"27","Content-Type":"application/json;
 charset=utf-8","Host":"localhost:20656","User-Agent":"Apache-HttpClient/4.4.1    (Java/1.8.0_60)"},"httpVersion":"1.1","method":"POST","post":"{\"id\":\":wdc:1464585689847\"}","url":"/clear","urlParsed":{"anchor":"","query":"","file":"clear","directory":"/","path":"/clear","relative":"/clear","port":"","host":"","password":"","user":"","userInfo":"","authority":"","protocol":"","source":"/clear","queryKey":{},"chunks":["clear"]},"urlOriginal":"/session/f3c3f510-2625-11e6-bd5a-cf69d84f8a16/element/:wdc:1464585689847/clear"}} 
``` 


 **The solution**

 1.跟浏览器窗口大小有很大的关系，一般要在配置文件中设置浏览器大小，防止页面css样式变化而导致页面不可操作,可这样配置一下             phantomJsDriver.manage().window().maximize()

----------

``` 
Element is not currently visible and may not be manipulated exception  #11637
```

  **The solution**

 1. 元素不可见时不可以操作，可能的原因是元素本身是不可见的，还有就是页面js或者ajax延迟加载隐藏元素还没有加载出来，要在触发js或者ajax后让浏览器等待服务器响应一定的时间后再进行操作，可以这样：Thread.sleep(3000)
 2. 如果截图上已经可见隐藏元素显示出来，还报出些错，这个跟浏览器窗口大小有关系，一般配置好了，这个问题就可能会避免
 3. 页面的js或者ajax没有被触发，原因也很多比如浏览器的js不兼容，触发不了，这时要修改js,此外定位的元素不够精准，范围太大也会触发不了js

    

----------


 

``` 
java.lang.AssertionError: no browser confirm() was raised  
```

**The solution**

 1. 这个是在弹出confrim 或者alert时容易报的错，主要原因还是元素定位不准确，比如现在用的phantomJS对元素的定位就要精准，尽量定位到时a标签或者button上click()，定位元素时最好要定到最内层标签

----------


  

```  
geb.waiting.WaitTimeoutException: condition did not pass in 60.0   
 seconds. Failed with exception: - See more at: 
 ```

**The solution**

1. 等待超时，可以延长一下等待时间，同时当前的运行环境，服务器的响应速度对Geb测试也有很大的影响，程序执行的快，而响应的时间长就会超时，所有只能估计一个大概等待时间
2. 自己本身的错误，如错误的页面跳转

----------


 

```  
[INFO  - 2016-05-26T09:46:48.760Z] SessionManagerReqHand -   
 _cleanupWindowlessSessions - Asynchronous Sessions clean-up phase starting NOW 
 ```

**The solution**

1. 在不断开romteServer的情况下再打开一个新的driver创建新的session继续执行测试   
```  
def setupSpec(){  
          def phantomJsDriver= new PhantomJSDriver()   
          phantomJsDriver.manage().window().maximize()       
          this.driver=phantomJsDriver      
          new WebDriverWait(driver,30000)  
    }  
```
###总结
用Geb做测试还是比较快捷的，也是目前最流行的自动化测试工具，不用考虑程序本身的逻辑问题，就可以轻松测试并生成网站截图，相对于项目中一些复杂的单元测试和集成测试还是要容易很多,同时对程序运行环境的要求较高，维护成本也较高，对于页面Ui的测试和复杂的功能可以使用Geb来做测试。
###参考资料
* [http://www.gebish.org/manual/current/] ( http://www.gebish.org/manual/current/ )
