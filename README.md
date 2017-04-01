# iOS可视化动态绘制连通图(Swift版)
第一部分我们会画出相应的图，并该图是可以对每个点进行拖动的，在拖动的过程中，我们对其进行重绘。第二部分会取消拖动，使用UIView自带的动画来让其自己变换，当然本部分你也可以使用Timer或者GCD的TimerSource让其运动。第三部分则是第二部分的升级，再第二部分的基础上我们稍作改进，此部分我们使用的是DispatchSourceTimer来让每个点进行运动的。在第三部分我们让局部范围的点进行连线，也就是在运动的过程中，我们需要找出在当前点的规定范围内有哪些点，然后将这些点进行连接。

上述这三部分的内容下方会详细的进行介绍，并会附有相应的运行结果图。接下来就进入我们的主题部分。

 

### 一、图的绘制

在本篇博客的第一部分我们要按照要求先把图给绘制出来，我们会随机的生成几个坐标点，然后在这些坐标点上添加上View，然后再将这些坐标点使用Bezier进行连接。当然，在连接时我们使用的是邻接矩阵来记录的每两点之间的关系。在绘制的过程中，我们会随机的为每个点每条边分配颜色。

当相应的图绘制好后，我们需要为每个点添加上Move事件，在对每个点进行拖动时，我们会及时的重新绘制整个图的关系。下方就是我们本部分要实现内容的运行效果，如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222114008323-1662236809.gif)　

如果理解了数据结构中图的构建，实现上述效果，并不困难。解析来我们就来看一下实现上述效果的核心代码。

 

#### 1、图的节点View的封装

首先我们来封装上述图的节点View，当然此节点View的封装比较简单。核心就在于给每个节点View添加一个TouchesMoved事件，然后在TouchesMoved事件执行时，将触摸的移动点设置成当前View的Center即可。这样我们就可以拖动每个节点View了。在拖动节点View时，我们还需要将拖动的事件回调到节点View的父视图上，让父视图知道当前用户拖动的是哪个View。接下来我们就来看一下节点View的核心代码。

下方这段代码的上一部分就是我们定义的一个闭包类型，用来将节点View的触摸事件回调给父视图。该闭包类型需要传一个参数，该参数就是当前View的Tag, 这样父视图就知道当前用户拖动的是哪个节点了。

而randomColor()函数则是用来负责随机生成颜色的，上面每次颜色的变化都是使用的下方这个函数所随机生成的UIColor对象。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222145503307-1919382538.png)
　　

 

下方这段就是节点View的TouchesMoved事件，在该事件中我们获取到当前用户触摸移动的坐标点，然后将该点赋值给当前节点View的Center，然后调用更新父视图的闭包回调对象即可。如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222150328932-1113585708.png)
　　

 

#### 2、图View的封装

接下来我们要实现画图的View了，也就是上述节点View的父视图了。父视图主要负责的工作内容就是创建上述的节点View，然后使用Bezier将每个节点进行连接即可。当然，在用户拖动相应的View的时候，需要对当前图进行重绘。

下方这个方法就是往父视图上添加相应的节点视图，在节点视图初始化后，要设置一个闭包回调，该回调用来移动后图的重绘。在该闭包回调中，我们会调用drawLine()方法。当然在创建节点View时，我们也创建了相应的BezierPath的对象。每个节点对应一个BezierPath对象，用来绘制该节点所连节点的线。具体代码如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222150958339-403294211.png)
　　

 

我们整个图的关系是存储在邻接矩阵中的，所以我们要对邻接矩阵进行创建，在重绘时要对该邻接矩阵进行初始化。下方就是该邻接矩阵创建和初始化的代码，关于邻接矩阵的内容在此就不做过多赘述了，具体内容请参考之前的博客。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222151741182-946112528.png)
　　

 

节点View和邻接矩阵的准备工作完成后，接下来就是画线的工作了。下方就是画线的核心代码，在画线之前我们要先将相应的BezierPath对象上的点移除掉，然后再添加上新的点，最后就是进行重绘了。在往BezierPath对象上添加点时，我们要将节点的关系在邻接矩阵中进行记录。如果两个点之间已经画完线了，那么邻接矩阵上的内容我们设置为true，未画线的节点之间则是false。具体代码如下所示。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222152256917-1532959447.png)
　　

 

在上述方法调用setNeedsDisplay()方法后，就会执行View的draw()方法，我们就在此方法中进行线条的绘制。当然下方的代码比较简单，在此就不做过多赘述了。 
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222152805089-1190224129.png)

上述这些代码就是本部分所展示的效果图核心代码，完整示例请移步本篇博客末尾的github分享链接。

 

### 二、图的自动变换

上一部分是我们手动的拖动让创建的图进行变换的，接下来我们对上述代码进行改造一下，使其自动的进行变换。在点自动移动时，如果碰到屏幕的边界，我们让其反弹接着进行移动。下方就是我们本部分要实现的效果。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222114024276-1780829261.gif)
　　

当然有了第一部分作为基础，我们实现本部分的效果并不复杂。我们需要做的事情是随机生成每个节点所移动的方向。然后判断移动时是不是超出屏幕范围，如果超出屏幕范围我们就要对运动方向进行修正，让其往反方向进行移动。本部分我们只需要修改节点View，而节点View的父视图不做修改。

下方这段代码片就是为了让其自动变换所实现的方法。下方的这两个方法会替换掉第一部分的TouchesMoved方法。下方的randomIncrement()方法用来生成当前View的x坐标和y坐标的偏移量。x的偏移量为1则表示往右运动，-1表示往左运动。y的偏移量为1则往下运动，-1则是往上运行。

下方的changePoint()就是根据x和y的偏移量不断修改当前节点View的坐标的方法。为了简单，此处使用了UIView自带的Animate来实现的。在修改x和y坐标的值时要判断是否超出屏幕边距，如果超出屏幕边界就往反方向移动。为了让点一直运动下去，我们需要不断的调用changePoint()方法，如下所示。当然每调用一次changePoint()方法，我们就需要调用一下重绘的回调。具体代码如下所示。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222155030886-2130325466.png)
　　

 

### 三、特定区域内画图

接下来我们要做的就是继续在上述内容中做一些东西。在节点自动运动的过程中，我们不把所有的点都连接起来，本部分要做的事情是当点运动时，我们以改点为中心划定个区域，如果有其他点在该区域内，我们就将该区域内的点进行连接。如果点在运动的过程中超出了划定的范围，那么我们就去除之前画的线。效果如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222114026682-1249479486.gif)
　　

本部分主要修改的内容是节点View的父视图，核心就是要计算当前点与周围点的距离，如果该距离小于我们规定的距离的话，那么我们就画线，否则就不画线。下方代码片段就是本部分的核心代码。主要就是往贝塞尔上添加点时进行距离的判断。下方的countDistance()函数就是用来计算两点之间直线距离的函数，在areaPoints()中调用了该函数来确定当前区域中的点。核心代码如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222160523464-1620306226.png)
　　

 

### 四、点击新增节点

本部分也将在上述部分的代码上进行更新。该部分要做的事情是点击屏幕，往屏幕上添加新的节点。这一点在上述基础上实现是比较简单的。只需给节点的父View添加上新的节点即可。下方就是第四部分要实现的效果，每点击一次屏幕，就会在屏幕点击的地方生成一个节点，该节点就会运动。具体效果如下所示。
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222170309401-1054106911.gif)
　　

要想实现上述效果，下方是我们修改的代码片段。就是给父视图添加了一个TouchesEnded事件，在点击的地方生成一个节点View即可。具体如下所示：
![](http://images2015.cnblogs.com/blog/545446/201612/545446-20161222171851276-157454121.png)

　　
