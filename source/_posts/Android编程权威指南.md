---
title: Android编程权威指南
date: 2018-11-04 10:00:25
tags:
---

# Android开发初体验
1. 应用由activity和layout组成。activity负责管理用户与信息屏的交互；layout定义了一系列用户界面对象以及它们显示在屏幕上的位置，组成布局的定义保存在XML文件中。
1. activity定义widget，组件是用户界面的构造模块。
1. android:text属性值一般不硬编码为字符串，而是对字符串资源的引用，字符串资源包含在一个独立的名为strings的XML文件中。
1. Android的toast是用来通知用户的简短弹出消息，用户无需输入或进行任何操作。
1. setContentView方法会实例化布局文件中定义的每一个View对象。

# Android与MVC设计模式
1. activity作为控制器，向内通过业务逻辑和数据获得模型状态，向外展现视图布局。
1. 模型对象，表示应用的状态，即数据和业务逻辑，通常用来映射与应用相关的一些事物。
1. 视图对象，负责与用户进行交互。
1. 控制对象，应用的逻辑单元，是视图和模型的联系纽带，管理模型与视图间的数据流动。

# Activity的生命周期
1. FrameLayout是最简单的ViewGroup醉驾案，它不以特定方式安排其子视图的位置，子视图的位置排列取决于它们各自的gravity。
1. 在应用运行中，只要设备配置发生了改变，就会销毁当前activity，然后再创建新的activity。
1. Android回收内存时，只有在暂停或停止状态才可能会被销毁，此时，会调用onSaveInstanceState方法将用户数据保存在Bundle对象中，然后操作系统将Bundle对象放入activity记录中。
1. 后退键会彻底销毁当前的activity，并清除暂存的activity记录，系统重启或长时间不使用activity也会清除暂存的activity记录。

# 第二个activity
1. intent对象是component用来与操作系统通信的一种媒介工具。
1. ActivityManager维护者一个非特定应用独享的回退栈。

# Android SDK版本与兼容
1. 最低版本代表兼容性，编译版本代表新特性。
1. 目标版本告知Android应用设计的目的版本。

# UI fragment与fragment管理器
1. fragment会让UI设计变得灵活，也会让应用变得复杂。
1. fragment的生命周期方法必须是公共方法，因为托管fragemnt的activity要调用它们。
1. 总是使用fragment，但是应用单屏至多使用3个fragment。
1. 优先使用支持库版fragment，因为使用Android操作系统内置版fragment会降低兼容性，不过支持库版fragment会占用内存空间。

# 使用布局与组建创建用户界面
1. dp/dip，密度无关像素，1dp单位在设备屏幕上总是等于1/160英寸，无论屏幕密度如何，总能获得同样的尺寸。
1. sp，缩放无关像素，它是一种与密度无关的像素，这种像素会受用户系统字体大小设置的影响，通常用来设置屏幕上的字体大小。
1. 布局参数，以layout_开头的属性作用于组件的父组件，它们会告诉父布局如何在内部安排自己的子元素。
1. android:layout_weight属性的工作原理，在layout_width分配完空间后，剩余空间会按照layout_weight设置的值的占比分配，通常会设定所有子组件的layout_weight属性值总和为100。

# 使用RecyclerView显示列表 
1. RecyclerView负责回收再利用以及定位屏幕上的View，而定位功能委托给LayoutManager。
1. ViewHolder负责容纳View。
1. Adapter作为Controller，负责创建必要的ViewHolder，绑定ViewHolder至模型层数据。

# 使用fragment argument
1. 每个fragment实例都可附带一个Bundle对象，该bundle包含键值对。
1. fragment能够从activity中接收返回结果，但其自身无法持有返回结果，只有activity拥有返回结果。
1. Adapter的notifyDataSetChanged方法会通知RecyclerView刷新全部的可见列表项。而只需要刷新单个列表项的话，可以用Adapter的notifyItemChanged方法。

# 使用ViewPager
1. FragmentStatePagerAdapter代理，负责管理与ViewPager的对话并协同工作，代理首先需要将getItem返回的fragment添加给托管activity，并帮助ViewPager找到fragment的视图并一一对应。
1. FragmentStatePagerAdapter会彻底销毁fragment，而FragmentPagerAdapter只是销毁fragment的视图，fragment永远不会被销毁，适合fragment很少的情况。

# 对话框
1. 用DialogFragment封装AlertDialog，方便管理。


