---
title: git study
categories: git
tags: git
---
.
<!-- more -->
## **生成ssh秘钥**
```
ssh-keygen -t rsa -C "YOUR_EMAIL@YOUREMAIL.COM"
```
## 将ssh-key复制到github中
ssh-key 存放位置cat ~/.ssh/id_rsa.pub
将ssh-key复制到github中SSH keys位置即可。

## 验证github是否可用
```
localhost:Files zengguang$ ssh -T git@git.网址 
Welcome to GitLab, **!
```
## 从github中克隆项目
```
git clone git@github.com:USERNAME/PROJECTNAME.git
```
## 进目录中查看git状态
```
localhost:Files zengguang$ git status

On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

.DS_Store
others/ipscan-mac-3.4.1.zip新上传文件名字

others/wifi explorer.dmg

nothing added to commit but untracked files present (use "git add" to track)
```
## 添加将要上传的文件
```
localhost:tongxinFiles zengguang$ git add others/ipscan-mac-3.4.1.zip
localhost:tongxinFiles zengguang$ git add others/wifi\ explorer.dmg
localhost:tongxinFiles zengguang$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

new file:   others/ipscan-mac-3.4.1.zip
new file:   others/wifi explorer.dmgadd



Untracked files:
  (use "git add <file>..." to include in what will be committed)
```
## 为上传文件增加备注信息
```
localhost:tongxinFiles zengguang$ git commit -m "测试工具"添加备注信息

[master b8cef8c] 
测试工具

 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 others/ipscan-mac-3.4.1.zip
 create mode 100644 others/wifi explorer.dmg
 ```
 ## push文件
 ```
localhost:tongxinFiles zengguang$ git push origin master

Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 2.93 MiB | 0 bytes/s, done.
Total 5 (delta 1), reused 0 (delta 0)
To git.**.com:**.git
   20a7ae1..b8cef8c  master -> master
```
## git其他用法
```
git branch查看当前登陆用户
  master
* zengguang

git pull origin master
抓取远程仓库所有分支更新并合并到本地

git 删除
rm 本地库中文件
git status
git rm 目录文件
git commit -m “注释信息”
git push origin zengguang执行后无法使用git checkout命令恢复。

git fetch origin test:test   当本地无test分枝，需把远程test下载过来
git checkout test    切换test分枝
git pull origin test    获取最新代码
git checkout .    还原test分枝修改内容，以origin为准
git branch -D test        删除本地test分支（前提是要git checkout master切换到master分支后）
git fetch origin test:test        获取test分支
git checkout master 切换master分支
git merge shengchan合并shengchan分支
git checkout -b test remotes/origin/test
git log                    查看版本
git reset --hard 5610489937a49c7310c30c29ce41362b39485bb9    回滚版本

新路径上传
git init
git add .
git commit -m “.”
git remote add 地址
git push origin master
```