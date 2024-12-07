+++
date = '2024-12-07T00:22:09+08:00'
draft = false
title = '第一篇Hugo文章'
+++

<div style="display: flex; align-items: center;">
  <img src="https://github.com/2061360308/2061360308.github.io/blob/hugo/static/imgs/avatar.jpg?raw=true" alt="头像" style="width: 100px; height: 100px; margin-right: 20px; border-radius: 50%;">
  <h1 style="margin: 0;">盧瞳小站</h1>
</div>

## 个人博客《盧瞳小站》的Github仓库

请到 [`2061360308.github.io`](//2061360308.github.io) 浏览构建后的站点

## 使用方法
> 此博客项目完全依赖于Github Workflow在云端进行自动化构建，目前已部署到Github Pages

编辑文章：访问[github.com/2061360308/2061360308.github.io](https://github.com/2061360308/2061360308.github.io) 仓库直接在线添加货编辑Markdown文件即可

网站配置：可以编辑[`config.yaml`](//github.com/2061360308/2061360308.github.io/edit/hugo/config.yaml)

主题更改：直接修改[`.gitmodlues`](//github.com/2061360308/2061360308.github.io/edit/hugo/.gitmodlues)文件来更改子项目配置

可以使用[github.dev]()体验更完整的云IDE环境（速度较慢）[main分支](//github.dev/2061360308/2061360308.github.io/tree/main)、[hugo分支](//github.dev/2061306030/2061360308.github.io/tree/hugo)


## 构建说明

> 【分支】
> **hugo**分支下保存的是hugo创建的网站文件以及选用的主题Hugo NexT文件
> **main**分支下仅仅保存文章文件

项目已经配置了Workflow用来一键自动部署到Github Pages流程如下：
 - 会自动先检出hugo分支
 - 之后将main分支下的文章检出到./content/post文件夹下
 - 构建hugo网站
 - 发布到Github Page

**注意**：hugo分支的更改提交默认不会触发Github Pages的更新，需要手动激活Workflow

## 本地使用
此项目在云端即可运行，如果需要克隆到本地测试请按照下面步骤进行

1. 安装配置Hugo
2. 克隆项目的hugo分支
3. 克隆 main 分支到 content/posts 目录
4. 更新子模块

**Linux下命令示例**
```shell
# 克隆 hugo 分支
git clone -b hugo https://github.com/2061360308/2061360308.github.io.git
cd 2061360308.github.io

# 删除 content/posts 目录（如果存在且不包含重要内容）
rm -rf content/posts

# 克隆 main 分支到 content/posts 目录
git worktree add content/posts main

# 更新子模块（如果有）
git submodule update --init --recursive
```

**Windows Powershell下命令示例**
```shell
# 克隆 hugo 分支
git clone -b hugo https://github.com/2061360308/2061360308.github.io.git
cd 2061360308.github.io

# 删除 content/posts 目录（如果存在且不包含重要内容）
rmdir /S /Q content\posts

# 克隆 main 分支到 content/posts 目录
git worktree add content/posts main

# 更新子模块（如果有）
git submodule update --init --recursive
```

## 相关链接
[Hugo](https://gohugo.io)
[Hugo中文文档](https://hugo.opendocs.io/)
[Hugo NexT演示站](https://hugo-next.eu.org/)
[Hugo NexT Github](https://github.com/hugo-next/hugo-theme-next/)
