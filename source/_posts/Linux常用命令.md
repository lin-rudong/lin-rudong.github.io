---
title: Linux常用命令
date: 2018-10-27 21:29:03
tags:
categories: Linux
---

# 进程类

## top
1. 实时显示系统中各个进程的资源占用状态，类似于Windows的任务管理器。

## pmap
1. 查看进程的内存占用情况。
1. pmap 1

## ps
1. 显示进程的相关信息。

# 文件类

## grep Global Regular Expression Print
1. 文件中搜索特定的内容，并将含有这些内容的行标准输出。
1. grep '\[a-z]\\{2\\}' file

## awk
1. 把文件逐行的读入，以空位或者tab为默认分隔符将每行切片，切开的部分再进行各种分析处理。
1. awk -F ':' '{print $1}' file 
    * 以:作为分隔符切片，输出每行的第一个切片。
1. awk '{printf("first:%05d\n",$1)}' file
    * 可以执行编程语句，有内置变量。

## cut
1. 处理单位为行。
1. -b -nb 以字节剪切，n表示不要分割多字节字符。
1. -c 以字符剪切。
1. -f \[-d] 以字段剪切，默认是tab分割字段，d可以自定义分割字符。
    cut -d ':' -f 2,3 file

## sed stream edit
1. sed '/^$/d' file
    * 删除空白行
1. sed '2d' file
    * 删除第2行
1. sed '/^test/'d file
    * 删除开头是test的行