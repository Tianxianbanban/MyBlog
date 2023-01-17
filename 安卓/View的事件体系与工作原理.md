# Android View



>  结合思维导图回忆。



## View的事件体系

#### **View的事件分发机制**

首先事件分发机制分发的是**MotionEvent事件**，也就是点击事件，是当MotionEvent事件产生以后，系统需要把这个事件传递给一个具体的View并且得到处理的过程。 事件产生后的传递过程是从Activity--->Window--->View的隧道式传递，首先传递给 Activity ，Activity 会传递到 PhoneWindow 上，PhoneWindow 会传递给 RootView，而 RootView 其实就是 DecorView ，接下来便是从 DecorView 到 View 上的分发过程了，对于View和ViewGroup的事件分发过程稍有不同：

当事件分发到当前 **ViewGroup** 的时候，首先会调用它的 **dispatchTouchEvent** 方法，在 dispatchTouchEvent 方法里面会调用 **onInterceptTouchEvent** 来判断是否要拦截当前事件，如果要拦截的话，就会调用 ViewGroup 自己的 onTouchEvent 方法了，如果 onInterceptTouchEvent 返回 false 的话表示不拦截当前事件，那么事件会继续往当前 ViewGroup 的子 View 上面传递，如果它的子 View 是 ViewGroup 的话，就继续重复 ViewGroup 事件分发过程，如果子 View 就是 View 的话，就按照View 的事件分发过程进行传递。

事件分发给**View**，首先当然也是执行**dispatchTouchEvent** 方法，但是如果我们为当前 View 设置了 **onTouchListener** 监听器的话，首先就会执行它的回调方法onTouch，这个方法的返回值会决定事件是否要继续传递下去：如果返回 false，表示事件没有被消费，还会继续传递下去；如果返回 true，表示事件已经被消费了，不需要向下传递了；如果返回 false，那么将会执行当前 View 的 **onTouchEvent** 方法， 如果我们为当前 View 设置了 **onLongClickListener** 监听器的话，则首先会执行他的回调方法 onLongClick，和 onTouch 方法类似，如果该方法返回 true 表示事件被消费，不会继续向下传递，返回 false 的话，事件会继续向下传递，假定返回 false，如果我们设置了 **onClickListener** 监听器的话，则会执行他的回调方法 onClick，该方法是没有返回值的，所以也是我们事件分发机制中最后执行的方法了；可以注意到的一点就是只要你的当前 View 是 clickable 或者 longclickable 的，View 的 onTouchEvent 方法默认都会返回 true，**也就是说对于事件传递到 View 上来说，系统默认是由 View 来消费事件的**，但是 ViewGroup 就不是这样了。

上面的事件分发过程是正常情况下的，但是如果有这样的情况，比如**事件传递到最里层的 View 之后，调用该 View 的 onTouchEvent 方法返回了 false**，那么这时候事件将通过冒泡式的方式向他的父 View 传递，调用它父 View 的 onTouchEvent 方法，如果正好他的父 View 的 onTouchEvent 方法也返回 false 的话，这个时候事件最终将会传递到 Activity 的 onTouchEvent 方法了，也就是最终就只能由 Activity 自己来处理了。

事件分发还需要注意的是： 

(2) 正常情况下，**一个事件序列**只能被一个View拦截且消耗。当ViewGroup决定**拦截事件**后，那么后续的点击事件将会默认交给它处理并且不再调用它的onInterceptTouchEvent方法（有一种特殊情况，那就是FLAG_DISALLOW_INTERCEPT标记位，这个标记位是通过requestDisallowInterceptTouchEvent方法设置，ViewGroup将无法拦截除了ACTION_DOWN以外的其他点击事件，ACTION_DOWN事件到来时会重置FLAG_DISALLOW_INTERCEPT标志位。）。（但是通过特殊手段可以做到，比如一个View将本该自己处理的事件通过onTouchEvent强行传递给其他View处理。）

 (3) **如果一个 View ==开始处理事件==但是==又没有消费掉 DOWN 事件==，那么这个事件序列随后的事件也将不再由这个View 来处理**，事件将重新交由它的父元素去处理，即父元素的onTouchEvent会被调用。

如果View不消耗除ACTION_DOWN以外的其他事件，那么这个**点击事件会消失**，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终这些消失的点击事件会传递给Activity处理。

ViewGroup默认不拦截任何事件。View没有onInterceptTouchEvent方法。但是View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。

事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。

(1) **如果说除 Activity 之外的 View 都没有消费掉 DOWN 事件的话，那么事件就不会再传递到 Activity 里面的子 View 了，会直接由Activity调用自己的onTouchEvent方法来处理**；

(4) View的onTouchEvent方法是否执行是和它的onTouchListener回调方法onTouch 的返回值息息相关，如果onTouch 返回 true，那么onTouchEvent 方法就不会去执行；而如果onTouch 返回 false， onTouchEvent 方法执行，然后因为 onTouchEvent 里面会执行 onClick，所以造成了 onClick 是否执行和 onTouch 的返回值有关系。 



#### 从屏幕到APP

1. 硬件与内核

   当屏幕被触摸，`Linux`内核会将硬件产生的触摸事件包装为`Event`存到`/dev/input/event[x]`目录下。输入事件封装为通用的`Event`，供后续处理

2. SystemServer

   SystemServer进程会启动一系列系统服务，如AMS,WMS等，还有一个就是我们管理事件输入的InputManagerService，负责与硬件通信，接受屏幕输入事件。其内部会启动一个读线程`InputReader`，它会从系统也就是`/dev/input/`目录拿到任务，并且分发给`InputDispatcher`线程，然后进行统一的事件分发调度。

3. 跨进程通信传递给APP

   App`中的`Window`与`InputManagerService`之间的通信实际上使用的`InputChannel。（Activity`启动时会调用`ViewRootImpl.setView()，这个过程中，会同时注册`InputChannel`）

#### 从APP到页面

在App进程中拿到输入事件，分发到页面。

1. 事件回传到ViewRootImpl

   `ViewRootImpl`不仅负责界面的绘制，同时负责事件的传递。

2. 组装责任链

   `ViewRootImpl#setView`方法中，就把这条输入事件处理的责任链拼接完成了，不同的`InputStage`子类通过构造方法一个个串联起来了。

   事件到达应用端的主线程，会通过`ViewRootImpl`进行一系列`InputStage`来处理事件。这个阶段其实是对事件进行一些简单的分类处理，比如视图输入事件，输入法事件，导航面板事件等等。（我们的`View`触摸事件就发生在`ViewPostImeInputStage`阶段）

3. 第一次责任链分发

   `InputStage`是处理输入的责任链，在调用`deliver`时会遍历责任链传递事件。

   事件分发完成后会调用`finishInputEvent`，告知`SystemServer`进程的`InputDispatcher`线程，最终将该事件移除，完成此次事件的分发消费。

4. 事件在Activity、Window、DecorView中传递

   经过层层回调，事件已经传递到了`DecorView`，也就是我们界面的根布局。

   事件分发经过了：==DecorView== -> Activity -> PhoneWindow -> ==DecorView==

   

#### 页面内部的分发

1. ViewGroup是否拦截事件
2. 子View是否拦截
3. 如果ViewGroup和子View都不拦截会如何
4. DOWN后续事件如何分发

#### 滑动冲突的解决

1. 外部拦截法
2. 内部拦截法

**View的滑动冲突**

在自定义 View 的过程经常会遇到**滑动冲突问题**，一般滑动冲突的类型有三种：

(1) 外部View和内部View滑动方向不一致；(2)==外部 View和内部View滑动方向一致==；(3)上述两种情况的嵌套。

要解决滑动冲突，就是利用**事件分发机制**，分为外部拦截法和内部拦截法：

**外部拦截法**的实现思路是事件首先是通过父容器的拦截处理，如果父容器不需要处理该事件的话，则不拦截，将事件传递给子View，如果父容器决定拦截，就在父容器的 onTouchEvent 方法里面直接处理该事件，这种方法符合事件分发机制。具体实现措施是**修改父容器的 onInterceptTouchEvent 方法**，在达到某一条件的时候，让这个方法直接返回 true 把事件拦截下来进而调用自己的 onTouchEvent 方法来处理，**需要注意的是如果想要让子 View 能够收到事件，需要在 onInterceptTouchEvent 方法里面判断如果是 DOWN 事件的话，返回 false，这样后续的 MOVE 以及 UP 事件才有机会传递到子 View 上面**，如果你直接在 onInterceptTouchEvent 方法里面 DOWN 情况下返回了 true，那么后续 的 MOVE 以及 UP 事件会直接由当前 View 的 onTouchEvent 处理了，这样的拦截将根本没有意义了，拦截只是在满足一定条件才会拦截，并不是所有情况下都拦截。

**内部拦截法**实现思路是事件从父容器传递到子View，父容器不做任何干预性措施，所有的事件都传递给子 View，如果子元素需要该事件，那么就由子元素消耗掉，该事件也就不会回传了，如果子元素不需要该事件，就回传给父容器处理；具体实现重写子View的**dispatchTouchEvent**方法，并且需要借助**requestDisallowInterceptTouchEvent** 方法，这个方法用来告诉父容器是否拦截当前事件，为了配合子 View 能够调用这个方法成功，**父容器必须默认能够拦截除了 DOWN 事件以外的事件**，因为如果一旦父容器拦截了 DOWN 事件，那么后续事件将不会再传递给子元素了，内部拦截法也就失去作用了，同时默认拦截MOVE和UP事件是为了当子View调用parent.requestDisallowInterceptTouchEvent(false)的时候父元素能继续拦截所需事件 。

### 源码分析：

```java
//Activity#dispatchTouchEvent
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {//将事件传递给Window进行分发
        return true;
    }
    return onTouchEvent(ev);
}
```

```java
//Window#superDispatchTouchEvent
public abstract boolean superDispatchTouchEvent(MotionEvent event);

//PhoneWindow#superDispatchTouchEvent
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    //PhoneWindow将事件直接传递给了DecorView
	return mDecor.superDispatchTouchEvent(event);
}
```

```java
//ViewGroup#dispatchTouchEvent
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            //actionDown是一系列事件的开头 需要重置所有状态
            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.
            final boolean intercepted;
            //如果是down事件 或者mFirstTouchTarget不为空
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                //如果子类并没有调用parent.requestDisallowInterceptTouchEvent(true) 那么这个值为false
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                //如果没有子view消费了事件 并且事件不是down事件 intercepted 为true
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

            // Update list of touch targets for pointer down, if needed.
            final boolean isMouseEvent = ev.getSource() == InputDevice.SOURCE_MOUSE;
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0
                    && !isMouseEvent;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
            //如果没有cancel而且viewGroup本身也没拦截事件 则进入这个代码块
            if (!canceled && !intercepted) {
                // If the event is targeting accessibility focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x =
                                isMouseEvent ? ev.getXCursorPosition() : ev.getX(actionIndex);
                        final float y =
                                isMouseEvent ? ev.getYCursorPosition() : ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            //两个判断有一个条件不满足就会遍历下一个child
                            //child.canReceivePointerEvents() 标识child是否能接收触摸事件 判断条件是可见并且没有执行动画
                            //!isTransformedTouchPointInView(x, y, child, null) 用来判断触摸事件的x,y 是否在child的矩形区域内
                            if (!child.canReceivePointerEvents()
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            //调用子view的dispatchTouchEvent 在view的dispatchTouchEvent里面调用了onTouchEvent()
                            //如果子view消费了事件 则dispatchTransformedTouchEvent()返回true
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                //赋值newTouchTarget
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                //将其设置为true 下面代码会用到
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // mFirstTouchTarget == null 代表没有子view消费事件
                //第三个参数出的是null 会直接调用到super.dispatchTouchEvent()即view的dispatchTouchEvent() 
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    //down事件的时候如果有子view消费事件了 alreadyDispatchedToNewTouchTarget值会为true
                    //并且target和newTouchTarget地址值是一样的
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {//其他事件（除了down）都会走这个代码块
                        //如果viewGroup拦截了事件 则会导致cancelChild为true 此时传递子view的event.action 是ACTION_CANCEL
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```

当事件由ViewGroup的子元素成功处理时，mFirstTouchTarget会被赋值并指向子元素，换种方式来说，当ViewGroup不拦截事件并将事件交由子元素处理时mFirstTouchTarget ! = null。

当ViewGroup不拦截事件的时候，事件会向下分发交由它的子View进行处理。首先遍历ViewGroup的所有子元素，然后判断子元素是否能够接收到点击事件。是否能够接收点击事件主要由两点来衡量：子元素是否在播动画和点击事件的坐标是否落在子元素的区域内。dispatchTransformedTouchEvent实际上调用的就是子元素的dispatchTouchEvent方法，如果子元素的dispatchTouchEvent返回true，这时我们暂时不用考虑事件在子元素内部是怎么分发的，那么mFirstTouchTarget就会被赋值同时跳出for循环。

```java
//View#dispatchTouchEvent
public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }
        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
```

首先会判断有没有设置OnTouchListener，如果OnTouchListener中的onTouch方法返回true，那么onTouchEvent就不会被调用。

```java
//View#onTouchEvent
        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags &
            PFLAG_PRESSED) ! = 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE ||
                    (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
        }
```



## **View的绘制**

View 视图绘制需要搞清楚两个问题，一个是从哪里开始绘制，一个是怎么绘制？

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，**View的三大流程都是通过ViewRoot来完成的**。在ActivityThread中，当Activity对象被创建以后，会把DecorView添加到Window当中，同时会创建ViewRootImpl对象，并且把ViewRootImpl对象和DecorView相关联。**View的绘制流程就是从ViewRoot的performTraversals方法开始的**。performTraversals方法会依次调用**performMeasure、performLayout、performDraw**三个方法，这个三个方法分别完成顶级View的测量、布局、绘制三个流程。其中performMeasure会调用measure方法，在measure方法中会调用onMeasure方法，在onMeasure方法中会对所有子元素进行measure过程，然后measure流程就从父容器传递到子元素中了，就完成了一次measure过程。接着子元素会重复父容器的measure过程，这样反复地就完成了整个View树的遍历。performLayout和performDraw的传递过程也是一样的，只是绘制流程地draw方法的传递是通过dispatchDraw来完成的。

所以视图绘制的三大流程依次是测量、布局、绘制。

在我们的 Activity 中调用了 setContentView 之后，会转而执行 PhoneWindow 的setContentView，在这个方法里面会判断我们存放内容的 ViewGroup(这个 ViewGroup 可以是 DecorView 也可以是 DecorView 的子 View)是否存在。不存在的 话则会创建一个DecorView 出来，并且会创建出相应的窗体风格，存在的话则会删除原先 ViewGroup 上面已有的 View，接着会调用 LayoutInflater 的 inflate 方法以 pull 解析的方式将当前布局文件中存在的 View 通过 addView 的方式添加到 ViewGroup 上面来，接着在 addView 方法里面就会执行我们常见的 invalidate 方法了，这个方法不只是在 View 视图绘制的过程中经常用到，其实动画的实现原理也是不断的调用这个方法来实现视图不断重绘的，执行这个方法的时候会调用他的父 View 的 invalidateChild 方法， 这个方法是属于 ViewParent 的，ViewGroup 以及 ViewRootImpl 中都对他进行了实现， invalidateChild 里 面主要做的事就是通过 do while 循环一层一层计算出当前 View 的四个点所对应的矩阵在 ViewRoot 中所对应的位置，那么有了这个矩阵的位置之后最终都会执行到 ViewRootImpl 的invalidateChildInParent 方法，执行这个方法的时候首先会检查当前线程是不是主线程，因为我们要开始准 备更新 UI 了，不是主线程的话是不允许更新 UI 的，接着就会执行scheduleTraversals 方法了，这个方法会通过 handler 来执行 doTraversal 方法，在这个方法里面就见到了我们平常所熟悉的 View 视图绘制的起点方法 performTraversals 了；

那么接下来就是真正的视图绘制流程了，大体上讲 View 的绘制经历了 Measure 测量、Layout 布局以及 Draw 绘制三个过程，具体来讲是从 ViewRootImpl 的 performTraversals方法开始，首先执行的将是 **performMeasure 方法**，这个方法里面会传入**两个 MeasureSpec类型的参数**，他在很大程度上决定了 View 的尺寸规格，对于 DecorView 来说宽高的MeasureSpec 值的获取与窗口尺寸以及自身的 LayoutParams 有关，对于**普通 View 来说其宽高的 MeasureSpec 值的获取由父容器的MeasureSpec以及自身的 LayoutParams 属性共同决定**，系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个MeasureSpec测量出View的宽高，在performMeasure 里面会执行 measure 方法，在 measure 方法里面会执行 onMeasure 方法，到这里 Measure 测量过程对 View 与 ViewGroup 来说是没有区别的，但是从 onMeasure 开始两者有差别了，因为 **View 本身已经不存在子 View 了，所以他 onMeasure 方法将执行setMeasuredDimension 方法，该方法会设置 View 的测量值**，但是对于 ViewGroup 来说，因为它里面还存在着子 View，那么我们就需要继续测量它里面的子 View 了，会对子View调用measureChild 方法，该方法内部又会对子View执行measure 方法，而 measure 方法转而又会执行onMeasure 方法，这样不断的递归进行下去，直到整个 View 树测量结束，这样performMeasure 方法执行结束了；接着便是执行 performLayout 方法了， performMeasure只是测量出 View 树中 View 的大小了，但是还不知道 View 的位置，所以也就出现了performLayout 方法了， performLayout 方法首先会执行**layout 方法，以确定 View 自身的位置**，如果当前 View 是 ViewGroup 的话，则会执行 onLayout 方法。在 onLayout 方法里面又会递归的执行 layout 方法，直到当前遍历到的 View 不再是 ViewGroup 为止，这样整个layout 布局过程就结束了，**View会在layout方法当中确定四个顶点的位置**，顶点的位置确定了，在父容器中的位置就确定了；在 View 树中 View 的大小以及位置都确定之后，接下来就是真正的绘制 View 显示在界面的过程了，该过程首先从performDraw 方法开始， performDraw方法 首先执行 draw 方法，在 draw 方法中首先绘制背景、接着调用 **onDraw 方法绘制自己**，如果当前 View 是 ViewGroup 的话，还要调用 dispatchDraw 方法绘制当前 ViewGroup 的子View，而 dispatchDraw 方法里面实际上是通过 drawChild 方法间接调用 draw 方法形成递归绘制整个 View 树，直到当前 View 不再是 ViewGroup 为止，这样整个 View 的绘制过程就结束了。







## **自定义View**

**自定义View的分类**

+ 继承View重写onDraw方法
+ 继承ViewGroup派生特殊的Layout
+ 继承特定的View例如TextView
+ 继承特定的ViewGroup例如LinearLayout



**自定义View的步骤**

+ 继承 View 或者 View 的子类
+ 在 res/values/attrs.xml 声明属性集合
+ 将自定义 View 放到布局文件中，需要完整全类名，命名空间不再是android，xmlns:custom="http://schemas.android.com/apk/res/[自定义 View 所在的包路径]" 
+ 在 xml 文件中设定指定属性值
+ 获取自定义属性，并且为 View 设置必要的事件
+ 进行自定义的绘制，实现 onDraw 方法，甚至 onMeasure，onLayout等等
+ 覆写 onTouch 等事件相关方法
+ 优化自定义的 View，比如为了减低刷新频率可以在 onDraw 中调用四个参数 的 invalidate 方法，因为**无参的 invalidate 方法刷新的是整个 View 树**，而**四个参数的 invalidate 方法刷新的是指定部分的 View**。等等。



**自定义属性**

+ 在res/values/attrs.xml文件自定义属性
+ 继承View
+ 在自定义View中获取属性集合
+ 给View添加设置属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--定义名字叫MyAttributeView属性集合-->
    <declare-styleable name="MyAttributeView">
        <!--定义一个名字叫my_name并且类型是string的属性-->
        <attr name="my_name" format="string"/>
        <attr name="my_age" format="integer"/>
        <attr name="my_bg" format="reference|color"/>
    </declare-styleable>
</resources>

<!--使用自定义View的时候设置自定义属性-->
<cy.review.customattributes.MyAttributeView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:my_age="100"
        app:my_bg="@drawable/jtx"
        app:my_name="android0220"/>
```

```java
public class MyAttributeView extends View {
    private int myAge;
    private String myName;
    private Bitmap myBg;

    public MyAttributeView(Context context, AttributeSet attrs) {
        super(context, attrs);

        //获得属性集合
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MyAttributeView);
        for (int i = 0; i < typedArray.getIndexCount(); i++) {
            int index = typedArray.getIndex(i);

            switch (index) {
                case R.styleable.MyAttributeView_my_age:
                    myAge = typedArray.getInt(index, 0);
                    break;
                case R.styleable.MyAttributeView_my_name:
                    myName = typedArray.getString(index);
                    break;
                case R.styleable.MyAttributeView_my_bg:
                    Drawable drawable = typedArray.getDrawable(index);
                    BitmapDrawable drawable1 = (BitmapDrawable) drawable;
                    myBg = drawable1.getBitmap();
                    break;
            }
        }
        // 记得回收
        typedArray.recycle();
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Paint paint = new Paint();
        canvas.drawText(myName + "---" + myAge, 50, 50, paint);
        canvas.drawBitmap(myBg, 50, 50, paint);
    }
}
```



# 当我们点击应用图标时，系统做了什么？

## 问题剖析和知识储备

点击桌面图标，其实就是触发了应用启动。

+ 冷启动

  从0开始启动一个应用的过程。需要启动应用进程，创建应用实例，最终启动一个Activity页面展示在用户面前。

  APP进程创建——Application实例创建——Activity实例创建与展示

+ 热启动

  将已经启动过的Activity重新启动，或者切到前台。

  既然已经启动过，说明进程和Application都是就绪状态，剩余的核心就是理解Activity任务栈了。

+ 温启动

  在进程ready的情况下，恢复Application和Activity的过程。

上述会涉及到**Zygote**和**AMS**相关内容，以及**Activity任务栈和生命周期**。

初次之外，作为应用层开发RD，涉及的技术思考：

+ 启动未注册的Activity（动态代码下发和插件化）
+ 启动速度优化的一些思路



## 冷启动流程

……

## 热启动流程

……





----

# 文章补充

+ [ACTION_CANCEL，ACTION_OUTSIDE 的触发条件与使用](https://juejin.cn/post/6844904154716962830)
+ [【带着问题学】Android事件分发8连问](https://mp.weixin.qq.com/s/YyJu3b_-InWjgyVXz48B4g)

