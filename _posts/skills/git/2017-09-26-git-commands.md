---
layout: post
title: "Git 命令行使用"
date: 2017-09-26 09:08:00 +0800 
categories: git
tag: git
---
* content
{:toc}

git 命令行使用

<!-- more -->

## 1 科隆代码
   git clone https://github.com/huangshaobing/huangshaobing.git

## 2 新增文件
   #增加全部新增，修改文件
   git add . 
   #增加某个文件
   git add index.html

## 3 提交
   git commit -m '修改文件'

## 4 push 
   git push origin master  #push 到master分枝

   #强制覆盖提交
   git push --force origin master
   git push -f

   
