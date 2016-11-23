---
title: Xcode多工程联编及工程依赖
tags:
  - iOS
  - workspace
categories:
  - iOS配置
date: 2016-03-12 23:47:36
---


在了解xcode构建原则之前，需要熟悉workspace相关的概念，即workspace,project和target。

### 一:target

target指定了构建的product,包含将workspace或project中的一组文件构建成product的指令。单个target定义一个product,它将输入（源文件和处理这些源文件的指令（包含所设定的构建settings和phases））组织进构建系统中。project和target是一对多的关系。

target会继承project的build settings，但可以以target为粒度设置build settings,且Xcode中当前一次只会有一个有效target,此有效target在xcode scheme中标识。

若target A依赖target B的输出来构建，则A依赖B，当它们存在于一个xcode workspace中的时候，Xcode会发现此依赖关系并按顺序构建。这种依赖称为隐性依赖，当然也可以在build settings中声明显性依赖，也可将隐性依赖的两个target显性声明为没有依赖。比如可能在同一个workspace中同时构建一个库并构建一个依赖这个库的应用，则xcode会选择先构建库。但如果应用想链接库的某个特定版本，则可以显性声明这个依赖关系，此时隐性依赖即被覆盖了。

<!-- more -->

### 二:project

xcode project是存储构建一个或多个软件产品所需的所有文件，资源和信息的仓库。
project包含构建product所需的所有要素并维护它们之间的关系，它包含一个或多个targets(target指定了构建product的方式)。
project定义了所有target的默认build settings，各target可以重载。Xcode project文件包含如下信息：源文件的引用：源代码（包括头文件和实现文件），library和framework(xcode内部或者外部),资源文件，interface builder文件文件结构列表中组织源文件的Groupsproject级别的build选项targetsdebug或测试program的可执行环境：从xcode运行或调试时启动哪些可以执行文件，传给可执行文件的命令行参数，程序运行时设置的环境变量总之，project可以单独存在也可以包含在workspace中，同时可以在Scheme中指定哪个Target、build配置、哪个可执行配置在某个时刻是有效的。

### 三:build settings

一个build setting是一个指示产品某个方面构建方式的变量，比如决定xcode传给编译器的参数选项是怎样。其是一个常量或者一个公式供给xcode在构建的时候计算build setting。

### 四:workspace

workspace 是组织projects和其他协同工作的文档的一份文档。除此之外，它还维护project和target之间的显性及隐性的依赖关系。默认workspace中的所有Xcode projects都在同一个目录下构建，称为workspace build directory，由于所有project的所有文件都在同一个目录下，所以所有文件都对每个project可见。
比如两个project使用同一个库，则不用复制到另一个project目录中。workspace中的每个project都有其独立id,同时project可以属于多个workspace，可以单独打开project或者在其他workspace中打开，且都不用重新配置project或者workspace。
可以使用workspace默认的build 目录，也可以指定一个。如果project指定了构建目录，这个目录会被project构建时所在的workspace的build目录覆盖。

### 五:xcode schemexcode 

scheme定义了一系列构建的targets，构建时的配置，和一系列执行的测试。可以有很多scheme,但同一时刻只能有一个有效的。选择scheme时，意味着你也选择了一个运行目标（product构建的硬件平台)。可以指定scheme是否存储在project中，以便包含此project的所有workspace都可以使用些scheme,当然也可以指定只存储在某个workspace中。

一般的某个应用单独新建一个 project 就可以了，然后把所有的程序文件都放在里面，这个可以满足大部分普通的需求，但是有时候，项目有可能要使用其他的项目文件，或者引入其他的静态库文件，这个时候 workspace 就派上用场了，workspace 即可以单独管理多个项目，又可以通过配置，让各个项目相互依赖，如果不用 workspace，以前的做法是如果用到其他项目的文件，要手动 copy 文件到当前的项目，在 workspace 里这个步骤不需要了。

### 六:案例

下面是我自己的例子 ，现在用 workspace 管理2个 project，其中一个是 static library: MyStaticLib，另外一个是依赖这个静态库的 project:  MyUseStatic，菜单 xocde7 > file > New Workspace 新建一个空的workspace，名字可以随便取。
![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/1.png)

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/2.png)

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/3.png)

在左边 project navigator 右键菜单 New Project ,然后选择 Ios > Framework & Library > Cocoa Touch Static Library , 然后输入项目名称 MyStaticLib，这样就新建了一个空白的静态库项目，接着新建个类文件，名字为 MyLib, 选中 MyLib.h头文件，打开右边的 File inspector 窗口，在 Target membership 中将 MyStaticLib 后面的 project改成 public 。

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/4.png)

在左边的 project navigator 右键菜单 New Project ,然后选择 Ios > Application > Single View Application，然后输入项目名称 MyUseStatic，下面配置让它依赖 MyStaticLib，打开 Build Phases配置选项 然后展开 Link Binary With Libraries ，点击 + 会看到 Workspace > libMyStaticLib.a ，选中它，就让此项目产生了对 MyStaticLib的依赖关系，然后在 Build S Setting配置选项里 搜索 USER_HEADER_SEARCH_PATHS，将它的值设为 MyStaticLib 的build prouect 路径，在import静态库中的新文件时，会到这个路径中寻找。

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/6.png)

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/XcodeWorkSpace/7.png)
现在编译  MyUseStatic 会自动先编译依赖的 MyStaticLib。

参考文章：http://blog.carbonfive.com/2011/04/04/using-open-source-static-libraries-in-xcode-4/#set_the_installation_directory

[!更多文章可前往我的blog：http://gcblog.github.io](http://gcblog.github.io)






