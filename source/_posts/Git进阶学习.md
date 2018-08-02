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
  
### 