# Gradle基础

## Gradle简介
Gradle是一个基于`Apache Ant`和`Apache Maven`概念的项目自动化建构工具。它使用一种基`于Groovy DSL`声明项目设置，而不是传统的XML。

Gradle的配置文件相比Ant与Maven来说，更加灵活(基于Groovy DSL而非XML)，也更易懂。并且Gradle兼容Ant与Maven(有对应的插件直接支持Ant或Maven的配置)，可以无缝从Ant或Maven迁移过来。

Gradle支持多种语言的构建，并不仅限与JVM相关的语言。并且支持多项目构建。

Gradle目前社区十分活跃，官方插件和社区插件不断完善，大大简化了构建脚本的编写。

## 安装与使用
Gradle依赖于`JDK`，使用前务必确保JDK已经安装，并且`JAVA_HOME`环境变量指向了你需要的JDK的路径(多个JDK版本切换可以通过切换JAVA_HOME环境变量实现，让Gradle使用不同的JDK版本编译项目)。

然后直接从[官方下载](https://gradle.org/gradle-download/)页下载zip包解压即可。Gradle已经内置了Groovy运行环境，因此无需单独安装Groovy(实际即使安装了也没用，Gradle只会调用内置的Groovy)。

也可以从[sdkman](http://sdkman.io/)安装:

    sdk install gradle

安装过后应该能看到这样的输出:

```bash
$ /path/to/${GRADLE_HOME}/bin/gradle -v

------------------------------------------------------------
Gradle 2.14.1
------------------------------------------------------------

Build time:   2016-07-18 06:38:37 UTC
Revision:     d9e2113d9fb05a5caabba61798bdb8dfdca83719

Groovy:       2.4.4
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_101 (Oracle Corporation 25.101-b13)
OS:           Linux 4.4.0-31-generic amd64
```

此时我们就可以开始使用Gradle构建我们的项目了。

## Hello World
让我们先运行第一个极简的范例`Hello World`，在终端上输出一行`Hello World`字符串。

```bash
$ mkdir helloWorld
$ cd helloWorld
$ vim build.gradle
```

我们创建了一个目录`helloWorld`，在这个目录下创建一个名为`build.gradle`的文件，写入以下内容:

```groovy
task helloWorld {
    println "Hello World"
}
```

然后我们在这个目录下执行`gradle -q helloworld`，即可看到如下输出:

```bash
$ gradle -q helloworld  # 注意gradle的task在命令行上不区分大小写
Hello World
```

- gradle的构建步骤是以`task`方式运行，在gradle的命令行上指明要执行的`task`即可，可以写多个`task`。如: `gradle task1 task2 ... taskn`
- gradle默认的`task`配置文件为`build.gradle`
- gradle默认的`properties`配置文件为`gradle.properties`
- gradle默认的多项目构建配置文件为`settings.gradle`。单项目构建无用，多项目构建必备。

创建一个gradle工程项目，可以在项目目录下使用`gradle init`命令初始化，帮你生成参考配置文件。也可以用[lazybones](https://github.com/pledbrook/lazybones)工具快速创建一个模板。

## Task的依赖与扩展
gradle列出当前项目中可用的task使用命令`gradle tasks`即可，在一个空目录下看到的命令输出像是这样:
```bash
$ gradle tasks
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'empty'.
components - Displays the components produced by root project 'empty'. [incubating]
dependencies - Displays all dependencies declared in root project 'empty'.
dependencyInsight - Displays the insight into a specific dependency in root project 'empty'.
help - Displays a help message.
model - Displays the configuration model of root project 'empty'. [incubating]
projects - Displays the sub-projects of root project 'empty'.
properties - Displays the properties of root project 'empty'.
tasks - Displays the tasks runnable from root project 'empty'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL

Total time: 1.027 secs
```

`Task`之间是可以定义依赖关系的。比如我们要实现这样的依赖关系:

![](https://docs.gradle.org/current/userguide/img/commandLineTutorialTasks.png)

可以在`build.gradle`文件定义以下内容:

```groovy
task compile {
    doLast {
        println 'compiling source'
    }
}

task compileTest(dependsOn: 'compile') {
    doLast {
        println 'compiling unit tests'
    }
}

task test(dependsOn: ['compile', 'compileTest']) {
    doLast {
        println 'running unit tests'
    }
}

task dist(dependsOn: ['compile', 'test']) {
    doLast {
        println 'building the distribution'
    }
}
```

此时我们用`gradle tasks --all`看看当前可用的task(处于依赖底层的task需要加上`--all`才能显示出来):

```bash
$ gradle tasks --all
compiling source
compiling unit tests
running unit tests
building the distribution
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'helloWorld'.
components - Displays the components produced by root project 'helloWorld'. [incubating]
dependencies - Displays all dependencies declared in root project 'helloWorld'.
dependencyInsight - Displays the insight into a specific dependency in root project 'helloWorld'.
help - Displays a help message.
model - Displays the configuration model of root project 'helloWorld'. [incubating]
projects - Displays the sub-projects of root project 'helloWorld'.
properties - Displays the properties of root project 'helloWorld'.
tasks - Displays the tasks runnable from root project 'helloWorld'.

Other tasks
-----------
dist
    compile
    compileTest
    test

BUILD SUCCESSFUL

Total time: 1.223 secs
```

现在我们运行`gradle dist`看看输出:
```bash
$ gradle dist
compiling source
compiling unit tests
running unit tests
building the distribution
:compile UP-TO-DATE
:compileTest UP-TO-DATE
:test UP-TO-DATE
:dist UP-TO-DATE

BUILD SUCCESSFUL

Total time: 1.042 secs
```

完全符合我们的预期。

有时候我们可能会希望不改动原有task代码的基础上，做一些扩展(比如原本的task不是你自己写的，而是由插件引入的)。

```groovy
task mytask << {
    println 'do something.'
}

mytask.doLast {
    println 'do last.'
}

mytask {
    doFirst {
        println 'do first.'
    }
}

mytask.dependsOn 'another'

task another {
    println "executed: $another.name task"
}
```

> `<<`只是`doLast`的别名。`doFirst`和`doLast`都可以多次调用。

```bash
$ gradle -q mytask
executed: another task
do first.
do something.
do last.
```

我们直接定义`mytask`这个task的时候，使用了`doLast`形式定义，这也是社区通用的做法。

> 试试看，如果不用`doLast`形式定义task，会出现什么现象。

### 动态task
这个是Gradle一项非常强的功能，不需要每个task都单独定义。可以定义task模板，通过Groovy DSL批量生成对应的`task`。也可以指定task Rule。看看下面这样的范例:

build.gradle
```groovy
4.times { counter ->
    task "task$counter" << {
        println "I am task number $counter"
    }
}
```

```bash
> gradle -q task1
I am task number 1
```

这个范例来自于`barn`项目的`build.gradle`配置:
```groovy
tasks.addRule("Patten: db_<suffix>") {taskName ->
    if(taskName.startsWith("db_")){
        def names = taskName.split('_')
        task(taskName, type: JavaExec) {
            main = mainClass
            args = names.toList() << "${buildDir}/resources/main/testconfig.yml"
            classpath = sourceSets.main.runtimeClasspath
        }
    }
}
```

> 更多有关task的高级玩法，参考官方文档: https://docs.gradle.org/current/userguide/more_about_tasks.html

## 插件
Gradle的设计理念是，所有有用的特性都由[Gradle插件](https://docs.gradle.org/current/userguide/plugins.html)提供。由于社区的活跃，今天很多复杂的构建步骤都不用手写task了。只需要引入相应的插件，然后改下插件配置即可运行起来。

以下我们用最简单的编译一个Java源码的范例做演示。

此时`build.gradle`只有一行:
```groovy
apply plugin: 'java'
```

此时我们再列一下当前项目可用的task:
```bash
$ gradle tasks
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles main classes.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles test classes.

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Documentation tasks
-------------------
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'helloWorld'.
components - Displays the components produced by root project 'helloWorld'. [incubating]
dependencies - Displays all dependencies declared in root project 'helloWorld'.
dependencyInsight - Displays the insight into a specific dependency in root project 'helloWorld'.
help - Displays a help message.
model - Displays the configuration model of root project 'helloWorld'. [incubating]
projects - Displays the sub-projects of root project 'helloWorld'.
properties - Displays the properties of root project 'helloWorld'.
tasks - Displays the tasks runnable from root project 'helloWorld'.

Verification tasks
------------------
check - Runs all checks.
test - Runs the unit tests.

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL

Total time: 2.267 secs
```

很明显看到这个插件已经帮我们加入了很多java构建的时候需要的步骤。

然后我们建立`src/main/java/Hello.java`文件，写一个Hello World程序:
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

执行`gradle build`构建项目看看:
```bash
$ gradle build
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL

Total time: 2.566 secs
```

构建好的程序默认存放在`build/`目录下，直接运行编译好的`Hello.class`文件:

```bash
$ java -cp build/classes/main Hello
Hello World!
```

> 这种目录结构是`java`这个plugin定义好的: https://docs.gradle.org/current/userguide/java_plugin.html#N152C8

总结一下`build.gradle`文件中引入插件的方法:
```groovy
apply from: 'other.gradle'  // Script plugins
apply plugin: 'java'        // Binary plugins

// Applying plugins with the buildscript block
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:0.4.1"
    }
}

apply plugin: "com.jfrog.bintray"

// Applying plugins with the plugins DSL
plugins {
    id 'java'
    id "com.jfrog.bintray" version "0.4.1"
}
```

> 插件使用详情请参考官方文档: https://docs.gradle.org/current/userguide/plugins.html

## 完整示例
这个示例来自于我们自己的how-to项目中的[kafkalog4jappender](https://github.com/shifudao/how-to/blob/master/mongodb2hbase/build.gradle)的`build.gradle`文件:

```groovy
buildscript {
    repositories {
        mavenLocal()
        jcenter()
        mavenCentral()
        maven {
            url = 'http://oss.sonatype.org/content/repositories/snapshots/'
        }
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:1.2.2'
    }
}

apply plugin: 'groovy'
apply plugin: 'idea'
apply plugin: 'com.github.johnrengelman.shadow'

version = '0.1'

if (!JavaVersion.current().java8Compatible) {
    throw new IllegalStateException('''Please install Java 8!'''.stripMargin())
}

repositories {
    jcenter()
    mavenLocal()
    maven {
        url = 'http://oss.sonatype.org/content/repositories/snapshots/'
    }
    mavenCentral()
}

dependencies {
    compile "io.vertx:vertx-core:3.2.1"
    compile "org.codehaus.groovy:groovy-all:2.4.5"
    compile "io.vertx:vertx-lang-groovy:3.2.1"
    compile('org.apache.kafka:kafka-clients:0.9.0.0') {
        exclude group: 'com.sun.jdmk', module: 'jmxtools'
        exclude group: 'com.sun.jmx', module: 'jmxri'
        exclude group: 'org.slf4j', module: 'slf4j-log4j12'
        exclude group: 'log4j', module: 'log4j'
    }

    compile fileTree(dir: "lib", includes: ['*.jar'])

    runtime 'org.slf4j:slf4j-log4j12:1.6.2'
    runtime('org.apache.kafka:kafka-log4j-appender:0.9.0.0') {
        exclude group: 'org.slf4j', module: 'slf4j-log4j12'
        exclude group: 'log4j', module: 'log4j'
    }

    testCompile 'org.spockframework:spock-core:1.0-groovy-2.4'
    testRuntime 'cglib:cglib-nodep:3.1'
    testRuntime 'org.objenesis:objenesis:2.1'
}

shadowJar {
    classifier = 'fat'
    manifest {
        attributes 'Main-Class': 'io.vertx.core.Launcher'
        attributes 'Main-Verticle': 'groovy:foxgem.MainVerticle'
    }
    mergeServiceFiles {
        include 'META-INF/services/io.vertx.core.spi.VerticleFactory'
    }
}
```
