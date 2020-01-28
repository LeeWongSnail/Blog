## git使用之.gitconfig与.gitattribute

我们公司产品有两条线 一个是针对日本地区的一个独立APP(B),另一个是针对除日本地区之外的其他的地区的APP(A) 。B是基于A的某个分支建立的,并且删除了部分功能,在过去的一段时间内两条产品线各自为战,但是前段时间公司希望将A中的某个功能直接迁移到B上,为此我们开始了下面的工作。

对于这个任务我们分为下面三步：
1、因为B是基于A创建的所以有些基础的部分二者是相同的(双方都有修改),所以第一步是抽取公共部分。
2、在A中将希望移植的功能与A的其他功能解耦(可以来来1中公共的部分)
3、将2中抽取的功能移植到B中

额,有点跑偏了,我们这片文章主要是想介绍git的使用,但是上面为什么从重构开始说呢？那是因为在重构的过程中我们大量修改了现有工程的目录结构,之前工程大都是虚拟文件夹,重构后我们统一使用实体文件夹管理。但是我们在重构项目的过程中,还是有新的需求在不断的添加在以旧的工程目录结构为标准的项目中。在我们完成模块迁移后就发现了一个重要的问题：合代码！！！

下面我来简单的描述下这个悲伤的故事：
因为项目的目录结构被大量的修改,所以配置文件`project.pbxproj`有大量的冲突,配置文件的冲突大概分为两类：文件位置冲突、文件夹位置冲突。对于文件又可以分为:both modified、deleted by us、new file、added by us、added by them 这几大类。

先来看下`project.pbxproj`这个文件的结构：

![结构](https://upload-images.jianshu.io/upload_images/173709-e9dfbced99256edc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)


下面来简单的看下这几种冲突：

#### 文件位置冲突

![文件夹位置冲突](https://ws2.sinaimg.cn/large/006tNc79ly1g4zosdoibzj31we0f0jy9.jpg)

对于这种冲突，我们一般的解决方法是领边都保留。

#### 文件夹位置冲突

![1](https://ws3.sinaimg.cn/large/006tNc79ly1g4zovzn2gmj31we0f00xx.jpg)

![2](https://ws2.sinaimg.cn/large/006tNc79ly1g4zovvvqj5j31we0f0jwq.jpg)

![3](https://ws4.sinaimg.cn/large/006tNc79ly1g4zoxfo223j31we0f0gp1.jpg)

#### both modified 

双方都有修改

![both modified](https://ws1.sinaimg.cn/large/006tNc79ly1g4zp23crs6j31we0fmtc3.jpg)

#### deleted by us

本地分支删除(也有可能是位置被修改),但是需要注意,本地分支如果移动位置且远端分支有修改那么本地分支会标记为deleted by us 远端的标记为new file。这时候我们需要对比两个文件,保留修改。

#### new file

新增文件,但是不要被这个名称迷惑,这里面很有可能是本地分支修改了位置,但是远端分支做了修改,但是因为本地的配置文件之前的位置已经没有这个文件了,所以在合并代码的时候会被标识为new file


#### added by us

本地分支添加的文件,注意这里也有可能是从之前文件夹移动到新文件夹的文件


#### added by them

远端分支添加

### 难点

1、配置文件的合并
从上面分类中我们也可以看到,文件位置的冲突其实都好解决,但是文件夹位置的冲突,我们真的很难解决这个地方非常耗时。而且`project.pbxproj`这个文件是非常脆弱的,只要我们有一个地方没有修改正确 我们的工程都无法正常打开。所以我们一般在修改这个文件冲突的时候都选择两者都保留 或者根据文件夹的ID去查找。但是有些位置我们甚至无法做到两者都保留

![无法都保留](https://ws1.sinaimg.cn/large/006tNc79ly1g4zpgcmxkwj31we0fmjv7.jpg)

这种情况下我们为了尽快可以打开项目,然后通过编译的方式查看刚才删除文件夹。我们可能会直接删除其中一个。

2、范围的确定

因为第一步的操作过程中,给我们埋下了坑,第二步的时候需要我们找到,可能某些文件的实体文件是存在的但是项目中没有引用(第一步直接删除),当然 还有另外的可能就是:本地分支已经删除,远端还保留,那么这些文件实际上是需要删除。另外也有可能是本地分支移动了这个文件的位置导致某个文件在本地存有两份。

所以针对这种情况,我们需要仔细分析,然后在去解决冲突。

总结：根据上面的描述,我们在每次主分支发布新版本,重构代码版本需要合并新分支代码的时候,我们都要预留大概3-5天的时间去合并代码。耗时又费力。

### mergepbx

先来介绍一下使用方法：

#### 安装

可以直接使用 brew 直接安装 mergepbx

```
brew install mergepbx
```

#### 将mergepbx设置添加到〜/ .gitconfig

```objc
git config --global merge.mergepbx.name“Xcode项目文件合并”
git config --global merge.mergepbx.driver“mergepbx％O％A％B”
```

#### 配置 .gitconfig

```
[合并“mergepbx”]
    name = Xcode项目文件合并
    driver = mergepbx％O％A％B
```

#### .gitattributes

在项目的目录下(与.git同级)新建一个.gitattributes文件 同时在里面写入

```
* .pbxproj merge = mergepbx
```

#### git merge 

这时候 在使用本地分支merge远程 我们惊奇的发现 `project.pbxproj`这个该死的玩意没有冲突。感觉都已经要成功一半的样子。

### 原理

首先看下 `.gitattributes`

gitattributes文件中的每一行都是以下格式：

```
pattern        attr1 attr2 ...
```

.gitattributes实际上是定义在发生冲突时，应该采取的行动，比如merge=ours就表示文件冲突时使用原文件内容，merge=theirs表示使用其他分支的文件内容。

我们上面的定义表示 在发生冲突的时候使用mergepbx来确定冲突解决方法

然后我们在来看下 `.gitconfig`

这个文件表示 git的配置信息

我们通过

```objc
git config --global merge.mergepbx.name“Xcode项目文件合并”
git config --global merge.mergepbx.driver“mergepbx％O％A％B”
```
这两句 实际上就是在.gitconfig中添加了

```
[合并“mergepbx”]
    name = Xcode项目文件合并
    driver = mergepbx％O％A％B
```
这两行


参考文档
[pbxprojファイルのマージが便利になるmergepbxをインストールするスクリプト書いた](https://qiita.com/kaneshin/items/1deebde685c973fda6b8)

[iOSXcodeProject的内部结构分析](https://www.jianshu.com/p/50cc564b58ce)

[XCodeConfig](https://nicreals.github.io/Gear/Xcode.html)
