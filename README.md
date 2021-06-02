我们根据gradle 的task任务，可以做一些在项目编译期间，运行插件来帮助我们完成一些额外的工作。在Android中，我们只要依赖了
```
dependencies {
    implementation gradleApi()
    implementation localGroovy()
    api "com.android.tools.build:gradle:$apgVersion"
    implementation 'commons-io:commons-io:2.6'
    implementation 'org.javassist:javassist:3.27.0-GA'
}
```
我们就可以使用java代码的方式，如 `implements Plugin<Project> `和`extends Transform` 这样的方式定义自己的插件。拿代码插桩来说，代码插桩的任务，就是在编译时在相应的位置插件自己的一段代码逻辑。这种aop式的技术在很多技术框架中都有很多应用；例如：dagger2，butterknife，arouter等等。
（**注**：本文在android studio  `gradle-6.5-all.zip`和`gradle:4.1.0` 下实现，可以在项目中查看**Appplugin **项目文件 ）。
一、创建插件项目
如：一个 **Appplugin **项目。里面创建baseplugin和其它xxplugin。
xxplugin可以看作是一个module，我们在他的resources文件下面定义一个插件名称，例如：
implementation-class=com.Appwill.plugin.AppRouterPlugin
同时，对他的build.gradle文件。这样处理
```
plugins {
    id 'java-library'
    id 'kotlin'
    id 'groovy'
    id 'java-gradle-plugin'
}

dependencies {
    implementation gradleApi()
    implementation localGroovy()
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implementation project(":BasePlugin")
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc6'
}

gradlePlugin {
    plugins {
        version {
            // 在 app 模块需要通过 id 引用这个插件
            id = 'Approuter-plugin'
            // 实现这个插件的类的路径
            implementationClass = 'com.Appwill.plugin.AppRouterPlugin'
        }
    }
	
}
```
我们定义了 plugins的id ,这个ID就是插件名称。在后面会根据这个名字来使用当前插件
二、代码方式编写插件
具体可查看项目`ApprouterPlugin`项目
三、应用
我们这里不需要将插件上传到maven 等仓库，而是可以直接本地依赖，
在项目最外层的build.gradle 中添加
```
 plugins {
     id "Approuter-plugin" apply false
}
```
保证运行时能够走插件任务
之后，在app项目里，在添加使用`apply plugin: 'Approuter-plugin'`
这样，一个本地插件开发配置就走完了，当然细节还是要自己去练一练。gradle 的task 任务是一个有向无环结构，在使用时要注意task 的先后以及依赖关系。

	