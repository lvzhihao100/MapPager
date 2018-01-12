github:https://github.com/lvzhihao100/MapPager   
简书:https://www.jianshu.com/p/3e1fd35b8d2c
***
##### 效果图
![使用ViewPager 自定义PageTransformer](http://upload-images.jianshu.io/upload_images/4179767-a0d022a473d91503.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 实现方式 
+ ###### 使用ViewPager 自定义PageTransformer
```java
viewPager.setPageTransformer(false, new MapTransformer());
```

源码地址:[MapTransformer](https://github.com/lvzhihao100/MapPager/blob/master/MapTransformer.java)
***
##### 效果图
![使用RecyclerView 设置PagerSnapHelper实现](http://upload-images.jianshu.io/upload_images/4179767-72f07c9731fd4cac.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 实现方式 
+ ###### 使用RecyclerView 设置PagerSnapHelper实现
源码地址:[PagerMapSnapHelper](https://github.com/lvzhihao100/MapPager/blob/master/PagerMapSnapHelper.java)
```
PagerMapSnapHelper linearSnapHelper = new PagerMapSnapHelper();
        linearSnapHelper.attachToRecyclerView(recyclerView);
```
***
##### 效果图
![recyclertop.gif](http://upload-images.jianshu.io/upload_images/4179767-a1a801117e768451.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 实现方式 
+ ###### 使用RecyclerView 设置LayoutManager实现
```java
        recyclerView.setLayoutManager(new VegaLayoutManager());
```
源码地址:[VegaLayoutManager](https://github.com/lvzhihao100/MapPager/blob/master/VegaLayoutManager.java)
