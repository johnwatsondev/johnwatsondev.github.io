---
title: 赵峰 / 高级安卓开发工程师 / 5年经验
excerpt: 赵峰个人简历
layout: cv
permalink: /mycv/
---

## 个人信息

 - 男/1991
 - 教育经历：本科 山东工商学院 电子信息科学与技术 2009-2013
 - 期望薪资：面议
 - 手机：15513898286
 - 技术博客：[johnwatsondev.com](http://johnwatsondev.com)
 - Email：johnwatsondev.com@gmail.com

## 职业技能

- 比较理解 Java 面向对象设计五大原则
- 阅读过 Framework 层 View、AMS、WMS 等模块源码
- 阅读过 Retrofit、OkHttp、Glide 以及 Otto 等优秀开源项目源码
- 熟练使用 React Native 编写应用，改造官方提供的原生控件，阅读过部分源码以及 Flex 布局引擎 Yoga 源码
- 熟练封装非业务独立能力的组件以及业务相关组件，供给其他工程师使用
- 熟练改造开源轮子达到业务要求
- 能够使用 kotlin 编写业务代码

## 工作经历
### 安卓高级开发工程师 [浙江钱粒科技有限公司](https://www.fqgj.net/) *（ 2017年2月 ～ 2018年2月 ）*

#### [钱粒账单](http://sj.qq.com/myapp/detail.htm?apkName=com.qianlizhangdan.app)
使用 React Native 以及原生代码把其他业务中的核心模块封装为 npm 私有依赖，直接让宿主应用一键调用。  
通过编译自己的 android gradle plugin 解决打包过程中遇到的问题。  
搭建线上 Jenkins 马甲打包环境，支持自定义参数构建，并且把产出物上传到 OSS 服务器。

#### [金壹贷](http://sj.qq.com/myapp/detail.htm?apkName=com.qiantu.youqian)
简化原先的 Clean Architecture 架构，去除 Retrofit 以及 Dagger，减少冗余代码，编写业务模版，最终使开发速度加快 30%。  
封装模块化业务 UI 组件，以便服务端使用约定的参数动态配置业务界面。  
部分表现层的纯展示型业务采用 React Native 开发，缩短开发周期，方便快速迭代。  

### 移动端负责人 [山西联康科技有限公司](http://www.sx-uh.com/) *（ 2015年6月 ～ 2017年1月 ）*

#### [山西挂号用户版](http://sj.qq.com/myapp/detail.htm?apkName=com.uh.rdsp)
采用 Clean Architecture 架构来分离表现层、业务层和数据层。  
把应用内 Common UI、DB、Network 等模块解耦，剥离为单独组件，方便快速开发。  
优化前端数据加载的过程中，倒逼后端重构臃肿的接口，缩短数据解析时间 30%。  
在团队中积极推动 TDD 实践，把应用的 Crash 率降到 0.2% 以下。

#### [山西挂号医生版](http://sj.qq.com/myapp/detail.htm?apkName=com.uh.hospital)
利用 ⌈模块化⌋ 重构原有面条式代码，增强代码扩展能力。  
表现层采用 MVP 架构，使用 Retrofit 和 Rxjava 优雅实现和 Server 端交互。  
部分灵活展示性业务采用 hybrid - native 方法缩短开发周期，方便后期快速迭代。  
为组员普及优化应用性能和规避内存泄漏的方法，周末定期组织大家 review 关键代码。

### 安卓开发组长 [宇轩伟业科技有限公司](http://www.yuxuanweiye.com/) *（ 2014年4月 ~ 2015年5月 ）*

#### [九洲烟行零售终端](http://www.yuxuanweiye.com/product/show-120.aspx)
技术上负责整体框架搭建和购物车核心模块。帮助组内解决疑难问题，定期审查组员代码。  
管理上负责保证项目进度与质量，另外帮助组员解决同别组的资源与协作问题。

#### [食药监监管平台](http://www.yuxuanweiye.com/product/show-107.aspx)
原项目采用 Socket 在单一线程中轮询接收网络数据，当下载大量离线数据时 APP 卡顿明显。  
针对此问题，将底层重构为 HTTP/IP 协议获取数据。首次下载数据耗时从5分钟缩短为30秒，大大提升了用户体验。

## 独立开发工程师 亿地信息科技有限公司 *（ 2013年7月 ~ 2014年3月 ）*

* ### 税企 E 通
独立开发所有功能，并且负责从前期需求整理到后期产品上线与功能迭代等一系列工作，经历了产品从 0 到 1 的整个过程。

* ### 模仿网易新闻下拉刷新
在工作之余模仿网易新闻左右滑动后，下拉刷新自动触发功能。并且发到 eoe 社区 (时任源码下载区版主)。
