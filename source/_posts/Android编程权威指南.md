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

# 工具栏

## AppCompat
1. 原生工具栏无法支持比较老的版本，不过Google将它移植到了AppCompat库，这样都可以使用工具栏。

**使用AppCompat库**
1. 添加AppCompat依赖项; 使用一种AppCompat主题; 确保activity都是AppCompatActivity子类.
1. AppCompat自带三种主题, Theme.AppCompat黑色, Theme.AppCompat.Light浅色, Theme.AppCompat.Light.DarkActionBar黑色工具栏的浅色; 应用级别的主题设置在AndroidManifest.xml文件中application标签的android.theme属性; 主题定义在styles.xml文件中.

## 工具栏菜单

**在XML文件中定义菜单**
1. 菜单是一种类似于布局的资源, 菜单定义文件在res/menu目录; 菜单定义文件一般布局文件同名; showAsAction属性用于指定菜单项是显示在工具栏上, 还是隐藏于溢出菜单.
1. app命名空间, 出于兼容性考虑, AppCompat库需要使用app命名空间, 提供了定制版showAsAction属性.
1. 应用使用的图标有两种, 系统图标和项目资源图标, 系统图标是Android操作系统内置的图标, android:icon属性值@android:drawable/ic_menu_add就引用了系统图标; 不同设备或操作系统版本间, 系统图标的显示风格差异很大, 不能统一应用的界面风格; 一种解决方案是创建定制图标, 针对不同屏幕显示密度或各种可能的设备配置, 准备不同版本的图标; 另一种解决方案是找到适合应用的系统图标, 将它们直接复制到项目的drawable资源目录中; 系统图标可以在Android SDK目录下找; 第三个也是最容易的解决方案, 使用内置的Android Assert Studio工具, 生成图像后, 通过@drawable引用.

**创建菜单**
1. Activity类提供了管理菜单的回调函数, 需要选项菜单时, Android会调用Activity的onCreateOptionsMenu方法; Fragment有一套自己的选项菜单回调函数, 创建菜单和响应菜单项选择事件的两个回调方法, onCreateoptionsMenu和onOptionsItemSelected; onCreateOptionsmenu中调用MenuInflater.inflate创建菜单文件中定义的菜单项目; 创建菜单方法中也调用了超类的方法, 作为一项开发规范, 推荐这么做; 创建菜单方法是由FragmentManager负责调用的, 当activity接收到操作系统的回调请求时, 我们必须明确告诉FragmentManager, 其管理的fragment应接收调用指令, 通知FragmentManger需在onCreate中调用setHasOptionsMenu.

**响应菜单项选择**
1. 用户点击菜单项时, fragment会收到onOptionsItemSelected的回调请求, 传入参数是描述用户选择的MenuItem实例, 可以用switch来判断id; 方法返回的是布尔值, true表示任务已完成, 如果id不存在, 调用超类方法.

## 实现层级式导航
1. 简单的应用主要靠后退键在应用内导航, 后退键导航又称为临时性导航, 只能返回到上一次浏览过的用户界面, 而层级式导航可在应用内逐级向上导航; AndoridManifest.xml中添加parentActivityName属性, 开启层级式导航.
1. 层级式导航的工作原理, 会指示Android在回退栈中寻找指定的activity实例, 如果实例存在, 则弹出栈内所有其他activity.

## 可选菜单项
1. getString可以替换字符串资源中的占位符; 要访问工具栏, 将activity转换成AppCompatActivity.
1. activity.invalidateOptionsMenu重建工具; 考虑后退键情况; 层级式导航回退到的目标activity会被完全重建, 父activity是全新的, 实例变量值以及保存的实例转台显然会彻底丢失; 一种方案是在启动子activity时, 把状态作为extraextra信息传给它, 然后在子activity中覆盖getParentActivityIntent, 用附带了extra信息的intent重建父activity; 解决设备旋转问题, 只要利用实例状态保存机制, 在onSaveInstanceState中保存实例变量值就能解决问题.

# SQLite数据库
1. Android设备上的应用都有一个沙盒目录, 将文件保存在沙盒中, 可阻止其他应用甚至是设备用户的访问和窥探, 如果设备被root了, 用户就可以为所欲为; 应用的沙盒目录是/data/data/应用的包名称; 需要保存大量数据时, 大多数应用不会使用类似txt的普通文件, 因为在仅需要修改少量数据时, 就得首先读取整个文件的内容, 完成修改后再全部保存, 数据量大的话非常耗时; SQLite是开源关系型数据库, 不同于其他数据库的是, SQLite使用单个文件存储数据, 读写数据时使用SQLite库.

## 定义schema
1. database schema描述表名和数据字段; 首先创建定义schema的Java类; 在Schama类中, 再定义一个描述数据表的Table静态成员类; Table类唯一的用途就是定义描述数据表元素的String常磊, 定义数据库表名; 接下来定义数据表字段, 在Table类中创建静态成员类.

```Java
public class Schema{
    public static class Table{
        public static final String NAME="...";
        
        public static class Cols{
            public static final String UUID="uuid";
            ...
        }
    }
}

```

## 创建初始数据库
1. openOrCreateDatabase和databaseList是Android提供的Context底层方法, 用来打开数据库文件并将其转换为SQLiteDatabase实例.
1. 实践中步骤, 一确认目标数据是否存在; 二如果不存在, 首先创建数据库, 然后创建数据表并初始化数据; 三如果存在, 打开并确认Schema是否为最新; 四如果是旧版本, 就先升级到最新版本.
1. 以上工作可借助Android的SQLiteOpenHelper类处理; 调用getWritableDatabase时, 会打开沙盒下的数据库文件, 如果不存在, 就先创建数据库文件; 如果是首次创建数据库, 就调用onCreate, 然后保存最新的版本号; 如果已创建过数据库, 首先检查它的版本号, 如果Dababase类中的版本号更高, 就调用onUpgrade升级.
1. SQLite数据库表省事又简单, 创建表字段时, 不需要指定表字段类型.

```Java
public Database extends SQLiteOpenHelper{
    private static final int VERSION=1;
    private static final String DATABASE_NAME="database.db";

    public Database(Context context){
        super(context, DATABASE_NAME, null, VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db){
        db.execSQL("create table "+Table.NAME+"("+
        " id integer primary key autoincrement, "+
        Table.Cols.UUID+", "+
        ...
        ")");
    }

    @Override
    public void onUngrade(SQLiteDatabase db, int oldVersion, int newVersion){
    }
}
```

**使用Android Device Monitor查看文件**
1. 需要禁用ADB整合, 如果运行应用, 出现Instant Run requires ADB的错误, 关闭Instant Run.

**处理数据库相关问题**
1. 碰到要调整数据库表结构的情况, 常规做法是, 在SQLiteOpenHelper中记录版本号, 然后在onUpgrade升级数据表; 常规方法编写和维护很少更新版本的代码不好, 最好的做法是直接删除数据库文件, 再让onCreate重建; 直接从设备上删除应用是删除数据库文件最便捷的方法.

## 写入数据库
1. 写操作有增和改.

**使用ContentValues**
1. 负责处理数据库写操作的辅助类是ContentValues, 它是一个键值存储类; 将记录转换成ContentValuse; ContentValues的键就是数据表字段.

**插入和更新记录**
1. insert插入数据; update更新数据.

## 读取数据库
1. query读取数据,返回Cursor.

**使用CursorWrapper**
1. Cursor是表数据处理工具,其功能是封装数据表中的原始字段值;考虑到代码复用,可以扩展CursorWrapper类来封装Cursor对象,映射Cursor结果集.

**创建模型层对象**
1. 数据库cursor总是指向查询的某个地方,因此要从cursor中取出数据,首先要调用moveToFirst指向第一个元素,读取行记录后,再调用moveTonext读取下一行记录,直到isAfterLast表明数据可取为止;最后调用Cursor的close方法关闭.
1. 刷新模型层数据.

# 隐式intent
1. 在Android系统中,可利用隐式intent启动其他应用的activity.

## 使用格式化字符串
1. 应用运行前,无法获知的字符串,使用带有占位符的格式化字符串.

## 使用隐式intent

**隐式intent的组成**
1. 隐式intent的主要组成部分,要执行的操作,通常以Intent类中的常量表示,比如访问URLIntent.ACTION_VIEW和发送邮件Intent.ACTION_SEND;待访问数据的位置,可能是设备之外的资源,比如网页URL或者文件URI,或者是指向ContentProvider中某条记录的某个内容URI;操作涉及的数据类型,指的是MIME形式的数据类型,如果intent包含数据位置,那么通常可以从中推测出数据的类型;可选类别,通常用来描述何时,何地后者如何使用某个activity.
1. 通过配置文件中的intent过滤器设置,activity对外宣称自己是适合处理某种操作的activity;必须在intent过滤器中明确地设置DEFAULT类别,DEFAULT告诉操作系愿意处理某项任务.

**发送消息**
1. 使用隐式intent启动activity时,也可以创建每次都显示的activity选择器,用Intent.createChooser创建.

**获取联系人信息**
1. 隐式intent将有操作以及数据获取位置组成,操作为Intent.ACTION_PICK,联系人数据获取位置为ContactsContract.Contacts.CONTENT_URI;获取结果调用startActivityForResult.
1. 从联系人列表中获取联系人数据;ContentProvider类处理联系人信息,该类的实例封装了联系人数据库并提供给其他应用使用,可以通过ContentResolver访问它.
1. 联系人信息使用权限.

**检查可响应任务的activity**
