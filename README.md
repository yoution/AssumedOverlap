参照![compositing的文章](http://taobaofed.org/blog/2016/04/25/performance-composite/), 学习compositing过程中，发现AssumedOverlap原因比较抽象，所以深入研究了下。
composite过程是沿着paintLayer进行深度遍历，子paintLayer根据stackingContext的顺序进行遍历, 
paintLayer根据compositing 直接原因和compositing间接原因判断是否需要compositing, 需要compositing的paintLayer会生成dui对应的graphicsLayer。
而overlay和AssumedOverlay都是同一个stackingContext上才会发生的
* compositing 直接原因
 见![CompositingReasonFinder.cpp](https://github.com/bloomberg/chromium.bb/blob/master/src/third_party/WebKit/Source/core/layout/compositing/CompositingReasonFinder.cpp)
 * CompositingReasonVideoOverlay
 * kCompositingReasonRoot
 * ......
* compositing 间接原因
1, 64, 128, 256 8192 16382
1 * kCompositingReasonVideoOverlay，父级路径上最近的能compositing的paintLayer的layoutObject为<video>
64 kCompositingReason3DTransform，如,kPerspective, transform:perspective(400px),transform: translateZ(10)，transform:scaleZ(0.5)等
 > * kCompositingReasonRoot, layoutObject为HTMLDocument对应的paintLayer
kCompositingReasonActiveAnimation, 包括opacityAnimation,transformAnimation,filterAnimation,backDropFilterAnimation
8192 kCompositingReasonWillChangeCompositingHint,包括opacity,transform, translate, scale, rotate, top, left, bottom, right
16382 kCompositingReasonBackdropFilter，即 backdrop-filter
Overlay
AssumedOverlap

遍历stackingContext时当前paintLayer的前面的paintLayer不包transformAnimation  或者 当前paintLayer有compositing直接原因，则当前paintLayer的产生AssumedOverlap reason 效果
