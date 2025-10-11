# 附录 A. Cinder 基本功能参考

本书的这一部分将帮助您找到本书中使用的 Cinder 基本功能，以便日后参考。这个参考非常基础，所以如果您是一位经验丰富的开发者，正在寻找详细和深入的功能参考，您应该查看 Cinder 网站上的参考（[`libcinder.org`](http://libcinder.org)），或者由于 Cinder 是开源的，查看 Cinder 的实际源代码。

# 基本类型

这些是许多 Cinder 函数所消耗的基本类型。这假设您已经熟悉 C++ 中的 `int`、`float` 和其他基本数据类型。

```cs
cinder::Vec2f( float x, float y )
```

上述代码表示一个由 `float` 值组成的二维向量（`x` 和 `y`）。这通常用于表示二维空间中的位置或大小。

```cs
cinder::Vec2i( int x, int y )
```

上述代码表示一个由 `int` 值组成的二维向量（`x` 和 `y`）。这通常用于表示二维空间中的位置或大小。

```cs
cinder::Vec3f( float x, float y, float z )
```

上述代码表示一个由 `float` 值组成的三个维度的向量（`x`、`y` 和 `z`）。这通常用于表示三维空间中的位置或大小。

```cs
cinder::Rectf( float x1, float y1, float x2, float y2 )
```

上述代码表示一个由左上角和右下角坐标定义的抽象矩形。

```cs
cinder::Color( float r, float g, float b, float a )
```

在上述代码中，颜色对象或类代表 Cinder 环境中的颜色——`r` 代表红色，`g` 代表绿色，`b` 代表蓝色，`a` 代表 alpha，范围从 `0.0f` 到 `1.0f`。

# 应用程序

以下函数构成了您的 Cinder 应用程序的基础：

+   `setup()`

+   `update()`

+   `draw()`

我们使用 `setup()` 进行准备，`update()` 在运行循环中进行计算，`draw()` 用于在屏幕上绘制。您可以使用 `shutdown()` 方法在我们应用程序关闭之前做一些事情（清除内存、存储数据或与远程服务器通信）。请记住，您的 Cinder 主应用程序类应该从 `BaseApp` 类派生，并且您应该首先在类声明中实现这些方法。

```cs
void YourApp::setup() {
  // setup goes here
}

void YourApp::update() {
  // prepare data
}

void YourApp::draw() {
  // draw
}
void YourApp::shutdown() {
  // do something before the app closes
}
```

如果您想更改应用程序的初始窗口大小或其他初始参数，请使用 `prepareSettings()` 方法（不要忘记首先声明此方法）：

```cs
void YourApp::prepareSettings( Settings * settings ) {
  settings->setWindowSize( 800, 600 ); // set window to 800x600 px
}
```

然后，有一些有用的函数，您可以在设置和运行时使用。

```cs
cinder::app::getWindowWidth()
```

上述函数返回应用程序窗口宽度作为一个 `int` 值。

```cs
cinder::app::getWindowHeight()
```

上述函数返回应用程序窗口高度作为一个 `int` 值。

```cs
cinder::app::getWindowSize()
```

上述函数返回应用程序窗口大小作为一个 `Vec2i` 值。

```cs
cinder::app::getFrameRate()
```

有时，有必要知道应用程序的当前帧率。上述函数以 `float` 值返回它。

```cs
cinder::app::getElapsedSeconds()
```

上述函数返回自应用程序开始以来经过的秒数，作为一个 `double` 值。

```cs
cinder::app::getElapsedFrames()
```

上述函数返回自应用程序开始以来经过的帧数，作为一个 `int` 值。

```cs
console()
```

使用上述函数进行调试，例如，`console() << "debug text" << std::endl;`。

```cs
isFullScreen()
```

前面的函数如果应用程序处于 `fullscreen` 模式则返回 `true`，否则返回 `false`。

```cs
setFullScreen( bool fullScreen )
```

前面的代码通过传递 `true` 或 `false` 设置 `fullscreen` 状态。

要使这些函数正常工作，你必须按照以下方式导入 `AppBasic` 头文件：

```cs
#include "cinder/app/AppBasic.h"
```

# 基本图形

这些函数通常在 `draw()` 方法实现中使用。

```cs
cinder::gl::clear( Color color )
```

前面的代码使用在 `Color` 中指定的颜色清除屏幕。

```cs
cinder::gl::drawLine( Vec2f from, Vec2f to )
```

前面的代码从 `from` 定义的点绘制到 `to` 定义的点绘制一条线。

```cs
cinder::gl::color( float r, float g, float b, float a )
```

前面的代码设置了用于绘制形状和线条的颜色。颜色定义为单独的 `r`、`g`、`b` 和 `a`（可选）浮点值。

```cs
cinder::glLineWidth( float width )
```

前面的代码设置了 Cinder 线绘制函数的线宽。

```cs
cinder::gl::drawSolidCircle( Vec2f center, float radius, int numSegments = 0 )
```

前面的代码在 `center` 定位处绘制了一个填充的圆，半径由 `radius` 定义。

```cs
cinder::gl::drawStrokedCircle( Vec2f center, float radius, int numSegments = 0 )
```

前面的代码仅绘制了圆的轮廓，其位置由 `center` 定义，半径由 `radius` 定义。

```cs
cinder::gl::drawSolidRect( Rectf rect )
```

前面的代码绘制了一个在 `rect` 中定义的实心填充矩形。

```cs
cinder::gl::drawStrokedRect( Rectf rect )
```

前面的代码绘制了 `rect` 中定义的矩形轮廓。

```cs
cinder::gl::drawSolidEllipse( Vec2f position, float radiusX, float radiusY, int numSegments = 0 )
```

前面的代码在 `position` 定位处绘制了一个实心填充的椭圆。x 轴上的半径由 `radiusX` 定义，y 轴上的半径由 `radiusY` 定义。`numSegments` 是可选的；它定义了用于绘制椭圆的三角形数量，因为此形状使用 OpenGL 三角形扇形绘制。如果为 `0`，扇形的数量将自动决定。

```cs
cinder::gl::drawStrokedEllipse( Vec2f position, float radiusX, float radiusY, int numSegments = 0 )
```

前面的代码仅绘制了椭圆的轮廓，其位置由 `position` 定义，x 轴上的半径由 `radiusX` 定义，y 轴上的半径由 `radiusY` 定义。并且可选地，在 `numSegments` 中定义段数。

```cs
cinder::gl::drawSolidRoundedRect( Rectf rect, float cornerRadius, int numSegmentsPerCorner = 0 )
```

前面的代码绘制了一个具有圆角的实心填充矩形。矩形由 `rect` 定义，其圆角半径由 `cornerRadius` 定义，并且可以在 `numSegmentsPerCorner` 中可选地定义用于绘制每个角落的段数。如果设置为 `0`，段数将自动决定。

```cs
cinder::gl::drawStrokedRoundedRect( Rectf, float cornerRadius, int numSegmentsPerCorner = 0 )
```

前面的代码仅绘制了具有圆角的矩形轮廓。矩形由 `rect` 定义，其圆角半径由 `cornerRadius` 定义，并且可以在 `numSegmentsPerCorner` 中可选地定义用于绘制每个角落的段数。如果设置为 `0`，段数将自动决定。

要使这些函数正常工作，你必须按照以下方式导入 OpenGL 头文件：

```cs
#include "cinder/gl/gl.h"
```

# 图片

以下是将图像加载到纹理中的基本代码：

```cs
gl::Texture myTexture = gl::Texture( loadImage( loadAsset( "image.png" ) ) );
```

此纹理随后在 `draw()` 函数中使用以下代码进行绘制：

```cs
gl::draw( myTexture, Rectf(100,100,540,380) );
```

第一个参数是包含加载图像的纹理，第二个参数是在纹理中绘制的矩形。

要使此代码示例正常工作，你必须导入以下头文件：

```cs
#include "cinder/gl/gl.h"
#include "cinder/ImageIo.h"
#include "cinder/gl/Texture.h"
```

# 其他函数

如果你对 3D、视频、声音或其他类型的函数参考感兴趣，请参阅本书的所有章节，因为这类功能使用类，这些函数的参考和解释将占用与章节相同的空间。

如果你想要更多关于 Cinder 的信息，但在这本书中找不到你想要的内容，请参考在 [`libcinder.org`](http://libcinder.org) 可用的原始 Cinder 函数参考，或者许多 C++ 或 OpenGL 语言参考之一。另一个寻找帮助的好地方是友好的 *libcinder.org* 论坛。

由于 Cinder 是开源的，如果你遇到麻烦并且觉得自己经验足够，能够找到你想要的东西，或者修复挡在你道路上的问题，查看 Cinder 的源代码是个不错的选择。

这本书只是对 Cinder 实际能做什么的一个简要介绍。
