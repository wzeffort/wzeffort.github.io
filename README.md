+++
date = '2024-12-07T00:22:09+08:00'
draft = false
title = '第一篇Hugo文章'
+++

## 简介

个人博客盧瞳小站的Github仓库

个人站点[`2061360308.github.io`](//2061360308.github.io)

## 使用方法
### 日常编辑文章
如果需要创建新的文章可以使用[**github.dev**](//github.dev/2061360308/2061360308.github.io/tree/main)进行编写

仅仅是修改文章可以直接在github仓库下选择对应文章打开后点击编辑即可

### 网站配置
网站的配置需要切换到hugo分支进行修改

更改网站配置可以编辑`config.yaml` [Github在线编辑](//github.com/2061360308/2061360308.github.io/edit/hugo/config.yaml)

如果需要更换主题可以编辑`.gitmodlues`文件来更改子项目配置 [Github在线编辑](//github.com/2061360308/2061360308.github.io/edit/hugo/.gitmodlues)

>**注意**：hugo分支的更改提交默认不会触发Github Pages的更新，需要手动激活Workflow
>简单修改也可以使用[**github.dev**](//github.dev/2061306030/2061360308.github.io/tree/hugo)

## 构建说明

> 【分支】
> **hugo**分支下保存的是hugo创建的网站文件以及选用的主题Hugo NexT文件
> **main**分支下仅仅保存文章文件

项目已经配置了Workflow用来一键自动部署到Github Pages流程如下：
 - 会自动先检出hugo分支
 - 之后将main分支下的文章检出到./content/post文件夹下
 - 构建hugo网站
 - 发布到Github Page

## 其他
访问 [Hugo](https://gohugo.io) 网站！
