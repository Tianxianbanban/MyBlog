## Volley



**基本使用**

+ 添加依赖

+ 简单使用

  + StringRequest

    ```java
    class Util{
        private void stringRequest(){
            RequestQueue mQueue = Volley.newRequestQueue(getApplicationContext());
            StringRequest mStringRequest = new StringRequest(Request.Method.GET,
                    "http://www.baidu.com", new Response.Listener<String>() {
                @Override
                public void onResponse(String response) {
                    Log.e(TAG, "onResponse: " + response);
                }
            }, new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    Log.e(TAG, "onErrorResponse: ", error);
                }
            });
    
            //将请求添加到请求队列当中
            mQueue.add(mStringRequest);
        }
    }
    ```

  + JsonRequest

    ```java
    class Util{
        private void UseJsonRequest() {
            String requestBody = "ip=59.108.54.37";
            JsonObjectRequest mJsonObjectRequest = new JsonObjectRequest(
                    Request.Method.POST,
                    "http://ip.taobao.com/service/getIpInfo.php?ip=59.108.54.37",
                    null,
                    new Response.Listener<JSONObject>() {
                        @Override
                        public void onResponse(JSONObject response) {
                            IpModel ipModel = new Gson().fromJson(response.toString(), IpModel.class);
                            if (null != ipModel && null != ipModel.getData()) {
                                String city = ipModel.getData().getCity();
                                Log.e(TAG, city);
                            }
                        }
                    }, new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    Log.e(TAG, error.getMessage(), error);
                }
            }
            );
            mQueue.add(mJsonObjectRequest);
        }
    }
    ```

  + ImageRequest加载图片

    ```java
    class Util{
        private void UseImageRequest() {
            ////ImageRequest已经过时
            ImageRequest imageRequest = new ImageRequest(
                    "http://img.my.csdn.net/uploads/201603/26/1458988468_5804.jpg",
                    new Response.Listener<Bitmap>() {
                        @Override
                        public void onResponse(Bitmap response) {
                            //ImageView
                            iv_image.setImageBitmap(response);
                        }
                    }, 0, 0, Bitmap.Config.RGB_565,
                    new Response.ErrorListener() {
                        @Override
                        public void onErrorResponse(VolleyError error) {
                            iv_image.setImageResource(R.drawable.ico_default);
                        }
                    });
            //查看ImageRequest的源码发现，它可以设置你想要的图片的最大宽度和高度。
            // 在加载图片时，如果图片 超过期望的最大宽度和高度，则会进行压缩。
            mQueue.add(imageRequest);
        }
    }
    ```

  + ImageLoader加载图片

    ```java
    class Util{
        private void UseImageLoader() {
            ImageLoader imageLoader = new ImageLoader(mQueue, new BitmapCache());
            ImageLoader.ImageListener listener = ImageLoader.getImageListener(iv_image, R.drawable.ico_default, R.drawable.ico_default);
            imageLoader.get("http://img.my.csdn.net/uploads/201603/26/1458988468_5804.jpg", listener);
        }
    
    }
    ```

  + NetworkImageView加载图片

    ```java
    class Util{
        private void UseNetworkImageView() {
            ImageLoader imageLoader = new ImageLoader(mQueue, new BitmapCache());
            //NetworkImageView是一个自定义控件，继承自ImageView，
            // 其封装了请求网络加载图片的功能。
            nv_image.setDefaultImageResId(R.drawable.ico_default);
            nv_image.setErrorImageResId(R.drawable.ico_default);
            nv_image.setImageUrl("http://img.my.csdn.net/uploads/201603/26/1458988468_5804.jpg",
                    imageLoader);
            //NetworklmageView并没有提供设置最大宽度和高度的方法，
            //而是根据我们设置控件的宽和高，结合网络图片的宽和高，内部会自动实现压缩。
            //如果我们不想要压缩，也可以设置NetworkImageView控件的宽和高都为 wrap_content。
    
        }
    }
    ```

    

**源码梳理**

+ RequestQueue

  如果Android版本大于或等于2.3，则调用基于HttpURLConnection的HurlStack， 否则就调用基于HttpClient的HttpClientStack。

  接着创建RequestQueue，并调用它的start（）方法。

  在start（）方法中，创建CacheDispatcher**缓存调度线程**，并调用了它的start（）方法。然后在循环中（默认4次）创建NetworkDispatcher**网络调度线程**，调用它的 start（）方法。也就是默认开启了4个网络调度线程，加上1个缓存调度线程，就有5个线程在后台运行并等待请求的到来。

  然后我们的使用中还会继续创建Reques，并且添加进RequestQueue。

+ RequestQueue#add

  通过判断request.shouldCache（）来判断请求是否可以缓存，默认是可以缓存的。如果不能缓存，则将请求添加到**网络请求队列mNetworkQueue**中。如果能缓存，就判断之前是否有执行相同的请求且还没有返回结果的，如果有的话将此请求加入**mWaitingRequests队列**，不再重复请求；没有的话就将请求加入**缓存队列 mCacheQueue**。

  RequestQueue的add方法并没有请求网络或者对缓存进行操作。当将请求添加到网络请求队列或者缓存队列时，在**后台的网络调度线程和缓存调度线程轮询各自的请求队列，若发现有请求任务则开 始执行**。下面先看看缓存调度线程。 

+ CacheDispatcher**缓存调度线程**

  首先从缓存队列取出请求，判断请求是否被取消了，如果请求没有被取消则判断该请求是否有缓存的响应。如果有缓存的响应并且没有过期，则对缓存响应进行解析并回调给主线程；如果没有缓存的响应，则将请求加入mNetworkQueue网络请求队列。

+ NetworkDispatcher**网络调度线程**

  网络调度线程也是从队列中取出请求并且判断该请求是否被取消了。如果该请求没被取消，就去请求 网络得到响应并回调给主线程。

  请求网络时调用this.mNetwork.performRequest（request），这个 mNetwork 是一个接口，实现它的类是 BasicNetwork。

  请求网络后，会将响应结果存在**缓存**中，并调用this.mDelivery.postResponse（request，volleyError1）来回调给主线程。

+ BasicNetwork#performRequest

  调用**HttpStack**的performRequest方法请求网络，接下来根据不同的响应状态码来返回不同的NetworkResponse。另外HttpStack也是一个接口，实现它的两个类我们在前面已经提到了，就是 HurlStack和HttpClientStack。

+ Delivery#postResponse

  中间的执行流程中会调用Request 的 deliverResponse 方法。例如StringRequest就会在这里调用this.mListener.onResponse（response），最终将response回调给了 Response.Listener的onResponse（）方法。

+ 总结

  Volley分为三类线程，分别是主线程、缓存调度线程和网络调度线程，其中网络调 度线程默认开启4个。使用的时候，首先请求会加入缓存队列，缓存调度线程从缓存队列中取出请求。如果找到该请求的 缓存响应就直接读取缓存的响应并解析，然后回调给主线程；如果没有找到缓存的响应，则将这条请求加入网络请求队列，然后网络调度线程会轮询取出网络队列中的请求，取出后发送HTTP请求，解析响应并将响应存入缓存，并回调给主线程。

  适用于频繁的小数据网络请求，对于 post 大文件 的情况，显得有点无能为力了。



**设计模式**

​                                                                                                  



**文章**