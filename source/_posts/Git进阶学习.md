---
title: Git进阶学习
date: 2018-08-02 18:21:05
tags:
categories:
copyright:
---
## Git基础补充
  
### 状态简览
  
```
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt
```
  
`??`表示文件未跟踪，` M`表示文件被修改但未放入暂存区，`M `表示文件被修改并放入暂存区，`MM`表示文件在工作区被修改并提交到暂存区后又在工作区中被修改了，`A`表示新添加到暂存区的文件。  
  
### 忽略文件
  
.gitignore格式规范：  
  
* 所有空行或者以#开头的行都会被Git忽略。
* 可以使用标准的glob模式匹配。
* 匹配模式可以以(/)开头防止递归。
* 匹配模式可以以(/)结尾指定目录。
* 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号(!)取反。
  
### 查看已暂存和未暂存的修改
  
查看未暂存文件的修改：  
  
```
git diff
```
  
查看已暂存文件的修改：  
  
```
git diff --cached
```
  
或  
  
```
git diff --staged
```
  
### 移除文件
  
删除未暂存的文件：  
  
```
git rm 
```
  
删除已暂存的文件：  
  
```
git rm -f
```
  
删除已暂存的文件，文件保存在工作区：  
  
```
git rm --cached
```
  
### 移动文件
  
```
git mv file_from file_to
```
  
### 查看提交历史
  
查看提交历史中每次的变化：  
  
```
git log -p
```
  
查看近2次的提交历史：  
  
```
git log -2
```
  
查看提交历史中近2次的变化：  
  
```
git log -p -2
```
  
查看提交简略变化：  
  
```
git log --stat
```
  
按照不同格式查看提交历史：  
  
```
git log --pretty=oneline
```
  
```
git log --pretty=short
```
  
```
git log --pretty=full
```
  
```
git log --pretty=fuller
```
  
按照某种格式查看提交历史：  
  
```
git log --pretty=format:"%h - %an, %ar : %s"
```
  
git log --pretty=format常用选项：  
  
| 选项 | 说明 |
|------|-----|
| %H | 提交对象（commit）的完整哈希字串 |
| %h | 提交对象的简短哈希字串 |
| %T | 树对象（tree）的完整哈希字串 |
| %t | 树对象的简短哈希字串 |
| %P | 父对象（parent）的完整哈希字串 |
| %p | 父对象的简短哈希字串 |
| %an | 作者（author）的名字 |
| %ae | 作者的电子邮件 |
| %ad | 作者修订日期（可以用 --date= 选项定制格式） |
| %ar | 作者修订日期，按多久以前的方式显示 |
| %cn | 提交者（committer）的名字 |
| %ce | 提交者的电子邮件地址 |
| %cd | 提交日期 |
| %cr | 提交日期，按多久以前的方式显示 |
| %s | 提交说明 |
  
git log的常用选项：  
  
| 选项 | 说明 |
|-----|------|
| -p | 按补丁格式显示每个更新之间的差异 |
| --stat | 显示每次更新的文件修改统计信息 |
| --shortstat | 只显示--stat中最后的行数修改添加移除统计 |
| --name-only | 仅在提交信息后显示已修改的文件清单 |
| --name-status | 显示新增、修改、删除的文件清单 |
| --abbrev-commit | 仅显示SHA-1的前几个字符，而非所有的40个字符 |
| --relative-date | 使用较短的相对时间显示（比如，“2 weeks ago”） |
| --graph | 显示ASCII图形表示的分支合并历史 |
| --pretty | 使用其他格式显示历史提交信息 |
  
### 限制输出长度
  
限制git log输出的选项：  
  
| 选项 | 说明 |
|-----|------|
| -(n) | 仅显示最近的n条提交 |
| --since, --after | 仅显示指定时间之后的提交 |
| --util, --before | 仅显示指定时间之前的提交 |
| --author | 仅显示指定作者相关的提交 |
| --committer | 仅显示指定提交者相关的提交 |
| --grep | 仅显示指定关键字的提交 |
| -S | 仅显示添加或移除了某个关键字的提交 |
