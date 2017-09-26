---
layout: post
title: "Git 命令行使用"
date: 2017-09-26 09:08:00 +0800 
categories: iOS
tag: CocoaPods
---
* content
{:toc}

Mac 环境下CocoaPods 搭建。

<!-- more -->

## 1 安装ruby
   （1）curl -L https://get.rvm.io | bash -s stable
   （2）source ~/.rvm/scripts/rvm
   （3）检查安装是否成功 rvm -v

## 2 安装 CocoaPods
   （1）如果已经安装过CocoaPods，并且版本过低，先删除：gem uninstall cocoapods
   （2）安装：gem install cocoapods
   （3）pod setup 更新Pod源

## 3 编写 Podfile 示例
```javascript
   	source 'https://github.com/CocoaPods/Specs.git'
	platform :ios, '10.0'
	use_frameworks!

	target "ShopList" do

	pod 'Alamofire', '~>4.0'
	pod 'RealmSwift'

	end
```

## 4 执行 pod install/update