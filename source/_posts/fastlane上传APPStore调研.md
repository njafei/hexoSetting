---

title: fastlane上传appStore功能调研
date: 2017-05-17 17:10:34
tags: [自动化,fastlane]
categories: 自动化

---


# 背景

目前jenkins服务器上面使用的版本比较老，是1.* 的版本。这个版本的deliver，自定义的功能比较少，存在以下弊端：

	如APP正在审核版本9，这时候上传一个版本,deliver会自动取消当前版本的审核状态。而这种自动的提交和取消审核状态是团队无法接受的。
	所以今天调研了最新的版本
	
	
# 文档
	
查看了deliver的[最新文档](https://github.com/fastlane/fastlane/tree/master/deliver)，最新的提供了新的参数`submit_for_review `，目前配置如下：
	
```
app_identifier "com.**.**" # The bundle identifier of your app	
username "njafei@163.com" # your Apple ID user
force true #don’t show me the preview html
submit_for_review false # 这个开关就是控制是否要展示的
```
	
# 使用流程

```
$ fastlane deliver init //初始化fastlane 会生成配置文件Deliverfile、文档数据（用于各种说明等）文件夹metadata、screenshots（展图）等文件
$ 输入itunes connenct账号等
$ 配置Deliverfile文件，配置同文档中所列
$ fastlane deliver *.ipa

```
	
# 数据
直接使用公司网络上传3次，均失败
使用lantern上传了8次，成功了一次,上传时间大约20min
	
线上无app在review中：正常上传，无影响
线上有app在review中：正常上传，无影响

	
# 其他

使用deliver的话，一定要配置一个版本更新说明，地址在`./metadata/zh-Hans/release_notes.txt`;

#总结
fastlane的上传功能现在已经可以满足使用条件，但是网络状态实在比较差，可以考虑写脚本由上传人员使用，配合jenkins的话，恐怕失败率比较高

