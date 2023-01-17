## Bitmap



**Bitmap**

Bitmap在Android中指的是一张图片，可以是png格式的也可以是jpg等其他常见图片格式。



**Bitmap加载的方法**

BitmapFactory类提供了四类方法：

+ decodeFile（文件系统）
+ decodeResource（资源）
+ decodeStream（输入流）decodeFile和decodeResource间接调用decodeStream
+ decodeByteArray（字节数组）

这几个方法在底层对应了几个native方法。



**Bitmap的高效加载**

高效加载需要采用BitmapFactory.Options按照一定的**采用率**来加载缩小后的图片，这可以在一定程度上降低内存占用避免OOM，提高了Bitmap加载时的性能。BitmapFactory的四类方法都是支持BitmapFactory.Options参数的，这样就能比较方便地进行图片的采样缩放。

通过BitmapFactory.Options缩放图片，主要是用到了它的**inSampleSize**参数，也就是**采样率**。当inSampleSize为1的时候，采用后的图片是原始大小；而当inSampleSize大于1，比如为n，采样后的图片宽高均为图片原始大小的1/n，那么像素数就是原图的1/(n^2)，占用的内存大小也是原图的1/(n^2)；采用率小于1时无缩放效果。另外，inSampleSize的取值应该总是2的指数，那么系统会向下取整并且选择一个最接近的2的指数来代替，但是这个结论并非所有Android版本上都成立，所以当做开发建议即可。**通常图片最终大小如果大于ImageView是可以的，但是如果小于的话会导致图片被拉伸而模糊**。

获取采样率的流程：

1. 将BitmapFactory.Options的inJustDecodeBounds参数设置为true并加载图片。当这个参数为true的时候只会**解析图片的原始宽高信息**，而不会真正地加载图片，所以这个操作是轻量级的。
2. 从BitmapFactory.Options中**取出图片的原始宽高信息**，它们对应于outWidth和outHeight参数。
3. 根据采样率的规则，结合目标View的所需大小**计算出采样率inSampleSize**。
4. 将BitmapFactory.Options的inJustDecodeBounds参数设置为false，然后**重新加载图片**。

经过这4个步骤，加载出的图片就是最终缩放后的图片，当然也可能不需要缩放。需要注意的是，这个时候BitmapFactory获取的图片宽高信息和图片位置以及程序运行的设备有关，比如同一张图片放在不同的drawable目录或者程序运行在不同屏幕密度的设备上，都可能导致BitmapFactory获取到不同的结果。这与Android的资源加载机制有关。

```java

public class ImageResizer {
    private static final String TAG = "ImageResizer";

    public ImageResizer() {
    }

    public Bitmap decodeSampledBitmapFromResource(Resources res,
            int resId, int reqWidth, int reqHeight) {
        // First decode with inJustDecodeBounds=true to check dimensions
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeResource(res, resId, options);//解析出图片原始宽高信息

        // Calculate inSampleSize
        options.inSampleSize = calculateInSampleSize(options, reqWidth,
                reqHeight);

        //真正地去加载图片
        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(res, resId, options);
        //除了BitmapFactory.decodeResource方法以外，
        //其他三个decode系列的方法也是支持采样率加载的，处理方式类似
    }

    public Bitmap decodeSampledBitmapFromFileDescriptor(FileDescriptor fd, int reqWidth, int reqHeight) {
        // First decode with inJustDecodeBounds=true to check dimensions
        final BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFileDescriptor(fd, null, options);

        // Calculate inSampleSize
        options.inSampleSize = calculateInSampleSize(options, reqWidth,
                reqHeight);

        // Decode bitmap with inSampleSize set
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeFileDescriptor(fd, null, options);
    }

    public int calculateInSampleSize(BitmapFactory.Options options,
            int reqWidth, int reqHeight) {
        if (reqWidth == 0 || reqHeight == 0) {
            return 1;
        }

        //取出图片原始宽高信息
        // Raw height and width of image
        final int height = options.outHeight;
        final int width = options.outWidth;
        Log.d(TAG, "origin, w= " + width + " h=" + height);
        int inSampleSize = 1;

        //结合处目标View的大小，计算出采样率
        if (height > reqHeight || width > reqWidth) {
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;

            // Calculate the largest inSampleSize value that is a power of 2 and
            // keeps both
            // height and width larger than the requested height and width.
            while ((halfHeight / inSampleSize) >= reqHeight
                    && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }

        Log.d(TAG, "sampleSize:" + inSampleSize);
        return inSampleSize;
    }
}

```





**Bitmap优化**

+ 尽量使用较小的图片：使用较小的图片，可以减少 OOM，同时也减少 InflateException 异常，因为我们可能在刚刚加载图片的时候就可能因为内存不足而加载失败，这时报的是InflateException 异常。

+ 减少 Bitmap 占用的内存空间：进行适当的**图片压缩**以及选用合适的**图片解码**器。具体来讲的话，在我们将图片加载到内存之前，可以通过设置加载图片的BitmapFactory.Options中inJustDecodeBounds 属性为 true，这样的话**只会将图片的头部信息加载到内存**，我们可以得到图片的宽度和高度；然后**根据显示图片的View的宽度和高度计算出一定的压缩比**，并设置到BitmapFactory.Options的 inSampleSize 属性，也就是采样率。接着设置加载图片的inJustDecodeBounds 属性为false，这时候再次加载图片的时候，加载到内存中的就是压缩过后的图像了。

  以及**采用不同的图片颜色格式，占用的内存空间也是不同的**，对于 ARGB_8888 编码的图片，其每个像素占用 4 个字节，对于其他的图片编码方式，每个像素占用的字节数也是有所不同的。

+ 使用**缓存机制**减少 Bitmap 占用内存的大小：
  可以使用 LRUCache 内存缓存和 DiskLruCache 来减少 Bitmap 占用的内存空间大小。
  
+ 使用 inBitmap 高级属性
  Bitmap 的 inBitmap 高级属性会大大提升 Android 系统在分配和释放 Bitmap 占用空间上的效率，具体来讲的话，对于每一个想要加载的 Bitmap， **inBitmap 属性会告知图片解码器优先使用已经在内存中的 Bitmap 区域，而不是重新开辟一段内存空间**，但是要想真正使用好 inBitmap 高级属性的话，需要具备两个条件，我们新申请的 Bitmap 大小必须小于等于旧 Bitmap 的大小，此外新旧 Bitmap 的图片解码器类型也应该一致，要使用 ARGB_8888就都是 ARGB_8888，不过，为了优化，我们可以设置一个可重用 Bitmap 的对象池，这样后续的 Bitmap 到来之后都可以从池子里面找到一个适合自己的模板去复用了。



可参考文章：

[掘金Bitmap内存占用](https://juejin.im/post/599e698ff265da246b3c2d7d#heading-4)

[Bitmap内存优化](https://www.jianshu.com/p/3f6f6e4f1c88)

[Bitmap优化详谈](https://juejin.im/post/5bfbd5406fb9a049be5d2a20#heading-17)