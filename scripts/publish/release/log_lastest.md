--- 

#### 版本: 1.0.2
* 时间: 2019-03-08 17:13:21
* [参考文档](https://gitlab.fengmi.tv/tv-public/gradle_project/blob/master/README.md)
#### 更新点
1. 去除无用配置文件gradle.properties.upload_config中去掉无用配置。添加.cache.properties内部使用。
2. 邮件添加默认的css.支持表格等功能。
3. 添加build_publish_lib_basic.gradle脚本，在发布时，如果不希望上传源码，可以使用此脚本上传。

**注意，请将插件库更新至最新版本：**
```java
    com.bftv.tools.build:gradle:1.0.3
```
在使用release-1.0.2的脚本时使用如下方法使用：
```java
apply from:'https://gitlab.fengmi.tv/tv-public/gradle_project/raw/release-1.0.2/scripts/build_publish_lib.gradle'
```
#### Maven:
``` 
com.bftv.fui:gradle-scripts-test:1.0.2
``` 

#### 最后提交:
``` 
commit 24c8fc66c9b08b0eb748205458f2183781527942
Author: zhaopeng <zhaopeng@bftv.com>
Date:   2019-03-08 16:56:08 +0800

    打包发送邮件时添加默认的css.
``` 
