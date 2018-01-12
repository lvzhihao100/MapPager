github:https://github.com/lvzhihao100/MapPager   
简书:https://www.jianshu.com/p/3e1fd35b8d2c
##### 效果图
![使用ViewPager 自定义PageTransformer](http://upload-images.jianshu.io/upload_images/4179767-a0d022a473d91503.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 实现方式 
+ ###### 使用ViewPager 自定义PageTransformer
```java
viewPager.setPageTransformer(false, new MapTransformer());
```
###### MapTransformer
```java
public class MapTransformer implements ViewPager.PageTransformer {
    //卡片之间的高度差
    private static final float LENTH = 100;

    @Override
    public void transformPage(View page, float position) {
        page.setTranslationY(LENTH * Math.abs(position));
    }
}
```
##### 效果图
![使用RecyclerView 设置PagerSnapHelper实现](http://upload-images.jianshu.io/upload_images/4179767-72f07c9731fd4cac.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 实现方式 
+ ###### 使用RecyclerView 设置PagerSnapHelper实现
##### PagerMapSnapHelper
```
PagerMapSnapHelper linearSnapHelper = new PagerMapSnapHelper();
        linearSnapHelper.attachToRecyclerView(recyclerView);
```
```java
public class PagerMapSnapHelper extends PagerSnapHelper {
    private OnPosChange onPosChange;
    private int curPos;

    public void setOnPosChange(OnPosChange onPosChange) {
        this.onPosChange = onPosChange;
    }

    @Override
    public void attachToRecyclerView(@Nullable RecyclerView recyclerView) throws IllegalStateException {
        super.attachToRecyclerView(recyclerView);
        recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                int childCount = recyclerView.getChildCount();
                for (int i = 0; i < childCount; i++) {
                    View childAt = recyclerView.getChildAt(i);

                    int left = childAt.getLeft();
                    if (left <= childAt.getWidth()) {
                        childAt.setTranslationY((childAt.getWidth() - left) * 200 / childAt.getWidth());
                    } else if (left >= childAt.getWidth() && left <= childAt.getWidth() * 2) {
                        childAt.setTranslationY((left - childAt.getWidth()) * 200 / childAt.getWidth());
                    } else {
                        childAt.setTranslationY(200);
                    }
                }
            }

            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == SCROLL_STATE_IDLE) {
                    if (onPosChange != null) {
                        int childCount = recyclerView.getChildCount();
                        int minLenth = WindowUtil.csw;
                        int centerPos;
                        for (int i = 0; i < childCount; i++) {
                            View childAt = recyclerView.getChildAt(i);

                            int left = childAt.getLeft();
                            int right = childAt.getRight();
                            if (left <= WindowUtil.csw / 2 && right >= WindowUtil.csw / 2) {
                                centerPos = recyclerView.getChildAdapterPosition(childAt);
                                changePos(centerPos);
                                break;
                            } else {
                                if (right < WindowUtil.csw / 2) {
                                    minLenth = WindowUtil.csw / 2 - right;
                                } else if (left > WindowUtil.csw / 2) {
                                    if (left - WindowUtil.csw / 2 < minLenth) {
                                        centerPos = recyclerView.getChildAdapterPosition(childAt);
                                        changePos(centerPos);
                                        break;
                                    } else {
                                        centerPos = recyclerView.getChildAdapterPosition(recyclerView.getChildAt(i - 1));
                                        changePos(centerPos);
                                        break;
                                    }
                                }
                            }

                        }
                    }
                }
            }
        });
    }

    private void changePos(int centerPos) {
        if (curPos != centerPos) {
            curPos=centerPos;
            onPosChange.change(centerPos);
        }
    }

    public interface OnPosChange {
        void change(int curPos);
    }
}
```
##### 效果图
![recyclertop.gif](http://upload-images.jianshu.io/upload_images/4179767-a1a801117e768451.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 实现方式 
+ ###### 使用RecyclerView 设置LayoutManager实现
```java
        recyclerView.setLayoutManager(new VegaLayoutManager());
```
###### VegaLayoutManager
```java
public class VegaLayoutManager extends RecyclerView.LayoutManager {

    private int scroll = 0;
    private SparseArray<Rect> locationRects = new SparseArray<>();
    private SparseBooleanArray attachedItems = new SparseBooleanArray();
    private ArrayMap<Integer, Integer> viewTypeHeightMap = new ArrayMap<>();

    private boolean needSnap = false;
    private int lastDy = 0;
    private int maxScroll = -1;
    private RecyclerView.Adapter adapter;
    private RecyclerView.Recycler recycler;

    public VegaLayoutManager() {
        setAutoMeasureEnabled(true);
    }

    @Override
    public RecyclerView.LayoutParams generateDefaultLayoutParams() {
        return new RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
    }

    @Override
    public void onAdapterChanged(RecyclerView.Adapter oldAdapter, RecyclerView.Adapter newAdapter) {
        super.onAdapterChanged(oldAdapter, newAdapter);
        this.adapter = newAdapter;
    }

    @Override
    public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
        this.recycler = recycler; // 二话不说，先把recycler保存了
        if (state.isPreLayout()) {
            return;
        }

        buildLocationRects();

        // 先回收放到缓存，后面会再次统一layout
        detachAndScrapAttachedViews(recycler);
        layoutItemsOnCreate(recycler);
    }

    private void buildLocationRects() {
        locationRects.clear();
        attachedItems.clear();

        int tempPosition = getPaddingTop();
        int itemCount = getItemCount();
        for (int i = 0; i < itemCount; i++) {
            // 1. 先计算出itemWidth和itemHeight
            int viewType = adapter.getItemViewType(i);
            int itemHeight;
            if (viewTypeHeightMap.containsKey(viewType)) {
                itemHeight = viewTypeHeightMap.get(viewType);
            } else {
                View itemView = recycler.getViewForPosition(i);
                addView(itemView);
                measureChildWithMargins(itemView, View.MeasureSpec.UNSPECIFIED, View.MeasureSpec.UNSPECIFIED);
                itemHeight = getDecoratedMeasuredHeight(itemView);
                viewTypeHeightMap.put(viewType, itemHeight);
            }

            // 2. 组装Rect并保存
            Rect rect = new Rect();
            rect.left = getPaddingLeft();
            rect.top = tempPosition;
            rect.right = getWidth() - getPaddingRight();
            rect.bottom = rect.top + itemHeight;
            locationRects.put(i, rect);
            attachedItems.put(i, false);
            tempPosition = tempPosition + itemHeight;
        }

        if (itemCount == 0) {
            maxScroll = 0;
        } else {
            computeMaxScroll();
        }
    }

    /**
     * 对外提供接口，找到第一个可视view的index
     */
    public int findFirstVisibleItemPosition() {
        int count = locationRects.size();
        Rect displayRect = new Rect(0, scroll, getWidth(), getHeight() + scroll);
        for (int i = 0; i < count; i++) {
            if (Rect.intersects(displayRect, locationRects.get(i)) &&
                    attachedItems.get(i)) {
                return i;
            }
        }
        return 0;
    }

    /**
     * 计算可滑动的最大值
     */
    private void computeMaxScroll() {
        maxScroll = locationRects.get(locationRects.size() - 1).bottom - getHeight();
        if (maxScroll < 0) {
            maxScroll = 0;
            return;
        }

        int itemCount = getItemCount();
        int screenFilledHeight = 0;
        for (int i = itemCount - 1; i >= 0; i--) {
            Rect rect = locationRects.get(i);
            screenFilledHeight = screenFilledHeight + (rect.bottom - rect.top);
            if (screenFilledHeight > getHeight()) {
                int extraSnapHeight = getHeight() - (screenFilledHeight - (rect.bottom - rect.top));
                maxScroll = maxScroll + extraSnapHeight;
                break;
            }
        }
    }

    /**
     * 初始化的时候，layout子View
     */
    private void layoutItemsOnCreate(RecyclerView.Recycler recycler) {
        int itemCount = getItemCount();
        Rect displayRect = new Rect(0, scroll, getWidth(), getHeight() + scroll);
        for (int i = 0; i < itemCount; i++) {
            Rect thisRect = locationRects.get(i);
            if (Rect.intersects(displayRect, thisRect)) {
                View childView = recycler.getViewForPosition(i);
                addView(childView);
                measureChildWithMargins(childView, View.MeasureSpec.UNSPECIFIED, View.MeasureSpec.UNSPECIFIED);
                layoutItem(childView, locationRects.get(i));
                attachedItems.put(i, true);
                childView.setPivotY(0);
                childView.setPivotX(childView.getMeasuredWidth() / 2);
                if (thisRect.top - scroll > getHeight()) {
                    break;
                }
            }
        }
    }


    /**
     * 初始化的时候，layout子View
     */
    private void layoutItemsOnScroll() {
        int childCount = getChildCount();
        // 1. 已经在屏幕上显示的child
        int itemCount = getItemCount();
        Rect displayRect = new Rect(0, scroll, getWidth(), getHeight() + scroll);
        int firstVisiblePosition = -1;
        int lastVisiblePosition = -1;
        for (int i = childCount - 1; i >= 0; i--) {
            View child = getChildAt(i);
            if (child == null) {
                continue;
            }
            int position = getPosition(child);
            if (!Rect.intersects(displayRect, locationRects.get(position))) {
                // 回收滑出屏幕的View
                removeAndRecycleView(child, recycler);
                attachedItems.put(position, false);
            } else {
                // Item还在显示区域内，更新滑动后Item的位置
                if (lastVisiblePosition < 0) {
                    lastVisiblePosition = position;
                }

                if (firstVisiblePosition < 0) {
                    firstVisiblePosition = position;
                } else {
                    firstVisiblePosition = Math.min(firstVisiblePosition, position);
                }

                layoutItem(child, locationRects.get(position)); //更新Item位置
            }
        }

        // 2. 复用View处理
        if (firstVisiblePosition > 0) {
            // 往前搜索复用
            for (int i = firstVisiblePosition - 1; i >= 0; i--) {
                if (Rect.intersects(displayRect, locationRects.get(i)) &&
                        !attachedItems.get(i)) {
                    reuseItemOnSroll(i, true);
                } else {
                    break;
                }
            }
        }
        // 往后搜索复用
        for (int i = lastVisiblePosition + 1; i < itemCount; i++) {
            if (Rect.intersects(displayRect, locationRects.get(i)) &&
                    !attachedItems.get(i)) {
                reuseItemOnSroll(i, false);
            } else {
                break;
            }
        }
    }

    /**
     * 复用position对应的View
     */
    private void reuseItemOnSroll(int position, boolean addViewFromTop) {
        View scrap = recycler.getViewForPosition(position);
        measureChildWithMargins(scrap, 0, 0);
        scrap.setPivotY(0);
        scrap.setPivotX(scrap.getMeasuredWidth() / 2);

        if (addViewFromTop) {
            addView(scrap, 0);
        } else {
            addView(scrap);
        }
        // 将这个Item布局出来
        layoutItem(scrap, locationRects.get(position));
        attachedItems.put(position, true);
    }


    private void layoutItem(View child, Rect rect) {
        int topDistance = scroll - rect.top;
        int layoutTop, layoutBottom;
        int itemHeight = rect.bottom - rect.top;
        if (topDistance < itemHeight && topDistance > 0) {
            float rate1 = (float) topDistance / itemHeight;
            float rate2 = 1 - rate1 * rate1 / 3;
            float rate3 = 1 - rate1 * rate1;
            child.setScaleX(rate2);
            child.setScaleY(rate2);
            child.setAlpha(rate3);
            layoutTop = 0;
            layoutBottom = itemHeight;
        } else {
            child.setScaleX(1);
            child.setScaleY(1);
            child.setAlpha(1);

            layoutTop = rect.top - scroll;
            layoutBottom = rect.bottom - scroll;
        }
        int realtop = rect.top - scroll;
        int height = getHeight() >> 1;
        realtop = Math.abs(realtop - height);
        int width = getWidth() >> 2;

        layoutDecorated(child, realtop * width / height - width + rect.left, layoutTop, realtop * width / height - width + rect.right, layoutBottom);
    }

    @Override
    public boolean canScrollVertically() {
        return true;
    }

    @Override
    public int scrollVerticallyBy(int dy, RecyclerView.Recycler recycler, RecyclerView.State state) {
        if (getItemCount() == 0 || dy == 0) {
            return 0;
        }
        int travel = dy;
        if (dy + scroll < 0) {
            travel = -scroll;
        } else if (dy + scroll > maxScroll) {
            travel = maxScroll - scroll;
        }
        scroll += travel; //累计偏移量
        lastDy = dy;
        if (!state.isPreLayout() && getChildCount() > 0) {
            layoutItemsOnScroll();
        }

        return travel;
    }

    @Override
    public void onAttachedToWindow(RecyclerView view) {
        super.onAttachedToWindow(view);
        new StartSnapHelper().attachToRecyclerView(view);
    }

    @Override
    public void onScrollStateChanged(int state) {
        if (state == RecyclerView.SCROLL_STATE_DRAGGING) {
            needSnap = true;
        }
        super.onScrollStateChanged(state);
    }

    public int getSnapHeight() {
        if (!needSnap) {
            return 0;
        }
        needSnap = false;

        Rect displayRect = new Rect(0, scroll, getWidth(), getHeight() + scroll);
        int itemCount = getItemCount();
        for (int i = 0; i < itemCount; i++) {
            Rect itemRect = locationRects.get(i);
            if (displayRect.intersect(itemRect)) {

                if (lastDy > 0) {
                    // scroll变大，属于列表往下走，往下找下一个为snapView
                    if (i < itemCount - 1) {
                        Rect nextRect = locationRects.get(i + 1);
                        return nextRect.top - displayRect.top;
                    }
                }
                return itemRect.top - displayRect.top;
            }
        }
        return 0;
    }

    public View findSnapView() {
        if (getChildCount() > 0) {
            return getChildAt(0);
        }
        return null;
    }
}
```


