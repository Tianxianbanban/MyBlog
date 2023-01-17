# Android动画

View动画、帧动画和属性动画，其实帧动画也属于View动画的一种，只不过它和平移、旋转等常见的View动画在表现形式上略有不同而已。View动画通过对场景里的对象不断做图像变换（平移、缩放、旋转、透明度）从而产生动画效果，它是一种渐近式动画，并且View动画支持自定义。帧动画通过顺序播放一系列图像从而产生动画效果，可以简单理解为图片切换动画，很显然，如果图片过多过大就会导致OOM。属性动画通过动态地改变对象的属性从而达到动画效果。



----

### [View动画](https://weread.qq.com/web/reader/9d932320716a2b159d9b881ka5b325d0225a5bfc9e0772d)

#### View动画种类

支持==平移、缩放、旋转和透明度==动画。既可以通过**XML**来定义（可读性更好，标签：<translate>、<scale>、<rotate>、<alpha>），也可以通过代码动态创建（**TranslateAnimation、ScaleAnimation、RotateAnimation和AlphaAnimation**）；另外，<set>标签表示动画集合，对应**AnimationSet**类。

**其他属性**

+ android:interpolator 插值器影响动画的速度。
+ android:shareInterpolator 表示集合中的动画是否和集合共享同一个插值器。如果集合不指定插值器，那么子动画就需要单独指定所需的插值器或者使用默认值。
+ android:duration——动画的持续时间
+ android:fillAfter——动画结束以后View是否停留在结束位置
+ 监听动画

```java
 public static interface AnimationListener {
     void onAnimationStart(Animation animation);
     void onAnimationEnd(Animation animation);
     void onAnimationRepeat(Animation animation);
 }
```

#### 自定义View动画

派生一种新动画只需要继承**Animation**这个抽象类，然后重写它的`initialize`和`applyTransformation`方法，在initialize方法中做一些初始化工作，在applyTransformation中进行相应的**==矩阵变换==**即可，很多时候需要采用Camera来简化矩阵变换的过程。而矩阵变换是数学上的概念。

#### View动画特殊使用场景

**LayoutAnimation**

为ViewGroup指定一个动画，这样当它的子元素出场时都会具有这种动画效果。

1. 定义LayoutAnimation

   + android:delay 表示子元素开始动画的时间延迟

   + android:animationOrder 表示子元素动画的顺序

   + android:animation 为子元素指定具体的入场动画

2. 为子元素指定具体的入场动画

3. 为ViewGroup指定android:layoutAnimation属性

**Activity的切换效果**

+ Activity有默认的切换效果，也可以自定义，主要用到`overridePendingTransition(int enterAnim, int exitAnim)`这个方法，必须在startActivity(Intent)或者finish()之后被调用才能生效，它的参数含义如下：(1) enterAnim——Activity被打开时，所需的动画资源id；·（2）exitAnim——Activity被暂停时，所需的动画资源id。

+ Fragment也可以添加切换动画，可以通过FragmentTransaction中的setCustomAnimations()方法来添加切换动画，这个切换动画需要是View动画。



----

### 帧动画

帧动画是顺序播放一组预先定义好的图片，类似于电影播放。系统提供**AnimationDrawable**来使用帧动画。使用比较简单，首先需要通过**XML**来定义一个AnimationDrawable，然后将这个Drawable**作为View的背景**并通过Drawable来播放动画即可。

帧动画的使用比较简单，但是比较**容易引起OOM**，所以在使用帧动画时应尽量避免使用过多尺寸较大的图片。



----

### 属性动画

它对作用对象进行了扩展，可以对任何对象做动画，甚至还可以没有对象。效果也得到了加强，不仅只能支持四种简单的变换。属性动画中有ValueAnimator、ObjectAnimator（继承自ValueAnimator）和AnimatorSet等概念，通过它们可以实现绚丽的动画。

**使用属性动画**

动画默认时间间隔300ms，默认帧率10ms/帧。其可以达到的效果是：在一个时间间隔内完成对象从一个属性值到另一个属性值的改变。因此，属性动画几乎是无所不能的，只要对象有这个属性，它都能实现动画效果。

属性动画可以通过代码实现以外，也可以通过XML来定义（定义在res/animator/目录下）。

+ 代码

  实际开发中==建议采用代码来实现属性动画==，这是因为通过代码来实现比较简单。更重要的是，很多时候一个属性的起始值是无法提前确定的，无法将属性动画定义在XML中。

  ```java
  ObjectAnimator.ofFloat(myObject, "translationY", -myObject.getHeight()).
  start();
  ```

+ XML

  <animator>标签只是比<objectAnimator>少了一个android:propertyName属性而已，其他都是一样的。

  + <set>标签的android:ordering属性有两个可选值：“together”和“sequentially”，其中“together”表示动画集合中的子动画同时播放，“sequentially”则表示动画集合中的子动画按照前后顺序依次播放。
  + android:propertyName——表示属性动画的作用对象的属性的名称；
  + android:duration——表示动画的时长；
  + android:valueFrom——表示属性的起始值；
  + android:valueTo——表示属性的结束值；
  + android:startOffset——表示动画的延迟时间，当动画开始后，需要延迟多少毫秒才会真正播放此动画；·
  + android:repeatCount——表示动画的重复次数；默认值为0，其中-1表示无限循环；
  + android:repeatMode——表示动画的重复模式；有两个选项：“repeat”和“reverse”，分别表示连续重复和逆向重复。
  + android:valueType——表示android:propertyName所指定的属性的类型，有“intType”和“floatType”两个可选项。另外，如果android:propertyName所指定的属性表示的是颜色，那么不需要指定android:valueType，系统会自动对颜色类型的属性做处理。

**插值器和估值器**

TimeInterpolator中文翻译为时间插值器，它的作用是**根据时间流逝的百分比来计算出当前属性值改变的百分比**，系统预置的有LinearInterpolator（线性插值器：匀速动画）、AccelerateDecelerateInterpolator（加速减速插值器：动画两头慢中间快）和Decelerate-Interpolator（减速插值器：动画越来越慢）等。

TypeEvaluator的中文翻译为类型估值算法，也叫估值器，它的作用是**根据当前属性改变的百分比来计算改变后的属性值**，系统预置的有IntEvaluator（针对整型属性）、FloatEvaluator（针对浮点型属性）和ArgbEvaluator（针对Color属性）。

插值器和估值算法除了系统提供的外，我们还可以自定义。

**属性动画监听器**

主要有如下两个接口：AnimatorUpdateListener和AnimatorListener。

+ AnimatorListener

  可以监听动画的开始、结束、取消以及重复播放。同时为了方便开发，系统还提供了AnimatorListener的适配器类**AnimatorListenerAdapter，这样我们就可以有选择地实现上面的4个方法了**，毕竟不是所有方法都是我们感兴趣的。

  ```java
  public static interface AnimatorListener {
    void onAnimationStart(Animator animation);
    void onAnimationEnd(Animator animation);
    void onAnimationCancel(Animator animation);
    void onAnimationRepeat(Animator animation);
  }
+ AnimatorUpdateListener

  它会监听整个动画过程，动画是由许多帧组成的，每播放一帧，onAnimationUpdate就会被调用一次，利用这个特性，我们可以做一些特殊的事情。

  ```java
  public static interface AnimatorUpdateListener {
      void onAnimationUpdate(ValueAnimator animation);
  }
  ```

**对任意属性做动画**

分析属性动画的原理：属性动画要求动画作用的对象提供该属性的get和set方法，属性动画根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用set方法，每次传递给set方法的值都不一样，确切来说是随着时间的推移，所传递的值越来越接近最终值。总结一下，我们对object的属性abc做动画，如果想让动画生效，要**同时满足**两个条件：

1. object必须要提供setAbc方法，如果动画的时候没有传递初始值，那么还要提供getAbc方法，因为系统要去取abc属性的初始值（如果这条不满足，程序直接Crash）。
2. object的setAbc对属性abc所做的改变必须能够通过某种方法反映出来，比如会带来UI的改变之类的（如果这条不满足，动画无效果但不会Crash）。

如果不满足上面条件，官方文档上告诉我们有3种解决方法：

+ 给你的对象加上get和set方法，如果你有权限的话；
+ 用一个类来包装原始对象，间接为其提供get和set方法；
+ 采用ValueAnimator，监听动画过程，自己实现属性的改变。

**属性动画的工作原理**

属性动画要求动画作用的对象提供该属性的set方法，属性动画根据你传递的该属性的初始值和最终值，以动画的效果多次去调用set方法。每次传递给set方法的值都不一样，确切来说是随着时间的推移，所传递的值越来越接近最终值。如果动画的时候没有传递初始值，那么还要提供get方法，因为系统要去获取属性的初始值。

源码分析：

```java
ObjectAnimator.ofInt(mButton, "width", 500).setDuration (5000).start()
```



```java
public void start() {
    // See if any of the current active/pending animators need to be canceled
    AnimationHandler handler = sAnimationHandler.get();
    if (handler ! = null) {
        int numAnims = handler.mAnimations.size();
        for (int i = numAnims -1; i >= 0; i--) {
            if (handler.mAnimations.get(i) instanceof ObjectAnimator) {
                ObjectAnimator anim = (ObjectAnimator) handler.mAnimations.
                get(i);
                if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                    anim.cancel();
                }
            }
        }
        numAnims = handler.mPendingAnimations.size();
        for (int i = numAnims -1; i >= 0; i--) {
            if (handler.mPendingAnimations.get(i) instanceof ObjectAnimator) {
                ObjectAnimator anim = (ObjectAnimator) handler.mPending-
                Animations.get(i);
                if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                    anim.cancel();
                }
            }
        }
        numAnims = handler.mDelayedAnims.size();
        for (int i = numAnims -1; i >= 0; i--) {
            if (handler.mDelayedAnims.get(i) instanceof ObjectAnimator) {
                ObjectAnimator anim = (ObjectAnimator) handler.mDelayed-
                        Anims.get(i);
                        if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                            anim.cancel();
                        }
                    }
                }
            }
            if (DBG) {
                Log.d(LOG_TAG, "Anim target, duration: " + getTarget() + ", " +
                getDuration());
                for (int i = 0; i < mValues.length; ++i) {
                    PropertyValuesHolder pvh = mValues[i];
                    Log.d(LOG_TAG, "    Values[" + i + "]: " +
                        pvh.getPropertyName() + ", " + pvh.mKeyframes.getValue(0)
                        + ", " +
                        pvh.mKeyframes.getValue(1));
                }
            }
            super.start();
        }
```

做的事情很简单，首先会判断如果当前动画、等待的动画（Pending）和延迟的动画（Delay）中有和当前动画相同的动画，那么就把相同的动画给取消掉，接下来那一段是log，再接着就调用了父类的super.start()方法。因为ObjectAnimator继承了ValueAnimator。

ValueAnimator的Start方法：

```java
private void start(boolean playBackwards) {
    if (Looper.myLooper() == null) {
        throw new AndroidRuntimeException("Animators may only be run on
        Looper threads");
    }
    mPlayingBackwards = playBackwards;
    mCurrentIteration = 0;
    mPlayingState = STOPPED;
    mStarted = true;
    mStartedDelay = false;
    mPaused = false;
    updateScaledDuration(); // in case the scale factor has changed since
    creation time
    AnimationHandler animationHandler = getOrCreateAnimationHandler();
      animationHandler.mPendingAnimations.add(this);
      if (mStartDelay == 0) {
          // This sets the initial value of the animation, prior to actually
          starting it running
          setCurrentPlayTime(0);
          mPlayingState = STOPPED;
          mRunning = true;
          notifyStartListeners();
      }
      animationHandler.start();
  }
```

可以看出属性动画需要**运行在有Looper的线程中**。上述代码最终会调用Animation-Handler的start方法，这个AnimationHandler并不是Handler，它是一个**Runnable**。看一下它的代码，通过代码我们发现，很快就调到了JNI层，不过JNI层最终还是要调回来的。它的run方法会被调用，这个Runnable涉及和底层的交互，可以忽略这部分，看重点：ValueAnimator中的doAnimationFrame方法：

```java
final boolean doAnimationFrame(long frameTime) {
    if (mPlayingState == STOPPED) {
        mPlayingState = RUNNING;
        if (mSeekTime < 0) {
            mStartTime = frameTime;
        } else {
            mStartTime = frameTime - mSeekTime;
            // Now that we're playing, reset the seek time
            mSeekTime = -1;
        }
    }
    if (mPaused) {
        if (mPauseTime < 0) {
            mPauseTime = frameTime;
        }
        return false;
    } else if (mResumed) {
        mResumed = false;
        if (mPauseTime > 0) {
            // Offset by the duration that the animation was paused
            mStartTime += (frameTime - mPauseTime);
            }
        }
    // The frame time might be before the start time during the first frame of
    // an animation.  The "current time" must always be on or after the start
    // time to avoid animating frames at negative time intervals.  In practice, this
    // is very rare and only happens when seeking backwards.
        final long currentTime = Math.max(frameTime, mStartTime);
        return animationFrame(currentTime);
    }
```

末尾调用了animationFrame方法，而animationFrame内部调用了animateValue：

```java
void animateValue(float fraction) {
    fraction = mInterpolator.getInterpolation(fraction);
    mCurrentFraction = fraction;
    int numValues = mValues.length;
    for (int i = 0; i < numValues; ++i) {
        mValues[i].calculateValue(fraction);
    }
    if (mUpdateListeners ! = null) {
        int numListeners = mUpdateListeners.size();
        for (int i = 0; i < numListeners; ++i) {
            mUpdateListeners.get(i).onAnimationUpdate(this);
        }
    }
}
```

上述代码中的calculateValue方法就是**计算每帧动画所对应的属性的值**。哪里调用属性的get和set方法的：在初始化的时候，如果属性的初始值没有提供，则get方法将会被调用，看Property-ValuesHolder的setupValue方法，可以发现get方法是通过反射来调用的，如下：

```java
private void setupValue(Object target, Keyframe kf) {
    if (mProperty ! = null) {
        Object value = convertBack(mProperty.get(target));
        kf.setValue(value);
    }
    try {
        if (mGetter == null) {
                  Class targetClass = target.getClass();
                  setupGetter(targetClass);
                  if (mGetter == null) {
                      // Already logged the error - just return to avoid NPE
                      return;
                  }
              }
              Object value = convertBack(mGetter.invoke(target));
              kf.setValue(value);
          } catch (InvocationTargetException e) {
              Log.e("PropertyValuesHolder", e.toString());
          } catch (IllegalAccessException e) {
              Log.e("PropertyValuesHolder", e.toString());
          }
      }
```

当动画的下一帧到来的时候，PropertyValuesHolder中的setAnimatedValue方法会将新的属性值设置给对象，调用其set方法。从下面的源码可以看出，set方法也是通过反射来调用的：

```java
void setAnimatedValue(Object target) {
    if (mProperty ! = null) {
        mProperty.set(target, getAnimatedValue());
    }
    if (mSetter ! = null) {
        try {
            mTmpValueArray[0] = getAnimatedValue();
            mSetter.invoke(target, mTmpValueArray);
        } catch (InvocationTargetException e) {
            Log.e("PropertyValuesHolder", e.toString());
        } catch (IllegalAccessException e) {
            Log.e("PropertyValuesHolder", e.toString());
        }
    }
}
```



### 动画使用注意事项

+ OOM问题

  主要出现在帧动画中，当图片数量较多且图片较大时就极易出现OOM，这个在实际的开发中要尤其注意，尽量避免使用帧动画。

+ 内存泄露

  在属性动画中有一类无限循环的动画，这类动画需要在Activity退出时及时停止，否则将导致Activity无法释放从而造成内存泄露，通过验证后发现View动画并不存在此问题。

+ View动画问题

  View动画是对View的影像做动画，并不是真正地改变View的状态，因此有时候会出现动画完成后View无法隐藏的现象，即setVisibility(View.GONE)失效了，这个时候只要调用view.clearAnimation()清除View动画即可解决此问题。

+ 不要使用px

  进行动画的过程中，要尽量使用dp，使用px会导致在不同的设备上有不同的效果。

+ 动画元素的交互

  从3.0开始，属性动画的单击事件触发位置为移动后的位置，但是View动画仍然在原位置。

+ 硬件加速

  使用动画的过程中，建议开启硬件加速，这样会提高动画的流畅性。

