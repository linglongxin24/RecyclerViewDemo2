#精通RecyclerView：打造ListView、GridView、瀑布流；学会添加分割线、 添加删除动画 、Item点击事件

>在上一篇[Android用RecyclerView练手仿美团分类界面](http://blog.csdn.net/linglongxin24/article/details/52997986)写了RecyclerView的基本用法，
今天想想，在这里重新学习一下RecyclerView的完整用法。包括如何打造一个普通的ListView和横向的ListView、普通的GridView和横向的GridView、如何添加分割线、
还有就是添加和删除的动画、以及如何设置Item的OnClick事件监听。这里包含了各种用法，也算是一个入门的用法，已经掌握的高手请略过。

 先看总体效果图
 
![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/effect.gif)

#一.RecyclerView介绍

RecyclerView 是Android L版本中新添加的一个用来取代ListView和GridView的SDK，它的灵活性与可替代性比ListView更好。
接下来通过一系列的内容讲解如何使用RecyclerView,彻底抛弃ListView和GridView。

#二.RecyclerView架构

![架构](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/introduce.png)

#三.RecyclerView的优点

 * 1.RecyclerView封装了ViewHolder的回收复用，也就是说RecyclerView标准化了ViewHolder，
 编写Adapter面向的是ViewHolder而不再是View了，复用的   逻辑被封装了，写起来更加简单。
 
 * 2.提供了一种插拔式的体验，高度的解耦，异常的灵活，针对一个Item的显示RecyclerView专门抽取出了相应的类，
 来控制Item的显示，使其的扩展性非常强。例如：你想控制横向或者纵向滑动列表效果可以通过LinearLayoutManager这个类来进行控制
 (与GridView效果对应的是GridLayoutManager,与瀑布流对应的还有StaggeredGridLayoutManager等)，
 也就是说RecyclerView不再拘泥于ListView的线性展示方式，它也可以实现GridView的效果等多种效果。你想控制Item的分隔线，
 可以通过继承RecyclerView的ItemDecoration这个类，然后针对自己的业务需求去抒写代码。
 
 * 3.可以控制Item增删的动画，可以通过ItemAnimator这个类进行控制，当然针对增删的动画，RecyclerView有其自己默认的实现。
 
#四.RecyclerView实现三部曲

 *  第一步：设置布局管理器
 
 用来确定每一个item如何进行排列摆放:     
  *  LinearLayoutManager 相当于ListView的效果   
  *  GridLayoutManager相当于GridView的效果      
  *  StaggeredGridLayoutManager 瀑布流
 
 *  第二步：添加分割线
 
 我们用到了网上流传的万能分割线DividerItemDecoration和DividerGridItemDecoration，首先在style.xml里面定义分割线图片：
  *  现在drawable中新建divider.xml
  ```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <solid android:color="@color/colorAccent"/>
    <size android:height="1dp"
        android:width="1dp"/>
</shape>
```
  *   在style.xml中设置android:listDivider

```xml
        <item name="android:listDivider">@drawable/divider</item>
```

  *  第三步：设置适配器
 
```java
package cn.bluemobi.dylan.recyclerviewdemo2;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

/**
 * RecyclerView的适配器
 * Created by yuandl on 2016-11-01.
 */

public class RvAdapter extends RecyclerView.Adapter<RvAdapter.MyViewHolder> {
    private Context context;
    private List<Integer> datas;

    /**
     * item的点击事件的长按事件接口
     */
    private OnItemClickListener onItemClickListener;
    /**
     * 瀑布流时的item随机高度
     */
    private List<Integer> heights = new ArrayList<>();

    /**
     * 不同的类型设置item不同的高度
     *
     * @param type
     */

    private int type = 0;

    public RvAdapter(Context context, List<Integer> datas) {
        this.context = context;
        this.datas = datas;
        for (int i : datas) {
            int height = (int) (Math.random() * 100 + 300);
            heights.add(height);
        }
    }

    public void setType(int type) {
        this.type = type;
    }

    /**
     * 设置点击事件
     *
     * @param onItemClickListener
     */
    public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
        this.onItemClickListener = onItemClickListener;
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View contentView = LayoutInflater.from(context).inflate(R.layout.item, parent, false);
        MyViewHolder viewHolder = new MyViewHolder(contentView);
        return viewHolder;
    }

    @Override
    public void onBindViewHolder(MyViewHolder holder, final int position) {
        RecyclerView.LayoutParams layoutParams;
        if (type == 0) {
            layoutParams = new RecyclerView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        } else if (type == 1) {
            layoutParams = new RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        } else {
            layoutParams = new RecyclerView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, heights.get(position));
            layoutParams.setMargins(2, 2, 2, 2);
        }
        holder.itemView.setLayoutParams(layoutParams);
        holder.imageView.setImageResource(datas.get(position));
        holder.tv.setText("分类" + position);
        /**设置item点击监听**/
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (onItemClickListener != null) {
                    onItemClickListener.onItemClickListener(position, datas.get(position));
                }
            }
        });

    }

    @Override
    public int getItemCount() {
        return datas == null ? 0 : datas.size();
    }

    /**
     * 用于缓存的ViewHolder
     */
    public class MyViewHolder extends RecyclerView.ViewHolder {
        private ImageView imageView;
        public TextView tv;

        public MyViewHolder(View itemView) {
            super(itemView);
            imageView = (ImageView) itemView.findViewById(R.id.iv);
            tv = (TextView) itemView.findViewById(R.id.tv);
        }
    }

    /**
     * 设置item监听的接口
     */
    public interface OnItemClickListener {
        void onItemClickListener(int position, Integer data);

    }
}

```
#五.打造各种效果

 * 1.竖向的ListView

```java
    rv.setBackgroundColor(Color.TRANSPARENT);
    rv.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
    rvAdapter.setType(0);
    rv.removeItemDecoration(itemDecoration);
    rv.setLayoutParams(new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.MATCH_PARENT));
    itemDecoration = new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST);
    rv.addItemDecoration(itemDecoration);
```

![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/listview.png)

 * 2.横向的ListView

```java
   rvAdapter.setType(1);
   rv.removeItemDecoration(itemDecoration);
   rv.setLayoutParams(new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.WRAP_CONTENT));
   rv.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false));
   itemDecoration = new DividerItemDecoration(this, DividerItemDecoration.HORIZONTAL_LIST);
   rv.addItemDecoration(itemDecoration);
```

![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/horizontalListView.png)
 
 
 * 3.竖向的GridView

```java
   rvAdapter.setType(1);
   rv.setBackgroundColor(Color.TRANSPARENT);
   rv.removeItemDecoration(itemDecoration);
   rv.setLayoutParams(new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.MATCH_PARENT));
   rv.setLayoutManager(new GridLayoutManager(this, 5));
   itemDecoration = new DividerGridItemDecoration(this);
   rv.addItemDecoration(itemDecoration);
```

![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/gridview.png)
  
 * 4.横向的GridView

```java
    rvAdapter.setType(1);
    rv.setBackgroundColor(Color.TRANSPARENT);
    rv.setLayoutParams(new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.WRAP_CONTENT));
    rv.removeItemDecoration(itemDecoration);
    rv.setLayoutManager(new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.HORIZONTAL));
    itemDecoration = new DividerGridItemDecoration(this);
    rv.addItemDecoration(itemDecoration);
```

![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/horizontalGridView.png)
  
 * 5.竖向的瀑布流

```java
     rvAdapter.setType(3);
     rv.setBackgroundColor(getResources().getColor(R.color.colorAccent));
     rv.setLayoutParams(new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.MATCH_PARENT));
     rv.removeItemDecoration(itemDecoration);
     rv.setLayoutManager(new StaggeredGridLayoutManager(5, StaggeredGridLayoutManager.VERTICAL));
```

![效果图](https://github.com/linglongxin24/RecyclerViewDemo2/blob/master/images/staggeredgridview.png)

#六.[GiHub](https://github.com/linglongxin24/RecyclerViewDemo2)