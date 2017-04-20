# DEPRECATED

中文站（更新到 1.1）：https://www.kotlincn.net/ 

Kotlin 中文论坛：https://www.kotliner.cn/


我们从 kotlin 0.4 版本翻译这个项目，非常感谢所有小伙伴做出的努力。kotlin 官网从 jekyll 改为 docker 由于工作原因一直没有时间维护，在此期间 灰蓝天际同学对这个项目做出了巨大贡献。

kotlin 中文同步翻译跟进请移步这个库进行协作翻译：https://github.com/hltj/kotlin-web-site-cn

欢迎加入Kotlin翻译小组(DEV kotlin translator group)   QQ群：419484222  

欢迎关注我的微信公众号：晓晨Android组 微信号：xiaochenAndroid

======

# Kotlin中文站

网址（更新到 1.0.3）：[http://tanfujun.com/kotlin-web-site-cn/](http://tanfujun.com/kotlin-web-site-cn/)  

中文站（更新到 1.1）：https://www.kotlincn.net/

> ~~说明：中文站是由[Jween](https://github.com/Jween)搭建在日本服务器上，感谢[Jween](https://github.com/Jween) 的服务器和域名支持~~
> ~~他会定期将本项目翻译合入，所以有时会出现两个站点显示不一致的情况~~

## 翻译流程

之前是在QQ群中认领，现在全部改为github流程。  
请大家先到issues中提名认领未翻译文章（issue标题请写“认领翻译XXX.md”），我会将文章更新为**已认领**，进行翻译后提出pull request（翻译请提交至Translation分支）。

* issues认领请写“认领翻译XXX.md”
* 为了避免重复认领，请你先到issues查找文件是否被认领，（主页的Readme可能不及时）
* 你可以先fork项目 然后切换到**translation**分支，在docs/reference 里可以找到需要翻译的md文件
* 翻译好后提交Pull Request我会合并进来
* 我合并了你的提交后你可以提PR更新主页的文件状态（给自己翻译的署名和链接，有时候我会帮你署名，在参与者一栏加上自己的名字）
* 如果你有好的意见或建议欢迎提出
* 从前在qq群参与翻译的小朋友，如果主页有遗漏你的话，别忘了来补上你的名字和链接哦

## 翻译进度

感谢所有参与这个项目的人！^^

| 文件名        | 文件状态           | 参与者  |  
| ------------- |:-------------:| -----:|  
|basic-syntax.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|idioms.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|coding-conventions.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|basic-types.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|packages.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|control-flow.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|Returns.md|已翻译|S_arige|  
|Classes.md|已翻译|dingsx|  
|Properties.md|已翻译|空白|  
|Interfaces.md |已翻译|[LyndonChin](https://github.com/LyndonChin)|  
|visibility-modifiers.md|已翻译|Jacky Xu|  
|extensions.md|已翻译|S_arige|  
|data-classes.md|已翻译|[Wahchi](https://github.com/wahchi)|  
|generics.md|已翻译|_Y|  
|nested-classes.md|已翻译|EasonZhou|  
|enum-classes.md|已翻译|EasonZhou|  
|object-declarations.md|已翻译|[Wahchi](https://github.com/wahchi)|  
|delegation.md|已翻译|EasonZhou|  
|delegated-properties.md|已翻译|EasonZhou,[pecpwee](https://github.com/pecpwee)|  
|functions.md|已翻译|Jacky Xu|  
|lambdas.md|已翻译|Airoyee,[pecpwee](https://github.com/pecpwee)|  
|inline-functions.md|已翻译|EasonZhou|  
|multi-declarations.md|已翻译|EasonZhou|  
|collections.md|已翻译|[灰蓝天际](https://github.com/hltj)|  
|ranges.md|已翻译|空白|  
|typecasts.md|已翻译|[Wahchi](https://github.com/wahchi)|  
|this-expressions.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|equality.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|operator-overloading.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|null-safety.md|已翻译|[drakeet](https://github.com/drakeet)|  
|exceptions.md|已翻译|[cx9527](https://github.com/cx9527)|  
|annotations.md|已翻译|[Wahchi](https://github.com/wahchi)|  
|reflection.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|type-safe-builders.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|dynamic-type.md|已翻译|[晓_晨Dev](http://tanfujun.cc)|  
|java-interop.md|已翻译|[zsn012](http://github.com/zsn012)|  
|java-to-kotlin-interop.md|已翻译|[xwSurfer](https://github.com/xwSurfer)|  
|kotlin-doc.md|已翻译|[化缘](http://frblog.sinaapp.com)|  
|using-maven.md|已翻译|[DemoJameson](http://www.demojameson.com)|  
|using-ant.md|**未翻译**||  
|using-gradle.md|已翻译|[chiahaolu](https://github.com/chiahaolu)|  
|kotlin-osgi.md|**未翻译**||  
|faq.md|已翻译|[灰蓝天际](https://github.com/hltj)|  
|comparison-to-java.md|已翻译|[chiahaolu](https://github.com/chiahaolu)|  
|comparison-to-scala.md|已翻译||  

## 生成网站

安装好Jekylly后切换到master分支 执行

```
rake build 
```
在_site文件夹下内容是生成的网站

## 生成pdf

安装好 wkhtmltopdf 在master分支执行
```
rake build_pdf 
```
会在根目录生成pdf 
在网站上你也可以找到pdf的下载链接！



## 说明

如果你发现了翻译错误，或者Kotlin文档有更新，请提PR到**Translation**分支，作者时不时会看看Kotlin网站文档有没有更新，但是毕竟精力有限，希望可以同大家共同维护。

本人能力有限，很多东西刚刚学习，希望大家不吝赐教。
