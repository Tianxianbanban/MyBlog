## 仿ViewPager



1. 继承ViewGroup

2. 添加子view

   ```java
   		myViewPager = findViewById(R.id.myviewpager);
   
           //添加页面
           for (int i = 0; i < ids.length; i++) {
               ImageView imageView = new ImageView(this);
               imageView.setBackgroundResource(ids[i]);
   
               //添加进MyViewPager当中
               myViewPager.addView(imageView);
           }
   ```

3. 如果省略measure的过程，就直接在layout过程中设置具体宽度高度数值。

   重写onLayout方法，遍历子View调用layout方法，给子View设置四个顶点的位置。

   ```java
   public class MyViewPager extends ViewGroup {
       //省略。。。
       
       @Override
       protected void onLayout(boolean changed, int l, int t, int r, int b) {
           Log.e("TAG", "onLayout: ");
           //遍历孩子，给每个孩子指定在屏幕的坐标位置
           for (int i = 0; i < getChildCount(); i++) {
               View childView = getChildAt(i);
               childView.layout(i*getWidth(),0,(i+1)*getWidth(),getHeight());
           }
       }
   }
   ```

4. 使MyViewpager能够滑动，切换不同的子View到屏幕中显示

   定义一个手势监听器GestureDetector，在MyViewpager的构造方法中初始化，然后在MyViewpager的onTouchEvent方法中将事件传递给GestureDetector。(使用手势识别器是因为它已经实现了一些触摸事件的回调了，比如长按、滑动、双击等等)

   ```java
   GestureDetector detector;
   detector = new GestureDetector(context,new GestureDetector.SimpleOnGestureListener(){
               @Override
               public void onLongPress(MotionEvent e) {
                   super.onLongPress(e);
                   Toast.makeText(context,"长按",Toast.LENGTH_SHORT).show();
               }
   
               @Override
               public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
                   //scrollBy与scrollTo都只改变View的内容
                   scrollBy((int)distanceX,0);
                   return true;
               }
   
               @Override
               public boolean onDoubleTap(MotionEvent e) {
                   Toast.makeText(context,"双击",Toast.LENGTH_SHORT).show();
                   return super.onDoubleTap(e);
               }
           });
   ```

5. 具体的滑动可以使用View的scrollBy或者scrollTo方法。滑动的参数就是手势监听器onScroll回调当中的参数。

6. 上面的步骤是可以让MyViewpager触摸滑动了，可是如果触摸停止了，MyViewpager的滑动也就停止了，效果还是不够的。所以要让手指滑动停止的时候，MyViewpager能够自定将子View滑动到屏幕中间。

   那么就要在onTouchEvent方法当中进一步处理MotionEvent事件，在分别处理DOWN、MOVE、UP的时候分别做一些处理。因为要让滑动停止的时候MyViewpager也能自动滑动到合适的位置，然后触摸停止的位置也不确定，所以就要进行一个判断，判断停止的位置偏左还是偏右，如果偏左就让子View往左滑动，偏右就让子View往右滑动。所以DOWN事件的时候可以记录下事件发生的坐标；滑动的过程因为设置了手势识别器，可以不用管了；UP事件发生的时候就需要对这次触摸事件的停止给出一个结果，得出UP事件发生的位置，然后通过DOWN事件和UP事件的位置计算出这次是构成左滑还是右滑，还是都不构成，就让子View复位。当然还要注意一下边界值的判断。

   ```java
    @Override
       public boolean onTouchEvent(MotionEvent event) {
           super.onTouchEvent(event);
           //3.把事件传递给手势识别器
           detector.onTouchEvent(event);
           switch (event.getAction()) {
               case MotionEvent.ACTION_DOWN:
                   startX = event.getX();
                   break;
               case MotionEvent.ACTION_MOVE:
                   break;
               case MotionEvent.ACTION_UP:
                   float endX = event.getX();
                   int tempIndex = currentIndex;
                   //左滑
                   if ((startX - endX) > getWidth()/2) {
                       //显示下一个页面
                       tempIndex ++;
                   } else if ((endX - startX) > getWidth()/2) {
                       //右滑，显示下一个界面
                       tempIndex--;
                   }
                   //根据下标位置移动到指定的页面
                   scrollToPager(tempIndex);
                   break;
           }
           return true;
       }
   
   	/**
        * 根据下标位置移动到指定的页面
        * 滑动过程限定边界，屏蔽非法值
        */
       private void scrollToPager(int tempIndex) {
           if(tempIndex < 0){
               tempIndex = 0;
           }
   
           if(tempIndex > getChildCount()-1){
               tempIndex = getChildCount()-1;
           }
   
           //当前页面的下标位置
           currentIndex = tempIndex;
   		scrollTo(currentIndex*getWidth(),0);
       }
   ```

7. 上面是用scrollTo进行的滑动，体验就是，松手的那一刻，滑动的很突然，很奇怪。

   所以最后这个滑动可以考虑下使用弹性滑动。可以使用Scroller。

   Scroller本身并不会帮助滑动，只是会记录当前传递进去的一些参数，包括滑动的起始点、滑动的距离、甚至也就是滑动的持续时间。要让View滑动，还是要依靠invalidate方法，因为这个方法内部也会调用onDraw()方法导致View的重绘，进而调用computeScroll方法，才能实现弹性滑动。就是：当View重绘后会在draw方法中调用computeScroll方法，这个方法中会去向Scroller获取当前scrollX和scrollY；然后通过scrollTo方法实现滑动；然后调用postInvalidate来进行下一次重绘，这一次的重绘过程和第一次一样，也会调用computeScroll方法，然后继续向Scroller获取当前的scrollX和scrollY，这样循环反复知道绘制完成。computeScrollOffset会根据时间流逝的百分比，计算出当前的scrollX和scrollY作为滑动的目标位置。

   ```java
   Scroller scroller;
   /**
        * 根据下标位置移动到指定的页面
        * 滑动过程限定边界，屏蔽非法值
        */
       private void scrollToPager(int tempIndex) {
           if(tempIndex < 0){
               tempIndex = 0;
           }
   
           if(tempIndex > getChildCount()-1){
               tempIndex = getChildCount()-1;
           }
   
           //当前页面的下标位置
           currentIndex = tempIndex;
           //但是这样子的滑动比较生硬，可以用Scroller替代
   //        scrollTo(currentIndex*getWidth(),0);
           int distanceX = currentIndex*getWidth() - getScrollX();
           scroller.startScroll(getScrollX(),getScrollY(),distanceX,0);
           invalidate();//会调用onDraw();computeScroll();
       }
   
   	//重写View的computeScroll方法
       @Override
       public void computeScroll() {
   //        super.computeScroll();
           if(scroller.computeScrollOffset()){
               float currX = scroller.getCurrX();
   
               scrollTo((int) currX,0);
   //            invalidate();
               postInvalidate();
           }
       }
   ```

8. 如果还想使子View滑动的时候能够切换不同的微小点图标，可以使用RadioGroup，向里面添加RadioButton。

   ```java
   	for(int i=0;i<myviewpager.getChildCount();i++){
               RadioButton button = new RadioButton(this);
               button.setId(i);//0~5的id
   
               if(i==0){
                   button.setChecked(true);
               }
   
               //添加到RadioGroup
               rg_main.addView(button);
           }
   ```

9. 点击RadioButton切换到不同的页面

   ```java
   		//设置RadioGroup选中状态的变化
           rg_main.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
               /**
                *
                * @param group
                * @param checkedId : 0~5之间
                */
               @Override
               public void onCheckedChanged(RadioGroup group, int checkedId) {
   
                   myviewpager.scrollToPager(checkedId);//根据下标位置定位到具体的某个页面
               }
           });
   ```

10. 滑动页面能够切换RadioButton

    那也需要给MyViewpager设置一个监听，能够在滑动的时候调用这个设置的监听器的回调方法。



11. 在其中一个页面添加入纵向滑动的页面以后，除了在onInterceptTouchEvent方法对于MOVE事件产生时解决滑动冲突以外，出现了一个新的问题，就是在纵向滑动列表完了如果立即横向滑动，会出现闪动的情况，一滑动后，手指原来触摸的视图的位置会在滑动的一瞬间出现跳跃到另一个位置的情况，就是滑动似乎没没有跟随手指的位置合适变化，而是出现了一个跳跃性的闪动。

    针对这个问题，我观察了onInterceptTouchEvent方法和onTouchEvent方法里面根据DOWN、MOVE、UP事件产生的日志，发现其实最开始DOWN事件在onInterceptTouchEvent方法是已经得到了处理的，然后在MOVE事件当中也能够根据滑动的方向，选择是否拦截事件。如果拦截事件也就调用onTouchEvent方法，然后在onTouchEvent方法当中去使用GestureDetector处理滑动横向滑动；如果不拦截事件那么就是返回false，然后事件向子VIew分发，子View会处理纵向滑动的。所以经过观察的话，这个地方之所以出现这种情况，就是因为损失了MOVE事件，因为GestureDetector是只在onTouchEvent调用了，但是在这整个事件传递的过程中，只有MOVE事件和UP事件会传递到onTouchEvent，也就是GestureDetector只能处理到这两个事件的话，GestureDetector当中计算出的distance一下子偏大了，所以产生突然滑动很大一块距离，跟手指触摸实际的情况差距很大的问题。所以的话从onInterceptTouchEvent方法开始，就让GestureDetector能够处理事件的话，就能比较细致的处理事件，不会造成这么大误差了。



**关于GestureDetector**

[Android手势检测——GestureDetector全面分析](https://blog.csdn.net/totond/article/details/77881180)