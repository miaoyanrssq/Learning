# RecyclerView.ItemDecoration实现标题悬浮列表

> 我们知道 RecyclerView 给提供了一个RecyclerView.ItemDecoration 来给我们使用，这可是好东西呀。RecyclerView.ItemDecoration 从字面上来看是用来给 RecyclerView 里面每个 item 添加装饰用的(当然也可以给RecyclerView的整体添加装饰)。例如，你可以通过 RecyclerView.ItemDecoration 来给 RecyclerView 的每个 item 添加分割线、给每个 item 添加 padding 等等。这里我们通过 RecyclerView.ItemDecoration 来实现 RecyclerView 分组悬浮列表的功能。

 我们先简单的看下RecyclerView.ItemDecoration里面几个函数：
 
 ```
 /**
     * 可以通过重写这个函数给RecyclerView绘制任意合适的decorations(装饰)
     * 会在RecyclerView item绘制之前绘制。可以认为是绘制在RecyclerView的下面
     * 会在RecyclerView类的onDraw()里面调用
     */
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
        onDraw(c, parent);
    }

    /**
     * deprecated掉的函数我们不管，忽视掉，不建议使用了
     */
    @Deprecated
    public void onDraw(Canvas c, RecyclerView parent) {
    }

    /**
     * 可以通过重写这个函数给RecyclerView绘制任意合适的decorations(装饰)
     * 会在RecyclerView item绘制之后绘制。可以认为是绘制在RecyclerView的上面(在上面在盖一层)
     * 会在RecyclerView类的super.draw()之后调用,
     */
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        onDrawOver(c, parent);
    }

    /**
     * deprecated掉的函数不建议使用了，忽视掉
     */
    @Deprecated
    public void onDrawOver(Canvas c, RecyclerView parent) {
    }


    /**
     * deprecated掉的函数不建议使用了，忽视掉
     */
    @Deprecated
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
        outRect.set(0, 0, 0, 0);
    }

    /**
     * 给RecyclerView　item对应的每个view增加一些offsets(你可以这么认为item对应的view外面还有一层布局，给这个布局增加padding)
     */
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        getItemOffsets(outRect, ((RecyclerView.LayoutParams) view.getLayoutParams()).getViewLayoutPosition(), parent);
    }


 ```
 
 看到 ItemDecoration 提供到我们的就三个函数了：onDraw()、onDrawOver()、getItemOffsets()。getItemOffsets()函数会在 RecyclerView 里面每个子 view 测量的时候调用，可以用来给每个子 view 添加offset(间距)。onDraw()会在RecyclerView的onDraw()方法里面调用。onDrawOver()函数会在RecyclerView的draw()函数里面调用。关于onDraw()、onDrawOver()两个函数的区分咱们可以简单的认为onDraw()是在RecyclerView绘制内容的时候调用。onDrawOver()是在RecyclerView绘制完内容之后再调用，相当于可以在RecyclerView之上在绘制一层内容。


通过对 RecyclerView.ItemDecoration 类的简单分析，再结合我们分组固定标题 View 的需求，我们是要把每个分组的标题 View 固定在顶部，恩，那肯定是在要绘制在RecyclerView层之上的吧，和RecyclerView.ItemDecoration里面的onDrawOver()函数正好对应上了

代码：

```
package cn.zgy.news.view;

import android.graphics.Canvas;
import android.graphics.Rect;
import android.support.v7.widget.RecyclerView;
import android.view.View;

import org.w3c.dom.Text;

import java.util.Collections;

import cn.zgy.multitype.ItemViewBinder;
import cn.zgy.multitype.MultiTypeAdapter;
import cn.zgy.news.bean.TextBean;
import cn.zgy.news.binder.TextBinder;

/**
* 实现recycleview分组悬浮列表
* @author zhengy
* create at 2018/9/12 上午10:44
**/
public class PinnedHeaderItemDecoration extends RecyclerView.ItemDecoration{

    private Rect mPinnedHeaderRect     = null;
    private int  mPinnedHeaderPosition = -1;

    /**
     * 把固定的view绘制在上层
     * @param c
     * @param parent
     * @param state
     */
    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        super.onDrawOver(c, parent, state);
        if(parent.getChildCount() > 0){
            MultiTypeAdapter adapter = (MultiTypeAdapter) parent.getAdapter();
            //找到要固定的pin view
            View firstView = parent.getChildAt(0);
            int firstAdapterPosition = parent.getChildAdapterPosition(firstView);
            int  pinnedHeaderPosition = getBinderIndex(firstAdapterPosition, adapter);
            mPinnedHeaderPosition = pinnedHeaderPosition;
            if(pinnedHeaderPosition != -1){
                //获取viewHolder
                TextBinder.TextHolder pinnedHeaderViewHolder = (TextBinder.TextHolder) adapter.onCreateViewHolder(parent,
                        adapter.getItemViewType(pinnedHeaderPosition));
                //更新pinnedHeaderViewHolder中的内容
                TextBinder binder = (TextBinder) adapter.getTypePool().getItemViewBinder(adapter.getItemViewType(pinnedHeaderPosition));
                binder.onBindViewHolder(pinnedHeaderViewHolder, (TextBean) adapter.getItems().get(pinnedHeaderPosition));
                //要固定的view
                View pinnedHeaderView = pinnedHeaderViewHolder.itemView;
                ensurePinnedHeaderViewLayout(pinnedHeaderView, parent);
                int sectionPinOffset = 0;
                for(int index=0 ; index < parent.getChildCount() ; index++){
                    if(adapter.getTypePool().getItemViewBinder(adapter.getItemViewType(index)) instanceof TextBinder){
                        View sectionView = parent.getChildAt(index);
                        int sectionTop = sectionView.getTop();
                        int pinViewHeight = pinnedHeaderView.getHeight();
                        if(sectionTop < pinViewHeight && sectionTop > 0){
                            sectionPinOffset = sectionTop - pinViewHeight;
                        }
                    }
                }

                int saveCount = c.save();
                RecyclerView.LayoutParams layoutParams = (RecyclerView.LayoutParams) pinnedHeaderView.getLayoutParams();
                if (layoutParams == null) {
                    throw new NullPointerException("PinnedHeaderItemDecoration");
                }
                c.translate(layoutParams.leftMargin, sectionPinOffset);
                c.clipRect(0, 0, parent.getWidth(), pinnedHeaderView.getMeasuredHeight());
                pinnedHeaderView.draw(c);
                c.restoreToCount(saveCount);
                if (mPinnedHeaderRect == null) {
                    mPinnedHeaderRect = new Rect();
                }
                mPinnedHeaderRect.set(0, 0, parent.getWidth(), pinnedHeaderView.getMeasuredHeight() + sectionPinOffset);


            }else {
                mPinnedHeaderRect = null;
            }

        }
    }


    private int getBinderIndex(int firstAdapterPosition, MultiTypeAdapter adapter){
        for(int index = firstAdapterPosition ; index >=0 ; index--){
            if(adapter.getTypePool().getItemViewBinder(adapter.getItemViewType(index)) instanceof TextBinder){

                return index;
            }
        }
        return -1;
    }


    private void ensurePinnedHeaderViewLayout(View pinView, RecyclerView recyclerView) {
        if (pinView.isLayoutRequested()) {
            /**
             * 用的是RecyclerView的宽度测量，和RecyclerView的宽度一样
             */
            RecyclerView.LayoutParams layoutParams = (RecyclerView.LayoutParams) pinView.getLayoutParams();
            if (layoutParams == null) {
                throw new NullPointerException("PinnedHeaderItemDecoration");
            }
            int widthSpec = View.MeasureSpec.makeMeasureSpec(
                    recyclerView.getMeasuredWidth() - layoutParams.leftMargin - layoutParams.rightMargin, View.MeasureSpec.EXACTLY);

            int heightSpec;
            if (layoutParams.height > 0) {
                heightSpec = View.MeasureSpec.makeMeasureSpec(layoutParams.height, View.MeasureSpec.EXACTLY);
            } else {
                heightSpec = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED);
            }
            pinView.measure(widthSpec, heightSpec);
            pinView.layout(0, 0, pinView.getMeasuredWidth(), pinView.getMeasuredHeight());
        }
    }
}

```

本例是结合MultiType实现的，思路在文中已经说清楚了，如果需要示例：  
[传送门](https://github.com/miaoyanrssq/DevProject.git)， 这是我搭建的一个代码框架，目前还在完善中

