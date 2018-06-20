# ViewController-复杂界面开发

> ViewController 是一种界面开发组件化的实现方式,利用它可以将一些复杂的 UI 界面开发组件化, 从而更好的组织代码结构,从而提高开发效率,降低维护成本。

> 不同于 Fragment ，ViewController 更加小巧、灵活、易控制，代码也超级简单，目前只有一个不到 100 行的类。


在日常开发中，当我们看到如下的 UI 原型时，在实际编码实现过程中，通常我们会把所有的布局先用 xml 实现，然后在 Activity 中编写所有的 UI 操作逻辑。

![] (image/vc1.png)

这里，稍微思考下，这种方式的开发会有什么弊端。

所有的 UI 元素都需要定义在同一个类中，所有的 UI 逻辑操作也都发生在同一个类里，在加上一些业务逻辑，这个界面的代码会在不断迭代的过程中越来越难保持 clean .

但是使用 ViewController 这种思想后，你可以把页面逻辑按 UI 块做拆分，进行组件化开发，最后在 Activity 中只需要把组件进行组合即可。

下面是一张 ViewController 示意图，如下所示。

![] (image/vc2.png)

如上图所示，现在把界面拆分为了四个 ViewController，每个ViewController 管理自己的一组View 集合，这样就巧妙的把原本需要写在 Activity 代码分发到了不同的 ViewController 中去。

##优点
* 界面开发组件化，解决 Activity/Fragment 中 UI 代码臃肿问题。
* 灵活的 UI 开发，同一组件可在多处复用，从而做到代码重用。
* 易维护，开发简单。


##具体实现

ViewController.class

```

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;


/**
 * core of view controler
 * Created by GuDong on 7/13/16 22:42.
 * Contact with gudong.name@gmail.com.
 *
 * @param <T> data type for this ViewController, a view controler should map a data type
 */
public abstract class ViewController<T> {
    private T mData;
    private Context mContext;
    private View mView;

    public ViewController(Context context) {
        this.mContext = context;
    }

    public void attachRoot(ViewGroup root) {
        initView(root);
        root.addView(mView);
        onCreatedView(mView);
    }

    public void attachRoot(ViewGroup root, int index) {
        initView(root);
        root.addView(mView, index);
        onCreatedView(mView);
    }

    public void attachRoot(ViewGroup root, int width, int height) {
        initView(root);
        root.addView(mView, width, height);
        onCreatedView(mView);
    }

    public void attachRoot(ViewGroup root, ViewGroup.LayoutParams params) {
        initView(root);
        root.addView(mView, params);
        onCreatedView(mView);
    }


    public void attachRoot(ViewGroup root, int index, ViewGroup.LayoutParams params) {
        initView(root);
        root.addView(mView, index, params);
        onCreatedView(mView);
    }

    private void initView(ViewGroup root) {
        int resLayoutId = resLayoutId();
        if (resLayoutId <= 0) {
            throw new IllegalStateException("Please check your layout id in resLayoutId() method");
        }
        if (mView != null) {
            throw new IllegalStateException("a viewController can't attachRoot twice");
        }
        mView = LayoutInflater.from(mContext).inflate(resLayoutId, root, false);
    }

    public void fillData(T data) {
        this.mData = data;
        if (mData != null) {
            onBindView(data);
        }
    }

    public void detachedRoot() {
        onDestoryView(mView);
    }

    /**
     * indicate layout id of this ViewControl
     *
     * @return layout id
     */
    protected abstract int resLayoutId();

    /**
     * view has been created
     *
     * @param view real view
     */
    protected abstract void onCreatedView(View view);

    /**
     * bind data to view
     *
     * @param data data
     */
    protected abstract void onBindView(T data);

    /**
     * view has been Destory
     *
     * @param view
     */
    protected void onDestoryView(View view) {

    }

    public Context getContext() {
        return mContext;
    }

    public View getView() {
        return mView;
    }

    public T getData() {
        return mData;
    }
}

```

HouseParamViewController.class

```

public class HouseParamViewController extends ViewController<HouseDetail.Param> {
    @Bind(R.id.tv_param_one)
    TextView mTvParamOne;
    @Bind(R.id.tv_param_two)
    TextView mTvParamTwo;
    @Bind(R.id.tv_param_three)
    TextView mTvParamThree;
    @Bind(R.id.tv_param_four)
    TextView mTvParamFour;
    @Bind(R.id.tv_param_five)
    TextView mTvParamFive;
    @Bind(R.id.tv_param_six)
    TextView mTvParamSix;

    public HouseParamViewController(Context context) {
        super(context);
    }

    @Override
    protected int resLayoutId() {
        return R.layout.detail_house_param;
    }

    @Override
    protected void onCreatedView(View view) {
        ButterKnife.bind(this,view);
    }

    @Override
    protected void onBindView(HouseDetail.Param data) {
        mTvParamTwo.setText(data.city+"\n城市");
        mTvParamOne.setText(data.address+"\n地址");
        mTvParamThree.setText(data.owner+"\n业主");
        mTvParamFour.setText(data.area+"M"+"\n面积");
        mTvParamFive.setText(formatTime(data.createTime)+"\n发布时间");
        mTvParamSix.setText("(空)");
    }

    private String formatTime(long time){
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.format(new Date(time));
    }

}

```

MainActivity.class

```
public class MainActivity extends AppCompatActivity {


    @Bind(R.id.ll_container)
    LinearLayout mLlContainer;
    @Bind(R.id.pb)
    ProgressBar mPb;
    @Bind(R.id.root_layout)
    FrameLayout mRootLayout;

    private ViewController<List<String>> mHousePhotoViewController;
    private ViewController<HouseDetail.Param> mHouseParamViewControler;

    private ViewController<HouseDetail.Param> mHouseParamTopViewControler;
    private ViewController<HouseDetail.Param> mHouseParamBottomViewControler;

    private ViewController<List<String>> mHouseCommentViewControler;
    private ViewController<String> mHouseDescViewControler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        ButterKnife.bind(this);

        mHousePhotoViewController = new HousePhotoViewController(this);
        mHouseParamViewControler = new HouseParamViewController(this);


        mHouseParamTopViewControler = new HouseParamViewController(this);
        mHouseParamTopViewControler.attachRoot(mRootLayout, Utils.dp2px(200), Utils.dp2px(50));


        mHouseParamBottomViewControler = new HouseParamViewController(this);
        FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT, Gravity.BOTTOM | Gravity.CENTER_HORIZONTAL);
        mHouseParamBottomViewControler.attachRoot(mRootLayout, layoutParams);


        mHouseDescViewControler = new HouseDescViewController(this);
        mHouseCommentViewControler = new HouseCommentViewController(this);

        mHousePhotoViewController.attachRoot(mLlContainer);
        mHouseParamViewControler.attachRoot(mLlContainer);
        mHouseDescViewControler.attachRoot(mLlContainer);
        mHouseCommentViewControler.attachRoot(mLlContainer);

        getData();
    }

    private void fillData(HouseDetail detail) {
        mHousePhotoViewController.fillData(detail.photos);
        mHouseParamViewControler.fillData(detail.param);

        mHouseParamTopViewControler.fillData(detail.param);
        mHouseParamBottomViewControler.fillData(detail.param);


        mHouseDescViewControler.fillData(detail.desc);
        mHouseCommentViewControler.fillData(detail.comments);
    }

    /**
     * mock fetch data
     */
    private void getData() {
        mPb.setVisibility(View.VISIBLE);
        mLlContainer.setVisibility(View.GONE);
        mLlContainer.postDelayed(new Runnable() {
            @Override
            public void run() {
                mPb.setVisibility(View.GONE);
                mLlContainer.setVisibility(View.VISIBLE);

                HouseDetail detail = mockHouseData();
                fillData(detail);
            }
        }, 1000);
    }

    private HouseDetail mockHouseData() {
        HouseDetail detail = new HouseDetail();

        List<String> photos = new ArrayList<>();
        photos.add("http://cdn.mse.mlwplus.com/avenger/mlw_default_18.jpg@540w.jpg");
        photos.add("http://cdn.mse.mlwplus.com/avenger/mlw_default_02.jpg@540w.jpg");

        List<String> comments = new ArrayList<>();
        comments.add("GuDong: 房子不错,但是太远了,上班很不方便啊,每天折腾3小时上班,小身板折腾不起啊。");
        comments.add("Jack: a nice house,i like, but the price is too ...");
        comments.add("Rose: good source");
        comments.add("MJ: just so so,just so so,just so so,just so so,");


        HouseDetail.Param param = new HouseDetail.Param();
        param.address = "ChaoYang";
        param.area = 80;
        param.city = "Beijing";
        param.createTime = System.currentTimeMillis();
        param.owner = "Mr Mao";


        detail.photos = photos;
        detail.comments = comments;
        detail.param = param;
        detail.desc = "小区离宋家庄地铁站只有100米距离，宋家庄是一个交通枢纽，有地铁5号线10号线还有亦庄线，所以说交通非常方便\n" +
                "房子是一个三居室，出租主卧，房间里面很干净，家电齐全，可以拎包入住\n" +
                "小区周边有多个大型超市和商场，有首开福茂，美廉美超市，物美超市等等生活和购物都很方便\n" +
                "要求租客有正当工作，下班不要打扰邻居间的休息，别扰民就行";
        return detail;
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mHousePhotoViewController.detachedRoot();
        mHouseParamViewControler.detachedRoot();
        mHouseDescViewControler.detachedRoot();
        mHouseCommentViewControler.detachedRoot();
        mHouseParamTopViewControler.detachedRoot();
        mHouseParamBottomViewControler.detachedRoot();
    }
}

```

main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<!--
  ~ Copyright 2016 GuDong
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->

<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#f9f5f5"
    android:id="@+id/root_layout"
    android:orientation="vertical">

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:id="@+id/ll_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

        </LinearLayout>
    </ScrollView>

    <ProgressBar
        android:id="@+id/pb"
        style="@style/Widget.AppCompat.ProgressBar"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:layout_gravity="center"
        android:visibility="gone"/>
</FrameLayout>
```


**over**