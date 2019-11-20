# CHANGE LOG
---
## 20191024
* build_publish_lib_basic.gradle添加 Task: publishSupportLibSnapshotNoMail.在上传snapshot版本时，不发送邮件通知。

---
* 添加build_tvkit_dependencies.gradle脚本。用来检测，依赖tvkit依赖库版本。
* build_publish_app.gradle默认加载build_tvkit_dependencies.gradle脚本。