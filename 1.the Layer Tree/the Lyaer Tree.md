# 层级树

<i>**妖怪有层，洋葱也有层。现在你明白了吧？我们都有层。**</i>

<i>**Ogres have layers，Onions have layers。You get it？We both have layers。**</i>

<i>**--Shrek**</i>

Core Animation的名字是有误导性的。读者可能会根据它的名字认为，它目的是来做动画的，但是实际上“动画”只是此框架的一个方面，甚至还只是Layer Kit以动画为中心的的一个名称而已。

Core Animation是一个组合式的引擎，它的工作是将屏幕上可见的、不同的块，以其最快的速度组合在一起，而这些“块”的内容就被分成独立的层在一个树状结构里存储起来的，也就是我们所熟知的层级树（the Layer Tree）。而这棵树也就组成了UIKit，以及我们在屏幕上所看到的iOS应用里可见的所有事物的基础。

在我们开始讨论动画之前，我们先从层级树开始，来看看Core Animation的静态组成部分和布局特性。

## Layers和Views（层和视图）

如果你曾经创建过iOS或者Mac OS下的应用程序，你肯定很熟悉“视图”的概念。一个“视图”就是一个用来显示内容（比如图片，文字，或者视频）的矩形对象，同时还能和用户的输入进行交互，比如鼠标点击或者触摸手势。“视图”可被嵌套进另外一个“视图”，从而形成一个树状结构，里面的每一个“视图”都各自管理着其子视图对象的位置。如图1.1，就是一个典型的视图树：

![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.1.png)

在iOS中，所有的视图都继承自一个通用的基类UIView。UIView处理触摸事件，也支持基于Core Graphics的绘图，线性变换（例如旋转和缩放），还有一些诸如滑动和渐变的简单动画。

你没意识到的是，大部分的这些任务其实都不是UIView自己处理的。渲染、布局、动画这些，都是由Core Animation中一个叫做CALayer的类来管理的。

### CALayer

CALayer类在概念上和UIView很相似。层，类似于视图，也是可以以树状结构进行分配的矩形对象，也能包含类似图片、文本、背景色的内容，同时也管理其子对象的位置。他们还有方法和属性来执行动画和变形。UIView里唯一最主要的不是由CALayer进行处理的特性就是<i>用户交互</i>。

CALayer并不知道“响应链”（iOS用来在视图树里传递触摸事件的机制），所以无法响应事件，虽然CALayer提供了方法用来判断触摸点是否在其显示的区域范围内（更多信息请参见第三章“层的几何”）。

### 平行树

每一个UIView都一个CALayer的实例属性layer。这也就是我们所知的<i>背景层</i>。该视图负责创建和管理此属性“层”，并保证视图树中子视图的添加和移除的同时，其对应的背景层在层级树中平行地进行关联（如图1.2）。

![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.2.png)

事实上就是这些背景层负责屏幕上可见事物的展示和动画。UIView仅仅只是进行封装的包装，以提供iOS下特定的功能，诸如触摸处理，以及Core Animation的底层方法的高级封装。

为什么iOS要有UIView和CALayer两个这样平行的树状结构呢？为什么不是一个树状结构对所有的东西进行处理呢？原因是想要进行功能区分，从而防止重复的代码。事件和用户交互工作在iOS上和Mac OS上有很大的不同。基于多点触摸的用户交互和一个鼠标与键盘式的相比，在最基础的范式（请参见[wikipedia](https://zh.wikipedia.org/wiki/%E8%8C%83%E5%BC%8F)）上就有所不同,这也就是为什么iOS有UIKit和UIView，而Mac OS有AppKit和NSView。他们在功能上相似，但是实现上却完全不同。

绘图、布局和动画相对应的概念，就类似于iPhone和iPad设备上的触摸所产生的交互行为，和在笔记本和台式机上所产生的交互行为。通过将这些逻辑功能区分开来并封装进对的Core Animation框架中，Apple（苹果公司）可以在iOS和Mac OS上共用同样的代码，对于苹果公司自己的操作系开发团队也好，对于那些在这两个平台上开发应用的第三方开发者也好，事情也都变的更加简单了。

事实上，不是两个树状结构，而是四个，每一个都扮演着不同的角色。在“视图”树结构和“层”树结构之外，还有“展示”树和“渲染”树，这也是我们将在第7章“隐式动画”和第12章“速度调优”中分别进行讨论的，

## 层的功能

那么如果CALayer仅仅只是UIView内部细节的实现，我们为什么要对它了解的那么清楚呢？诚然，苹果公司提供了精细友好而简单的UIView的接口，难道这样我们就不需要直接和Core Animation最原始的细节打交道了吗？

在一定程度上来讲的确是的。就一些简单的目的而言，我们确实不需要对CALayer直接进行处理，因为苹果公司已经高度封装了一些功能特性，如动画效果，从而使之变得简单易用，直接通过UIView接口使用高级的API接口，就可以间接地使用。

但是“简单”所带来的就是灵活性的缺失。如果你想要做一些通常性以外的一些事情，或者要使用苹果公司没有在UIView中暴露出来的一些接口，此时你就别无选择，只能迎入Core Animation框架怀抱，去寻求需要的底层选项了。

我们已经说过“层”无法处理用户触摸但是“视图”可以，那么有什么是“层”可以做但是“视图”却不可以做的呢？以下这些CALayer的特性就是没有被暴露在UIView中的：

+投影，圆角，颜色边框
+3D变换和位移
+非矩形边界
+内容的alpha掩码
+多步的、非线性动画

我们会在以下的章节里来探索这些特性，但是现在我们先来看看在app中怎么样利用CALayer。

## 使用CALayer

我们先来创建一个可以操作layer属性的简单工程。在XCode中，选择模板Single View Application创建一个iOS项目。

在屏幕的中心位置创建一个小视图（大概200x200点point），使用代码或者Interface Builder拖拽都行，你觉得怎么样舒服就怎么样来。但是要确保在视图控制器中有一个对应的属性与之相关连，这样的话就好直接对这个小视图进行访问了。姑且先称之为layerView吧。

如果你此时运行这个工程，就可以看见，在一个浅灰色背景的中央有一个白色的小方块（如图1.3）。如果看不见的话，那可能就要调整一下窗体或者视图的对象的背景色了。

![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.3.png)

这也没没觉得怎么样，那咱们就来加点染色。我们将在白色的小方块中加一个蓝色的小方块。我们可以通过创建一个新的蓝色的UIView，并将其作为子视图加到我们刚刚创建的白色视图对象中，来达到这种效果，但是这并没有告诉我们关于层的任何东西。

所以咱们换个方式，我们创建一个CALayer的对象，将其作为子层加入到我们的视图背景层中。尽管在UIView类里面暴露出来了layer这个属性，但是标准的XCode的iOS应用项目模板并没有包含Core Animation的头文件，因此在引入适当的框架到项目里之前，我们还不能使用layer的任何方法或者访问其相关属性。那么我们首先得在应用构建目标target的Build Phase标签tab中添加QuartzCore框架（如图1.4），然后在视图控制器的.m文件中添加代码：
<code>
import &lt;QuartzCore/QuartzCore.h>
</code>.

![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.4.png)

之后，我们就可以在代码中直接使用对应的CALayer的属性和方法了。代码清单1.1中，我们使用代码手工创建了一个CALayer，设置好它的背景色属性backgroundColor，然后将其添加到layerView的背景层的子层中（假设我们使用的是在IB中拖拽的方式创建的layerView，并且关联好了UIOutlet）。运行如图1-5所示。

Listing 1.1 向view添加蓝色的子layer
```objectivec
#import "ViewController.h"
#import &lt;QuartzCore/QuartzCore.h>
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *layerView;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //create sublayer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    blueLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```

![](https://github.com/BladeTail/ios_core_anination-advanced_techniques/blob/master/1.the%20Layer%20Tree/1.5.png)

一个视图对象仅有一个背景层（自动创建的），但是却可以拥有无数的子层。如代码清单1-1所示，你可以显式地创建单独的layers层，然后直接把它们作为子层添加到视图的背景层中。尽管以这种方式添加layer是可行的，但是通常情况下你只需使用视图对象和它们已有的背景层就够了，并不需要手工添加额外的子层。

在10.8之前版本的Mac OS中，使用背景层的视图树，而不是在单独视图中寄宿的layer层级树，会有很严重的性能影响。但是在iOS里面使用轻量级的UIView类，在对层进行处理的时候，几乎没有什么性能上的负面影响。（在Mac OS 10.8中，NSView的性能已经有了大幅度的提升了）

使用背景层为基础的UIView而不是寄宿的layer层级树的好处是，我们既可以访问CALayer的底层特性，同时也还能使用UIView类所提供的高级API接口（例如自动大小变化，自动布局，事件处理）。

但是，有可能因为一下几个远影，在现实中，你可能还是想使用寄宿的layer层级树而不是基于背景层的视图树：

* 你可能想要编写同时还能在Mac OS运行的跨平台代码
* 你可能想要使用多个CALayer的子类（参见第6章“特殊的层”），但是又不想创建新的UIView的子类来对他们进行封装。
* 你可能在做一些处于性能临界点的工作，即使在UIView上做任何一点及其微小的修改都会产生可观的性能差别（在这种情况下，你可能更想用类似于OpenGL的东西来进行图形绘制）。

但是这些可能性都是微乎其微的，而且通常来讲，相对于寄宿的layer层级树而言，基于背景层的视图对象处理起来也要更简单。

## 总结

这一章我们探索了层级树，一个平行于形成iOS界面的UIView视图树底部CALayer对象构成的层级树。我们还创建了我们自己的CALayer对象并将其添加到层级树中作为实验。

在第2章“背景图片”中，我们会对CALayer的背景图片，以及对应Core Animation所提供的用来操作展示的属性一探究竟。
