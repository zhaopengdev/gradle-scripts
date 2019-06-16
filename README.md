#### 简介
通过依赖scripts目录下的gradle脚本来提供一些公共的功能、工具类等

#### 脚本介绍
* build_utils.gradle 编译工具类，提供一些方便的任务
* build_support-lib.gradle 主要用来发布SupportLib。功能包括自动打包上传maven、发布邮件通知、记录日志等、打包时自动上传源码。
* build_publish_lib_basic.gradle 与build_support-lib.gradle相同，但在上传包时，不上传源码。
* maven_upload.gradle 管理上传lib到maven仓库的脚本。有些任务会修改upload_gradle.properties和gradle.properties
* publish_log.gradle 在版本发布任务执行中，负责记录一些发布信息。

#### 接入
此工程目的是可以使用一些自定义task来加速开发工作。开发者需要根据不同需求依赖scripts下的脚本，为自己项目添加所需的task.   
**以下使用发布一个Support Lib为例来说明如何使用。**
#### 发布SupportLib
###### 第一步：rootProject/build.gradle添加插件依赖
```java
buildscript {
    repositories {
        mavenLocal()
        maven {
            url "http://192.168.10.9:8081/repository/maven-public/"
            credentials {
                username 'xxx'
                password 'xxx'
            }
        }
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
        //必要的插件
        classpath 'com.bftv.tools.build:gradle:1.0.3'
    }
}

...
```
###### 第二步：projectDir/build.gradle中的配置
```java
//发布版本所用的脚本
apply from:'https://gitlab.fengmi.tv/tv-public/gradle_project/raw/master/scripts/build_publish_lib.gradle'
//注意，这里master分支会随着代码的提交，保持脚本是最新的。如果希望使用特定版本的脚本可以单独指定：
apply from:'https://gitlab.fengmi.tv/tv-public/gradle_project/raw/release-1.0.2/scripts/build_publish_lib.gradle'
//根据需求，配置一些一些task中的参数。
//配置生成发布日志时的参数
generatePublishLogMarkdownRelease{
    //项目参考文档地址
    reference 'http://xxx/xx/javadoc/'
    //本次发布更新点,通过更新project/publish/note_lastest.md来修改更新点

}

```
###### 第三步：根据需求配置一些Properties文件
* projectDir/upload_config.properties
```java
snapshotUrl=http\://192.168.10.9\:8081/repository/maven-snapshots/
mavenLocal=true
snapshot=true
#配置上传的版本号
mavenVersion=1.0.1
sourceJarDir=../sourcesJar/
#release版本的URL
releaseUrl=http\://192.168.10.9\:8081/repository/maven-releases/
mavenArtifactId=script
mavenGroupId:=com.bftv.fui
```
注意，此脚本不存在时，执行task:libProjectInit可自动生成此文件,并且做一些任务发布工作的准备。
最终上传到maven后引入方式为:
```java
  mavenGroupId:mavenArtifactId:mavenVersion
```
* rootDir/local.properties 配置maven上传用户、密码
```java
MAVEN_NAME=XXXX
MAVEN_PASSWORD=XXXX
```

###### 第四步：执行task
这里需要发布release版本的lib库。所以执行task:publishSupportLibRelease后便可将此次lib发布后并且使用邮件通知开发者。



###### 配置文件说明
* upload_config.properties 在上传maven档案之前，会读取此配置文件配置档案信息
```java
mavenVersion=1.0.1
snapshotUrl=http\://192.168.10.9\:8081/repository/maven-snapshots/
mavenLocal=true
snapshot=true
#配置上传的版本号
sourceJarDir=../sourcesJar/ #
#release版本的URL
releaseUrl=http\://192.168.10.9\:8081/repository/maven-releases/
mavenArtifactId=script
mavenGroupId=com.bftv.fui
```
  * mavenVersion 版本发布的基础版本号。
  * snapshotUrl,releaseUrl 上传maven的服务器地址
  * mavenLocal 是否上传档案到本机上  //不要自己指定
  * snapshot 是否是snapshot版本，为真时会自动在mavenVersion号后添加 'gitCommitCount'+"-SNAPSHOT" //不要自己指定
  * sourceJarDir 源码jar包存放位置
  * mavenArtifactId 名称
  * mavenGroupId 组名称  
  最终生成的maven依赖地址由mavenGroupid
###### 关于publish目录
publish目录是发布时的工作目录。在每次成功发布lib后，都会在publish下建立相关发布日志。分别位置snapshot/release俩个目录下。并且生成log_lastest文件来记录最后一次发版的日志。
publish目录下的note_lastest.md用来修改本次版本发布的更新点等信息。开发者可以根据需求，随意填写。最后的发版的日志以及通知邮件中，都会将此更新点以markdown的形式展示。

###### 发布相关Task说明
  * generatePublishRootDir
  此任务将在project下创建publish目录，并在其下创建以下文件:
    * release及snapshot俩个目录，用来记录发布日志。
    * .gitignore
    * note_lastest.md 用来编写更新点

  * libProjectInit
  此任务基于generatePublishRootDir及generateMavenProperties.当一个新创建一个lib模块时，可以执行一下此任务进行一些发布前的初始化工作。**注意如果本地存在upload_config文件，可以直接执行任务generatePublishRootDir**

  * markdownMail 使用markdown来发送邮件。
  ```java
  markdownMail{
      to 'user@bftv.com' //接收人
      cc 'others1@xx.com','other2@xx.com' //抄送
      subject 'Test' //邮件标题
      contentType 'text/html;charset=utf-8' //邮件编码
      markdownFile file('/Test.MD') //编辑此源文件用来发送邮件
      from "xx@qq.com" //邮件发送地址
      userName "xx@qq.com" //邮件用户名
      password "xxxxxxxx" //邮件密码或者受权码
      serverHost "smtp.exmail.qq.com" //serverHost
      serverPort "465" //serverPort
      time new Date().toString() //时间戳
  }
  ```
  * markdownToHtmlUtil markdown文档转成html
  ```java
    markdownToHtmlUtil{
        sourceFile file('/Test.MD')  //源文件
        outputFile file('/test_output.html') //输出文件
    }

  ```
  * updateUploadConfigXX(XX为Release/Snapshot/SnapshotMavenLocal)
    根据不同发布需求，改变upload_confing.properties文件.以下上传本地maven.并且为snapshot版本的实现。
    ```java
    task updateUploadConfigSnapshotMavenLocal(type : updateUploadConfig){
      group getProperty("taskGroup")
      release false //是否是Release版本
      mavenLocal true //是否上传到本地
    }
    ```
  * generatePublishLogMarkdownRelease/Snapshot 发布版本时，将自动在本地生成记录。发送邮件时将使用此task生成的内容。
  ```java
  generatePublishLogMarkdownRelease{
      time new SimpleDateFormat("yyyy-MM-dd HH:MM:ss").format(new Date())
      enableDoc true //是否显示reference字段
      enableGitLog false //是否显示提交记录
      changes '' //'更新点：1. XXX 2 XXXX' //这里已经弃用、通过更新project/publish/note_lastest.md来修改更新点
      gitLog '' //提交日志
      reference '' //'文档/帮助地址'
      mavenPath ''//'最终生成的maven地址'
      version ''//'版本'
      extraItemMap.put("Title:",'XXX') //添加以####(h4)展示项
      codeItemMap.put("Last commit:",commit) //添加以<code>形式展示项
      linkItemMap.put("百度:","http://baidu.com") //添加额外链接项使用此map
}
  ```
  * mailPreviewXX 预览邮件
mailPreview任务会自动执行generatePublishLogMarkdownXX任务后，将其转换成html存放于project/publish/.mail_content.html。直接打开此文件就可以进行邮件内容预览。


  * mailTask 发布版本时的最后一步，邮件通知更新点。
  ```java
  //mailTask内部字段与task:markdownMail一致。
  mailTask{
      to 'user@bftv.com' //此task默认收件人是所有TV组开发人员。如果只想发给对应的人，覆写此字段。实际开发中，可以修改此地址测试邮件发送内容。
  }

  ```
  * publishSupportLibXX(XX:Release/Snapshot等) 其中publishSupportLibRelease实现如下：
  ```java
  task publishSupportLibRelease{
      uploadArchives.mustRunAfter updateUploadConfigRelease
      dependsOn generatePublishLogMarkdownRelease,uploadArchives ,markdownToHtml,mailTask
  }
  ```
  ###### supportLib开发及发布一般流程图

```mermaid
 graph TD

  pStart((开始))
  pEnd((结束))
  cNew{首次使用}
  cRemote{发布到线上}
  pTaskInit[Task:libProjectInit]
  pConfig[根据需求修改upload_config.properties]
  pMailPreview[Task:mailPreview]
  pNoteLog[修改发布日志note_lastest.md]
  cRelease{是否Release版本}
  pCoding[Coding]
  cCheck{检查发布内容}
  pPublishSnapshot[Task:publishSupportLibSnapshot]
  pPublishRelease[Task:publishSupportLibRelease]
  pPublishMavenLocal[Task:publishSupportLibMavenLocal]
  pCommitLog[提交publish目录]

  pStart --> cNew
  cNew --是--> pTaskInit

  pTaskInit --> pCoding
  pCoding --可选--> pConfig
  pConfig --> cRemote
  pCoding --> cRemote
  cRemote --否--> pPublishMavenLocal
  pPublishMavenLocal --> pEnd
  cNew --否--> pCoding

  cRemote --是--> pNoteLog

  pNoteLog --> cCheck

  cCheck --是--> pMailPreview


  cCheck --否--> cRelease

  cRelease --是--> pPublishRelease
  cRelease --否--> pPublishSnapshot

  pPublishRelease --> pCommitLog
  pPublishSnapshot --> pCommitLog
  pCommitLog --> pEnd

```

