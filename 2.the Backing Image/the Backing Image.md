
# 背景图片

<i>**一张图片抵得上千言万语，而一个交互界面胜过千万张图片。**</i>

<i>A picture is worth of a thousand words.An interface is worth of a thousand pictures. --Ben Shneiderman</i>

## 内容图片

CALayer有一个叫做contents的属性。此属性的类型被定义为id类型，也就意味着此属性可以是任何对象。对，你可以将你想要的任何对象赋值给此属性，而且也可以编译通过，但是实际上，除了CGImage之外的任何对象，你的layer都是一片空白。

contents属性的这种怪异设定要归咎于Core Animation在Mac OS上的遗留问题。contents属性定义为id类型是因为在Mac OS上，你赋值一个CGImage或者NSImage对象给它，它都可以正常显式。但是如果你在iOS上赋值一个UIImage对象给它，它还是一片空白。这也是导致那些对Core Animation并不熟悉的iOS开发者产生疑惑的普遍原因。

然而让人头疼的地方还不止这些。你真正需要应用的对象实际上是一个CGImageRef的对象，这是一个指向CGImage结构体的指针。UIImage对象有一个属性返回这样一个其潜在的CGImageRef。然而，如果你尝试将其直接赋值给CALayer的contents属性，它还编译不过，因为CGImageRef并不是一个Cocoa对象，而是一个Core Foundation的类型。

尽管Core Foundation的类型在运行时和Cocoa对象表现相似，它们和id类型依然不是类型兼容的，除非你使用一个bridged cast进行转换（也就是我们所熟知的自由桥接：toll-free bridging）。要将一个图片赋值给contents属性，你需要这样做：
```objectivec
layer.contents = (__bridge id)image.CGImage;
```
如果你没有使用ARC的话，你就不需要使用__bridge这一部分。但是，请告诉我 *你为什么不用ARC？！*

下面我们修改一下我们第1章里面的创建的项目，来展示一张图片而不是单纯的背景颜色。我们不再需要额外的寄宿层，前面咱们已经讲过了，纯代码方式手工创建layer层是可以的，那么我们就直接把图片赋值给我们创建好的layerView背景层的contents属性。

List 2.1 将层的contents属性设置为图片
```objectivec
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //load an image
    UIImage *image = [UIImage imageNamed:@"Snowman.png"];
    //add it directly to our view's layer
    self.layerView.layer.contents = (__bridge id)image.CGImage;
  }
@end
```
![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.1.png)

这几行代码非常简单，但是我们这里所做的事情却非常有趣：使用CALayer的功能，我们在一个普通的UIView里显示了一张图片。它是一个UIImageView对象，也不是被设计用来正常现实图片的。通过直接操作层，我们已经揭露出新的功能，并使我们简陋的UIView变得有趣了一些。

### contentsGravity

你可能已经注意到了我们的雪人看上去有点儿肥。我们加载的图片并非严格的正方形的，所以为了和视图大小相符，它被拉伸了。可能你在使用UIImageView的时候见过类似的情景，解决的办法可能就是将此视图的contentMode属性设置为更合适的值，比如这样：
```objectivec
view.contentMode = UIViewContentModeScaleAspectFit;
```

这个方法在这里也一样适用（试一下吧），UIView里大部分视觉上的属性比如contentMode，在CALayer中也的确有与之相对应的操作属性。

CALayer中和contentMode等价的属性是contentsGravity，但是这是一个字符串NSString类型的对象，而不是像UIKit里的枚举类型。contentsGravity属性只能被赋值为以下常量值：

<table border="none">
    <tr><td>kCAGravityCenter</td><td>kCAGravityTopRight</td></tr>
    <tr><td>kCAGravityTop</td><td>kCAGravityBottomLeft</td></tr>
    <tr><td>kCAGravityBottom</td><td>kCAGravityBottomRight</td></tr>
    <tr><td>kCAGravityLeft</td><td>kCAGravityResize</td></tr>
    <tr><td>kCAGravityRight</td><td>kCAGravityResizeAspect</td></tr>
    <tr><td>kCAGravityTopLeft</td><td>kCAGravityResizeAspectFill</td></tr>
<table>

和contentMode一样，contentsGravity也是用来决定对应的内容在layer范围内如何排列的。我们将使用和UIViewContentModeScaleAspectFit具有同样功效的kCAGravityResizeAspect，所产生的效果就是不修改图片外部比例进行缩放来和layer层的区域相适配：
```objectivec
self.layerView.layer.contentsGravity = kCAGravityResizeAspect;
```
![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.2.png)

### contentScale

contentScale属性定义的是图层的背景图片和视图对象大小之间的像素级别的缩放比例。这是一个双精度的值，默认值为1.0。

有时候设置contentScale并不能达到立竿见影的效果，它并不是总能对屏幕上的图片有缩放的效果；如果你尝试将我们雪人的项目例子进行各种值的设置，你会发现根本就不起作用，因为contents的背景图片已经使用contentsGravity属性设置为了按照视图的大小进行缩放匹配了。

如果你仅仅只是想要对图层的内容图片进行缩放，可以使用图层的transform或者affineTransform属性（请参见解释“变换”的第5章“变换”），但是那也不是contentScale想要达到的效果。

实际上属性contentsScale是为了实现支持高分辨率（也就是我们所熟知的Hi-DPI或者Retina）屏幕的机制的一部分。在进行绘制layer层的时候，用它来确定其背景图的尺寸大小，对应的值也就是内容图片所需要展示的伸缩倍数（假定其contentGravity属性还没进行设置）。UIView还有一个与之等价但是几乎没有被用过的属性contentScaleFactor。

如果contentScale被设置的值为1，那么也就是1个像素占一个点进行绘制。如果设置为2，那就是2个像素一个点，也就是广为人知的视网膜Retina分辨率。（为了防止你不清楚像素和点之间的区别，本章的后续内容会有解释）

这种做法和使用kCAGravityResizeAspect没有任何区别，因为使用后者是不管分辨率是多少，都对图片进行缩放，使图片和layer层大小相匹配。但是如果我们换种做法，将contentsGravity设置为kCAGravityCenter（这种做法不会对图片进行伸缩），区别就会更加明显了（如图2.3）。

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.3.png)

如你所见，我们的雪人变的很大而且被像素化。因为CGImage和UIImage不一样，它根本就没有缩放的实质性概念。当我们使用UIImage来加载雪人图片的时候，它就可以正确地加载高质量的Retina版本。但是当我们使用CGImage进行展示，并将其赋值给我们layer层的contents属性时，在转换的过程中缩放的因素就被丢掉了。此时我们就可以手工设置contentsScale的值，来和我们的UIImage的scale属性相匹配（参见代码清单2.2）。结果如图2-4所示。
```objectivec
@implementation ViewController
- (void)viewDidLoad {
  [super viewDidLoad]; //load an image
  UIImage *image = [UIImage imageNamed:@"Snowman.png"];
  //add it directly to our view's layer
  self.layerView.layer.contents = (__bridge id)image.CGImage;
  //center the image
  self.layerView.layer.contentsGravity = kCAGravityCenter;
  //set the contentsScale to match image
  self.layerView.layer.contentsScale = image.scale;
}
@end
```
![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.4.png)

在对代码生成的背景图片的时候，通常需要注意设置layer层的contentsScale属性来匹配屏幕比例，要不然的话你的图片在Retina屏设备上就会出现像素化。我们可以这样进行设置：
```objectivec
layer.contentsScale = [UIScreen mainScreen].scale;
```

### masksToBounds

目前我们的雪人显示大小没问题，但是你可能注意到还有些其他的问题--雪人超出了视图本身的区域大小。默认情况下，UIView很乐意绘制出正确的内容，即便其子视图超出了它本身被设定的区域大小。对与CALayer而言也是一样的。

UIView有一个叫做clipsToBounds的属性，可以被用来启用或禁用“裁剪”（即控制是否需要对超出其区域范围的内容进行裁剪）。CALayer也有一个与之等价的属性：masksToBounds。将其设置为YES，我们的雪人就会被限制住了（如图2.5）。

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.5.png)

### contentsRect

CALayer的contentsRect属性允许我们用来设置背景图片在layer层的区域里面，哪一部分的矩形区域用来被显示。相比于contentsGravity属性，就对图片的裁剪和伸缩而言，这种做法的灵活度更高，

跟bounds、frame所不一样的是，contentRect并不是以点进行计量的，而是使用单位坐标系。单位坐标系被指定的大小范围是0到1，取的是相对值（和像点、像素使用的绝对值相反）。这种情况下，它们和背景图片的尺寸大小就是相对的。iOS系统使用了以下几种坐标类型：

* <b>Points</b>--iOS和Mac OS中最普遍使用的坐标系类型。点（Points）是虚拟的像素，也被称作逻辑像素。在标准化的设备上，1个点就是1个像素，但是在Retina设备商，1个点有2x2个物理像素。iOS就是使用点对整个屏幕做计量的，所以在Retina和非Retina设备上进行排版布局没有差别。

* <b>Pixels</b>--物理像素坐标系并不用来进行屏幕排版布局，但是通常在对图片处理时是相对的。UIImage是能感知屏幕分辨率，并使用“点”指定其尺寸，但是一些底层的图片展示方式（例如CGImage）则使用像素坐标，所以你就要注意了，它们所声明使用的尺寸大小和在Retina屏幕上显示出来的大小是不一致的。

* <b>Unit</b>--对于计量图片、layer层区域大小的相对大小来说，使用单位坐标系是一种非常合适的方法，而且也不需要因为其本身的大小发生变化而进行调整。单位坐标系在OpenGL中用得非常多，比如处理类似纹理坐标系，在Core Animation中也使用较多。

默认的contentsRect是{0,0,1,1},即默认情况下整个图片都是可见的。如果我们指定一个小一点的矩形块，图片就会被裁剪了（如图2-6）。

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.6.png)

事实上也可以将contentsRect赋值为一个负数或者范围大于{1,1}值，这种情况下，就会使用图片以外的像素进行延伸来填满剩余的其他区域。（试了一下，效果如下图：）

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.6.extend.png)

contentsRect最有趣的应用之一，就是被称作图片精灵（image sprites）的应用。如果你做过游戏开发，那对“精灵”的概念应该很熟悉，那就是一个可以在屏幕中和其他“精灵”分开、可以独立移动的图片。但是在游戏世界之外，这个词通常是用来指代一种用来加载精灵图片的常用技术，和移动什么的没有任何关系。

比较典型的，就是许多的精灵被打包进一个单独的大图中，可以一次全部加载。在内存使用、加载时间、渲染的性能方面，这样做比使用多个单独的图片文件来讲，有很多好处。

精灵Sprites可以被用在类似Cocoas 2D之类的2D游戏引擎中，其所使用的就是OpenGL来展示图片。但是我们也可以在普通的UIKit应用程序中使用contentsRect来显示“精灵”图片。

开始之前，我们需要一个精灵表单--一张包含了我们需要的小精灵的大图片。如图2.7就是一个精灵表单的例子。

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.7.png)

接下来，我们就需要在我们的APP中加载并显示这些小精灵。我们的原则很简单：加载我们的大图片，然后将其赋值给四个不一样的layer层（每个layer一个精灵），然后分别设置它们的contentsRect属性掩盖掉我们不需要的部分。

我们那还需要为我们小精灵的layers层添加额外的视图。（为了防止让代码变得臃肿，我们直接使用IB拖拽的方式创建，如果你喜欢使用代码创建也是可以的）。代码参见2.3,最后的结果参见2-8.

Listing 2.3 Splitting Up a Sprite Sheet Using contentsRect
```objectivec
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *coneView;
@property (nonatomic, weak) IBOutlet UIView *shipView;
@property (nonatomic, weak) IBOutlet UIView *iglooView;
@property (nonatomic, weak) IBOutlet UIView *anchorView;

@end

@implementation ViewController
- (void)addSpriteImage:(UIImage *)image withContentRect:(CGRect)rect
 toLayer:(CALayer *)layer //set image
{
      layer.contents = (__bridge id)image.CGImage;
      //scale contents to fit
      layer.contentsGravity = kCAGravityResizeAspect;
      //set contentsRect
      layer.contentsRect = rect;
}

- (void)viewDidLoad {
      [super viewDidLoad];
      //load sprite sheet
      UIImage *image = [UIImage imageNamed:@"Sprites.png"];
      //set igloo sprite
      [self addSpriteImage:image withContentRect:CGRectMake(0, 0, 0.5, 0.5) toLayer:self.iglooView.layer];
      //set cone sprite
      [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0, 0.5, 0.5) toLayer:self.coneView.layer];
      //set anchor sprite
      [self addSpriteImage:image withContentRect:CGRectMake(0, 0.5, 0.5, 0.5) toLayer:self.anchorView.layer];
      //set spaceship sprite
      [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0.5, 0.5, 0.5) toLayer:self.shipView.layer];
}

@end

```
![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.8.png)

精灵表单是一种减少app大小和加载时长的非常巧妙的方式（单个大图文件比多个小文件压缩率更好、加载更快），但是手动布局就显得笨重，而且一旦精灵表单创建好了，如果你要添加新的精灵或者要修改已有精灵的位置，就需要额外的维护成本。

已经有多款商业应用可以用来在你的Mac上自动创建精灵表单，这些工具通过自动创建XML和包含这些小精灵坐标心的属性文件，简化了精灵使用。随后这个文件会和对应的图片一并被加载，并用于设置contentsRect，而不需要开发者自己写代码去处理精灵的位置信息。

这些文件通常是被设计用来做OpenGL游戏的，但是如果你很想用在一般的应用里面的话，欢迎使用开源库[LayerSprites](https://github.com/nicklockwood/LayerSprites)，这个库就可以用来读取知名的Cocoas2D格式的精灵表单，并且可以使用普通的Core Animation层来展示这些小精灵。

### contentsCenter

本章我们最后需要探讨的关于contents相关的属性就是contentsCenter了。通过这个属性的名称，你可能觉得这个是用来处理contents图片位置的，但是实际上，这个属性的名称也是有误导性的。contentsCenter实际上是一个CGRect的矩形结构体，用来指定layer层内部的可拓展部分，从而将其作为layer层边缘的混合边框的。修改contentsCenter属性对于背景图的显示没有什么影响，但是一旦layer层的大小发生变化，对应的变化就会很明显了。

默认情况下，contentsCenter的值为{0,0,1,1},背景图片会和layer层的大小一起伸缩（当然，这也取决于contentsGravity属性）。但是如果我们增加contentsCenter位置对应的值（CGRect中前两个数值，即（x,y）），减少大小对应的值（CGRect中后两个数值，即(width,height)）,我们就可以在图片的周边创建出来边框了。如图2-9就显示了contentsCenter值为{0.25,0.25,0.5,0.5}的效果：

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.9.png)

这也就意味着我们可以任意的修改视图的大小，其边框也都会一致（如图2.10）。这种方法达到的效果和UIImage的``` -resizableImageWithCapInsets: ```方法是一样的，但是任何这种方法可以被应用于任何layer层背景图片，包括使用Core Graphics在运行时绘制的layer（本章后续会有介绍）。

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.10.png)

代码清单2.4展示了纯代码方式设置视图的伸展区的。当然不用编写任何代码，直接在IB的Stretching面板中直接进行定义，如图2.11.

Listing 2.4 <b>Setting Up Stretchable Views Using contentsCenter</b>

```objectivec
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *button1;
@property (nonatomic, weak) IBOutlet UIView *button2;
@end
@implementation ViewController
- (void)addStretchableImage:(UIImage *)image withContentCenter:(CGRect)rect toLayer:(CALayer *)layer
{
    //set image
    layer.contents = (__bridge id)image.CGImage;
    //set contentsCenter
    layer.contentsCenter = rect;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    //load button image
    UIImage *image = [UIImage imageNamed:@"Button.png"];
    //set button 1
    [self addStretchableImage:image withContentCenter:CGRectMake(0.25, 0.25, 0.5, 0.5) toLayer:self.button1.layer];

    //set button 2
    [self addStretchableImage:image withContentCenter:CGRectMake(0.25, 0.25, 0.5, 0.5) toLayer:self.button2.layer];
}

@end
```

![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.11.png)

### 自定义绘图

给layer层的contents赋值一个CGImage并非唯一填充背景的方式，直接使用Core Graphics进行绘制也是可以的。在UIView的子类中实现```-drawRect: ```方法就可以进行自定义绘图。

方法``` -drawRect: ```没有默认的实现，因为如果UIView已经使用某个填充色或者其背景层的contents属性已经有对应的图片进行填充了，就不需要自定义绘制的背景图了。如果UIView检测到``` -drawRect: ```方法的存在，就会创建一个新的背景图层，其对应的像素大小等于视图view的大小乘以其contentsScale的大小。

如果你不需要背景图还要创建，就是浪费内存和CPU，这也是为什么苹果公司推荐的，如果你不需要自定义绘制背景的话，就把UIView的子类中的``` -drawRect: ```方法留空就好了。

在视图view第一次出现在屏幕上的时候就会自动调用方法``` -drawRect: ```。方法``` -drawRect: ```内的代码直接使用Core Graphics进行绘图，对应结果在视图需要刷新之前会一直缓存在内存中（通常视图进行刷新是因为开发者手动调用了方法``` -setNeedsDisplay ```，当然，视图对象的一些能够引起其外貌发生变化的属性，例如bounds，发生变化的时候，也会自动进行刷新）。尽管``` -drawRect: ```申明为UIView的方法，但实际上，其底层还是属于CALayer，用来处理绘图并存储结果图片的。

CALayer有一个可选的delegate代理属性，遵循的协议为CALayerDelegate。当CALayer需要内容相关的信息时，就会从其代理请求对应的内通信息。CALayerDelegate是一个非正式协议，换句话来讲，在你的类接口中，并没有实际的CALayerDelegate @protocal 可以让你来引用。你只需要添加你需要的方法，CALayer就会调用你所提供的方法。（对应的delegate属性被申明为id类型，所有的代理方法也都被当成可选的。）

一旦需要重绘，CALayer就会请求其代理对象提供背景图片进行展示。CALayer会尝试调用以下方法进行请求绘制展示：
```objectivec
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```

再调用此方法前，CALayer会创建一个合适大小（根据layer层的bounds和contentsScale属性）的空背景图，并通过传入的参数ctx获取合适的Core Graphics绘图上下文进行绘制。

我们修改一下第一章的测试项目，然后实现CALayerDelegate协议，来画一下吧（代码清单2。5）。结果如2.12.

Listing 2.5 <b>Implementing the CALayerDelegate</b>

```objectivec
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];

    //create sublayer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    blueLayer.backgroundColor = [UIColor blueColor].CGColor;
    //set controller as layer delegate
    blueLayer.delegate = self;
    //ensure that layer backing image uses correct scale
    blueLayer.contentsScale = [UIScreen mainScreen].scale;
    //add layer to our view
    [self.layerView.layer addSublayer:blueLayer];
    //force layer to redraw
    [blueLayer display];
}

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx {
    //draw a thick red circle
    CGContextSetLineWidth(ctx, 10.0f);
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextStrokeEllipseInRect(ctx, layer.bounds);
}
@end
```
![](https://raw.githubusercontent.com/BladeTail/ios_core_anination-advanced_techniques/master/2.the%20Backing%20Image/2.12.png)

注意一下以下几件有趣的事：

* 我们必须手工调用一下bluerLayer的``` -display ```方法，强制让它刷新。和UIView不一样的是，CALayer不会在它显示出来的时候自动重绘，而是要等开发者的指示来确定是否需要重绘。

* 即使我们没有设置masksToBounds属性，我们的圆圈还是被layer层的区域大小给切割了。这是因为我们使用CALayerDelegate绘制背景图的时候，CALayer创建的绘图上下文就和layer层的大小是一样的。也没有指定好的规定说如何来绘制超出这个范围的部分。

现在你理解并知道如何使用CALayerDelegate了吧。但是除非你要自己创建单独的layer层，要不然的话你几乎永远不需要实现这个协议。这是因为UIView在创建其背景层的时候，就已经将自己设置为其背景层delegate代理了，并且提供了``` -displayLayer: ```的实现将这些都抽象封装进去了。

在使用视图背景层的时候，是不需要实现方法``` -displayLayer: ```或者``` -drawLayer:inContext: ```来绘制layer的背景图的，只需按照正常的套路实现方法``` -drawRect: ```即可，UIView会自己处理所有的事情，包括在需要重绘的时候自动调用layer的``` -display ```。

## 总结

本章讨论了layer层的背景图及其相关属性。你已经学习了放置和裁剪图片，从精灵表单中取出单独的图片，已经利用CALayerDelegate和Core Graphics的强大功能绘制层的contents。

在第3章“图层几何”中，我们将一探究竟layer层的几何特性，并检查layer层之间是如何放置，以及大小相关的处理。
