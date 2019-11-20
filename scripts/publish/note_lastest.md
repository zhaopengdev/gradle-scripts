#### 更新点
* 添加build_tvkit_dependencies.gradle脚本。用来检测，依赖tvkit依赖库版本。
* build_publish_app.gradle默认加载build_tvkit_dependencies.gradle脚本。
使用方法：
```java
一、在Android 应用的build.gradle中加载脚本：

apply from: tvkit_gradle_scripts_path+'build_tvkit_dependencies.gradle'
或者
apply from: tvkit_gradle_scripts_path+'build_publish_app.gradle'

二、依赖需要的模块：
//例如添加瀑布流模块依赖
api tvkit_latest['waterfall'] //tvkit_latest代表最新版本
或者
api tvkit_stable['waterfall'] //tvkit_stable代表稳定版本
```
另外可以通过task:tvkit app:checTvkitLatest、task:tvkit app:checTvkitStable任务来检测tvkit的版本
