# Android事件分发机制
## Touch事件有如下四种：
1. MotionEvent.ACTION_DOWN：按下View（所有事件的开始）
2. MotionEvent.ACTION_MOVE：滑动View
3. MotionEvent.ACTION_CANCEL：非人为原因结束本次事件
4. MotionEvent.ACTION_UP：抬起View（与DOWN对应）

事件列：从手指接触屏幕至手指离开屏幕，这个过程产生的一系列事件。任何事件列都是以DOWN事件开始，UP事件结束，中间有无数的MOVE事件：
ACTION DOWN->ACTION MOVE->ACTION MOVE->ACTION MOVE...->ACTION MOVE->ACTION UP

当一个MotionEvent 产生后，系统需要把这个事件传递给一个具体的 View 去处理

## 事件分发的本质
即为当一个点击事件发生后，系统需要将这个事件传递给一个具体的View去处理。这个事件传递的过程就是分发过程。

## 事件传递的对象
一个点击事件产生后，传递顺序是：Activity（Window） -> ViewGroup -> View
+ View是所有UI组件的基类
+ ViewGroup是容纳UI组件的容器，即一组View的集合（包含很多子View和子VewGroup）
    + 其本身也是从View派生的，即ViewGroup是View的子类
    + 是Android所有布局的父类或间接父类：项目用到的布局（LinearLayout、RelativeLayout等），都继承自ViewGroup，即属于ViewGroup子类。
    +与普通View的区别：ViewGroup实际上也是一个View，只不过比起View，它多了可以包含子View和定义布局参数的功能。
    
![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/OnTouch1.jpg )
    
## 主要涉及的方法：

+ public boolean dispatchTouchEvent(MotionEvent ev)  		这个方法用来分发TouchEvent
+ public boolean onInterceptTouchEvent(MotionEvent ev)	这个方法用来拦截TouchEvent
+ public boolean onTouchEvent(MotionEvent ev)			这个方法用来处理TouchEvent

## 流程图：

![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/OnTouch2.jpg )
 
![](https://github.com/jy1314/Android-Knowledge/blob/master/util/picture/OnTouch3.jpg )
 
其中：

+ super：调用父类方法
+ true：消费事件，即事件不继续往下传递
+ false：不消费事件，事件也不继续往下传递 / 交由给父控件onTouchEvent（）处理



 
