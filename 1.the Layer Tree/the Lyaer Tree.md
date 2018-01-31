# 层级树

<font color=gray size=5>妖怪有层级，洋葱也有层级。现在你明白了吧？我们都有层级。
Ogres have layers，Onions have layers。You get it？We both have layers。
--Shrek
</font>

Core Animation的名字是有误导性的。读者可能会根据它的名字认为，它目的是来做动画的，但是实际上“动画”只是此框架的一个方面，甚至还只是Layer Kit以动画为中心的的一个名称而已。

Core Animation是一个组合式的引擎，它的工作是将屏幕上可见的、不同的块，以其最快的速度组合在一起，而这些“块”的内容就被分成独立的层在一个树状结构里存储起来的，也就是我们所熟知的层级树（the Layer Tree）。而这棵树也就组成了UIKit，以及我们在屏幕上所看到的iOS应用里可见的所有事物的基础。

在我们开始讨论动画之前，我们先从层级树开始，来看看Core Animation的静态组成部分和布局特性。

## Layers和Views（层和视图）

如果你曾经创建过iOS或者Mac OS下的应用程序，你肯定很熟悉“视图”的概念。一个“视图”就是一个用来显示内容（比如图片，文字，或者视频）的矩形对象，同时还能和用户的输入进行交互，比如鼠标点击或者触摸手势。“视图”可被嵌套进另外一个“视图”，从而形成一个树状结构，里面的每一个“视图”都各自管理着其子视图对象的位置。如图1.1，就是一个典型的视图树：
![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.1.png)
