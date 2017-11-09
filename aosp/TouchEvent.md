# Managing Touch Events in a ViewGroup
> Managing Touch Events in a ViewGroup

```
├── View
│   ├── ViewGroup

├── View
│   ├── dispatchTouchEvent()
│   ├── onTouchEvent() 

├── ViewGroup
│   ├── this.dispatchTouchEvent() //分派事件
│   ├── onInterceptTouchEvent() //拦截事件
│   ├── super.onTouchEvent() //处理事件
```

下表省略了 PhoneWidow 和 DecorView。

> `√` 表示有该方法。
>
> `X` 表示没有该方法。

| 类型   | 相关方法                  | Activity | ViewGroup | View |
| ---- | --------------------- | -------- | --------- | ---- |
| 事件分发 | dispatchTouchEvent    | √        | √         | √    |
| 事件拦截 | onInterceptTouchEvent | X        | √         | X    |
| 事件消费 | onTouchEvent          | √        | √         | √    |

这个三个方法均有一个 boolean(布尔) 类型的返回值，通过返回 true 和 false 来控制事件传递的流程。

PS: 从上表可以看到 Activity 和 View 都是没有事件拦截的，这是因为：

> Activity 作为原始的事件分发者，如果 Activity 拦截了事件会导致整个屏幕都无法响应事件，这肯定不是我们想要的效果。
>
> View最为事件传递的最末端，要么消费掉事件，要么不处理进行回传，根本没必要进行事件拦截。

## 事件分发流程

```
WMS -> ViewRootImp -> PhoneWindow$decorView ->Activity －> PhoneWindow －> DecorView －> ViewGroup －> ... －> View

```
如果没有任何View消费掉事件，那么这个事件会按照反方向回传，最终传回给Activity，如果最后 Activity 也没有处理，本次事件才会被抛弃:

```
Activity <－ PhoneWindow <－ DecorView <－ ViewGroup <－ ... <－ View
```


关于`ViewGroup`的触摸事件，要能正确处理Touch事件。必须重写`onInterceptTouchEvent`方法。

## Intercept Touch Events in a ViewGroup
************

当`ViewGroup`检测到有事件发生时，`onInterceptToucheEvent()`将会被调用。包含`ViewGroup`和`View`。  
如果`onInterceptTouchEvent()`返回true，表示`MotionEvent`已经被拦截。他将不会下发到他的子视图。  
而是到当前`ViewGroup`的`onTouchEvent()`。

`onInterceptTouchEvent()`会拿到子视图的`MotionEvent`。  
如果`onInterceptTouchEvent()`返回true，发给子视图的事件将是ACTION_CANCEL.并且子视图的事件将会交给`ViewGroup`处理。

> 注意，ViewGroup提供了一个requestDisallowInterceptTouchEvent()方法，调用此方法后，ViewGroup将关闭拦截效果

## Use ViewConfiguration Constants
************

上面的代码片段使用了`ViewConfigutation`来获取一个变量`mTouchSlop`，你可以通过`ViewConfiguration`来获取
一些常用的距离，速度，次数。
`Touch slop`用来获取用户手指可以滑动的最大距离。用来避免一些偶然情况的发生。
另外两个`ViewConfiguration`的方法是`getScaledMinimumFlingVelcity()`和`getScaledMaximumFlingVelocity()`。这两个方法返回滑动的最小和最大速度值。

## Extend a Child View's Touchable Area（扩展子视图的可触摸面积）
android提供了一个`TouchDelegate`类，这让view的可触摸区域比本身区域更大变成可能。当子视图不得不很小时，这个类是非常有用的。如果想这么做的话，你也可以使用
此类来缩小视图的可触摸面积。

下面的例子中`ImageBotton`就是这样。

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/parent_layout"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     tools:context=".MainActivity" >

     <ImageButton android:id="@+id/button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:background="@null"
          android:src="@drawable/icon" />
</RelativeLayout>
```
下面这些代码片段做了这些事情：
* 获取其父视图并且在主线程上新建一个线程。  
这确保了在父视图调用`getHitRect()`之间已经绘制好了子视图，这个方法用于父视图定位子视图的可触摸区域。
* 根据`ImageButton`的`getHitRect`来获取可触摸的矩形区域。
* 扩展`ImageButton`所命中的可触摸的矩形区域
* 实例化`TouchDelegate`，需要一个矩形区域和视图作为参数。
* 在父视图上设置`TouchDelegate`，使用委托模式。原理：使用委托模式的情况下，
父视图将接受子视图的所有事件，当事件发生的区域在子视图区域，才会将这些事件下发给子视图。

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Get the parent view
        View parentView = findViewById(R.id.parent_layout);

        parentView.post(new Runnable() {
            // Post in the parent's message queue to make sure the parent
            // lays out its children before you call getHitRect()
            @Override
            public void run() {
                // The bounds for the delegate view (an ImageButton
                // in this example)
                Rect delegateArea = new Rect();
                ImageButton myButton = (ImageButton) findViewById(R.id.button);
                myButton.setEnabled(true);
                myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        Toast.makeText(MainActivity.this,"Touch occurred within ImageButton touch region.",
                                Toast.LENGTH_SHORT).show();
                    }
                });

                // The hit rectangle for the ImageButton
                myButton.getHitRect(delegateArea);

                // Extend the touch area of the ImageButton beyond its bounds
                // on the right and bottom.
                delegateArea.right += 100;
                delegateArea.bottom += 100;

                // Instantiate a TouchDelegate.
                // "delegateArea" is the bounds in local coordinates of
                // the containing view to be mapped to the delegate view.
                // "myButton" is the child view that should receive motion
                // events.
                TouchDelegate touchDelegate = new TouchDelegate(delegateArea,
                        myButton);

                // Sets the TouchDelegate on the parent view, such that touches
                // within the touch delegate bounds are routed to the child.
                if (View.class.isInstance(myButton.getParent())) {
                    ((View) myButton.getParent()).setTouchDelegate(touchDelegate);
                }
            }
        });
    }
}
```

# 一、概述
Android的事件分发是遵循类似责任链模式的，就是从根节点开始逐层往里分发事件，直到找到责任人（即响应事件的View）或找不到责任人事件“丢弃”为止。

# 二、触摸事件有下面一系列动作：

|   **动作**   | **描述** |
| :------------: | :----: |
|     ACTION_DOWN   |  表示用户开始触摸  |
|     ACTION_MOVE   |  表示用户在移动  |
|     ACTION_UP   |  表示用户抬起了手指  |
|     ACTION_CANCEL   |  表示手势被取消了（比如当你按下了按钮，然后移动到别处（按钮区域外）抬起，就会识别为事件取消）  |
|     ACTION_OUTSIDE   |  表示用户触碰超出了正常的UI边界  |
|     ACTION\_POINTER\_DOWN   |  有一个非主要的手指按下了  |
|     ACTION\_POINTER\_UP   |  有一个非主要的手指抬起来了  |

ACTION\_DOWN->ACTION\_MOVE->ACTION\_MOVE...->ACTION_UP.

# 三、对Touch事件的基本认知
* 一个事件只能被消费(consume)一次，事件消费了就不再传递。
* 一个事件只能有一个责任人
* 事件以按下为起点，谁消费了按下事件，后续的事件就交给谁处理。
* View可以自己处理事件，也可以分发给子View处理。

# 四、谁来处理Touch事件
![View](../img/View.jpg)

# 五、怎么处理Touch事件
事件的大概流程：事件接收层(底层：硬件和软件，一般不需要了解)--->窗口管理系统WindowManagerServicer-->因为
所有的窗口都是由他创建的，所以WMS知道当前活动的窗口是谁，WMS将事件交给当前活动窗口--->当前活动窗口拿到事件，调用
ViewRoot类的dispatchTouchEvent，给当前活动窗口的根view-->根view开始dispatchTouchEvent事件到具体view。

从根布局到子布局，即事件先传递给根布局，然后在传递给子布局

# 六、View中对事件的处理
在View中定义了跟事件处理相关的两个重要函数
![dispatchTouchEvent](../img/view_dispatchTouchEvent.jpg)

![onTouchEvent](../img/view_onTouchEvent.jpg)

# 七、ViewGroup中对事件的处理
ViewGroup是View的子类，所以自然继承了View的上述两个方法。ViewGroup还重写了`dispatchTouchEvent`方法。
ViewGroup包含了多个View，事件分发时总要先判断事件落在哪个View中，不像非ViewGroup那样简单。

# 八、Activity对Touch事件的处理
Activity持有一个Window，而Window持有一个DecorView。而事件是至上而下分发的，所以Activity对事件拥有最高的
优先处理权，它可以决定是否要将事件分发给Window。

默认情况下`dispatchTouchEvent`返回值是true，分发的；当没有任何的View来处理触摸事件时，会系统调用`onTouchEvent`方法。

# 九、总结
1. 事件的处理是至上而下的，从Activity到ViewGroup再到ViewGroup。
2. 理清事件处理关键在onDispatchTouchEvent方法。在这个方法中会做一个抉择：是要直接分发给子View处理，或是先交给自己onTouchEvent方法处理后再抉择。
3. ViewGroup作为容器类View，对事件的处理多了onInterceptTouchEvent这个阻断方法，其实我们只要看onDispatchTouchEvent就行了，因为它会在这个方法中调用onInterceptTouchEvent做是否阻断的判定。
4. 返回true，通常表示处理或消费了事件，不再传递。

# 给个图
![图裂](../img/function_touch.png)
![图裂](../img/motionEvent_activity.png)



