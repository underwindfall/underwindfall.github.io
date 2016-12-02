---
layout:     post
title:      "漫说ProgressBar之基础篇"
subtitle:   " \"ProgressBar详解以及自定义(一)\""
date:       2016-12-01 12:00:00
author:     "Undervoid"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 磨砻淬砺集
    - Android基础
---


# ProgressBar详解以及自定义(一)

>其中牵扯到一些关于MeasureSpec的知识可以去我的另一篇文章看下。

## 概念

【ProgressBar】既进度条，当我们在做一些耗时操作的时候（例如下载文件），可以使用ProgressBar给用户提供一个进度提示，告诉用户当前的进度。ProgressBar提供了两种进度显示模式，分别是具有进度值的【精确模式】和不具有进度值的【模糊模式】

## ProgessBar属性小结

>以下列举最常用的属性，详情请[点击](https://developer.android.com/reference/android/widget/ProgressBar.html)

* style
    * Widget.ProgressBar.Horizontal     =>横向进度条
    * Widget.ProgressBar                =>中号的圆形进度条
    * Widget.ProgressBar.Small          =>小号的圆形进度条
    * Widget.ProgressBar.Large          =>大号的圆形进度条
    * Widget.ProgressBar.Inverse        =>中号的圆形进度条
* attributeName
    * android:indeterminate:Allows to enable the indeterminate mode. [boolean] :允许使用不确定模式,true是启用不确定模式，false反之。
    * android:indeterminateBehavior：Defines how the indeterminate mode should behave when the progress reaches max.有两种(repeat;cycle);
    * android:indeterminateDrawable:Drawable used for the indeterminate mode.:定义不确定模式是否可drawable
    * android:indeterminateDuration:Duration of the indeterminate animation. ：设置模糊状态下ProgressBar一个周期动画的持续时间
    * android:progress:Defines the default progress value, between 0 and max.[integer]:定义默认的进度值
    * android:progressBackgroundTint:Tint to apply to the progress indicator background.:进度条的背景颜色
    * android:progressBackgroundTintMode：Blending mode used to apply the progress indicator background tint.
    * android:secondaryProgress:Defines the secondary progress value, between 0 and max.
    * android:secondaryProgressTint:Tint to apply to the secondary progress indicator.
    * android:secondaryProgressTintMode:Blending mode used to apply the secondary progress indicator tint.

## ProgessBar使用方法

>目前只介绍水平进度条使用，其他的类型也类似可以模仿使用。


```xml
//XML
<ProgressBar  
     android:id="@+id/progressbar"  
     style="@android:style/Widget.ProgressBar.Horizontal"  
     android:layout_width="match_parent"  
     android:layout_height="wrap_content"  
     android:secondaryProgress="70" />  
```

这时候整个ProgressBar已经被系统的默认样式设置。

```xml
<style name="Widget.ProgressBar.Horizontal">  
    <item name="android:indeterminateOnly">false</item>  
    <item name="android:progressDrawable">@android:drawable/progress_horizontal</item>  
    <item name="android:indeterminateDrawable">@android:drawable/progress_indeterminate_horizontal</item>  
    <item name="android:minHeight">20dip</item>  
    <item name="android:maxHeight">20dip</item>  
    <item name="android:mirrorForRtl">true</item>  
</style>  

```

在相对应的Fragment或者Activity中，声明调用就可。就想其他控件一样。

```java
//JAVA
public class MainActivity extends Activity implements OnClickListener {
    private ProgressBar progressBar;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            button = (Button) findViewById(R.id.button);
            editText = (EditText) findViewById(R.id.edit_text);
            imageView = (ImageView) findViewById(R.id.image_view);
            progressBar = (ProgressBar) findViewById(R.id.progress_bar);
            button.setOnClickListener(this);
        }
}
```

下面说下ProgressBar的方法.总体来说，可以分为两个部分。一是和自身属性相关的，比如获取进度、设置进度的最大值、设置插入器等等。二是和绘制相关的部分，如图所示：
![](https://s3.eu-central-1.amazonaws.com/undervoidfall/Android+Notes/Progressbar/ProgresBar_method.jpg)

## ProgessBar 自定义使用

这次自己选择了使用创建新的MyProgressBar类继承ProgressBar. 话不多说直接上代码。 在这次自定义的过程中主要学会了，用Canvas来画圆弧赛贝尔曲线。
完成图：
![](https://s3.eu-central-1.amazonaws.com/undervoidfall/Android+Notes/Progressbar/Progressbar_finish.jpg)

```xml
//XML
<com.custome.MyProgressBarHorizontal
            android:id="@+id/event_progress_bar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentStart="true"
            android:layout_below="@id/container"
            android:layout_gravity="bottom"
            android:layout_marginBottom="6dp"
            android:layout_marginLeft="60dp"
            android:layout_marginRight="50dp"
            android:progress="60"
            pb:gradient="true"
            pb:reachColor="@color/colorPrimaryDark"
            pb:reachHeight="8dp"
            pb:unReachColor="#9E9E9E"
            pb:unReachHeight="8dp"/>
```

```java
//JAVA
public class MyProgressBarHorizontal extends ProgressBar {

    private static final int DEFAUT_COLOR_UNREACH = 0XFFD3D6DA;
    private static final int DEFAUT_HEIGHT_UNREACH = 2; //DP
    private static final int DEFAUT_COLOR_REACH = 0XFFFC00D1;
    private static final int DEFAUT_HEIGHT_REACH = 2;

    private int mReachColor = DEFAUT_COLOR_REACH;
    private int mReachHeight = DEFAUT_HEIGHT_REACH;
    private int mUnReachColor = DEFAUT_COLOR_UNREACH;
    private int mUnReachHeight = DEFAUT_HEIGHT_UNREACH;

    //渐变颜色的配置
    private boolean needGradient = true;
    private int mStartColor = Color.BLUE;
    //    private int mMiddleColor = Color.GREEN;
    private int mEndColor = Color.GREEN;

    private Context context;
    private Shader mShader;
    private int mRealWidth;
    private Paint mReachPaint = new Paint();
    private Paint mUnReachPaint = new Paint();

    public MyProgressBarHorizontal(Context context) {
        this(context, null);
    }

    public MyProgressBarHorizontal(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyProgressBarHorizontal(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.context = context;
        obtainStyleAttrs(attrs);
    }

    private void obtainStyleAttrs(AttributeSet attrs) {
        TypedArray ta = getContext().obtainStyledAttributes(attrs, R.styleable.MyProgressBarHorizontal);
        mReachColor = ta.getColor(R.styleable.MyProgressBarHorizontal_reachColor, DEFAUT_COLOR_REACH);
        mUnReachColor = ta.getColor(R.styleable.MyProgressBarHorizontal_unReachColor, DEFAUT_COLOR_UNREACH);
        mReachHeight = (int) ta.getDimension(R.styleable.MyProgressBarHorizontal_reachHeight, DensityUtil.dp2px(context, DEFAUT_HEIGHT_REACH));
        mUnReachHeight = (int) ta.getDimension(R.styleable.MyProgressBarHorizontal_unReachHeight, DensityUtil.dp2px(context, DEFAUT_HEIGHT_UNREACH));
        needGradient = ta.getBoolean(R.styleable.MyProgressBarHorizontal_gradient, false);
        ta.recycle();
        //新建一个线性渐变，前两个参数是渐变开始的点坐标，
        // 第三四个参数是渐变结束的点的坐标。连接这2个点就拉出一条渐变线了，玩过PS的都懂。
        // 然后那个数组是渐变的颜色。
        //下一个参数是渐变颜色的分布，如果为空，每个颜色就是均匀分布的。
        // 最后是模式，这里设置的是循环渐变
        mShader = new LinearGradient(0, 0, 200, 0, mStartColor, mEndColor, Shader.TileMode.MIRROR);
    }

    @Override
    protected synchronized void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = measureHeight(heightMeasureSpec);
        setMeasuredDimension(width, height);
        mRealWidth = getMeasuredWidth() - getPaddingLeft() - getPaddingRight();
    }

    /**
     * 测量高
     * 重写onMeasure方法时要根据模式不同进行尺寸计算。下面代码就是一种比较典型的方式：
     *
     * @param measureSpec
     * @return
     */
    private int measureHeight(int measureSpec) {
        int mode = MeasureSpec.getMode(measureSpec);
        int size = MeasureSpec.getSize(measureSpec);
        int result = 0;
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else {
            //result=2图片最高的那个 + 2个padding 这是自己定义的
            result = Math.max(mReachHeight, mUnReachHeight) + getPaddingTop() + getPaddingBottom();
            if (mode == MeasureSpec.AT_MOST) {
                result = Math.min(size, result);
            }
        }
        return result;
    }

    @Override
    protected synchronized void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。
        canvas.save();
        canvas.translate(getPaddingLeft(), 0);
        //draw reach bar
        boolean needUnReach = false;
        float radio = getProgress() * 1.0f / getMax();
        if (radio < 1) {
            needUnReach = true;
        }
        //draw each bar
        float endX = mRealWidth * radio;
        if (endX > 0) {
            //画笔设置填充风格为实心
            mReachPaint.setStyle(Paint.Style.FILL);
            //设置抗锯齿
            mReachPaint.setAntiAlias(true);
            if (needGradient) {
                mReachPaint.setShader(mShader);
            } else {
                mReachPaint.setColor(mReachColor);
            }
            canvas.drawPath(drawReachBarPath(endX), mReachPaint);
        }
        //draw unreach bar
        if (needUnReach) {
            mUnReachPaint.setColor(mUnReachColor);
            mUnReachPaint.setStyle(Paint.Style.FILL);//设置实心
            mUnReachPaint.setAntiAlias(true);
            canvas.drawPath(drawUnReachBarPath(endX), mUnReachPaint);
        }
        //来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。
        canvas.restore();
    }

    private Path drawUnReachBarPath(float endX) {
        int r_cycle = getHeight() / 2;
        Path path = new Path();
        if (endX > 0) {
            path.moveTo(endX - r_cycle, 0);
            path.lineTo(mRealWidth - r_cycle, 0);
            path.quadTo(mRealWidth - r_cycle, r_cycle, mRealWidth - r_cycle, getHeight());
            path.lineTo(endX - r_cycle, getHeight());
            path.quadTo(endX-r_cycle,r_cycle,endX-r_cycle,0);
//            path.moveTo(endX - r_cycle, 0);
//            path.quadTo(endX, r_cycle, endX - r_cycle, getHeight()); //赛贝尔曲线画圆弧
//            path.lineTo(mRealWidth - r_cycle, getHeight());
//            path.quadTo(mRealWidth, r_cycle, mRealWidth - r_cycle, 0); //赛贝尔曲线画圆弧
//            path.lineTo(mRealWidth, 0);
            path.close();
        } else {
            path.moveTo(r_cycle, getHeight());
            path.lineTo(mRealWidth - r_cycle, getHeight());
            path.quadTo(mRealWidth - r_cycle, r_cycle, mRealWidth - r_cycle, 0);
            path.lineTo(r_cycle, 0);
            path.quadTo(0, r_cycle, r_cycle, getHeight());
            path.close();
        }
        return path;
    }

    //进度条边上圆弧赛贝尔曲线的执行点
    private Path drawReachBarPath(float endX) {
        int r_cycle = getHeight() / 2;
        Path path = new Path();
        //设置Path的起点
        path.moveTo(r_cycle, 0);
        // 添加上一个点到当前点之间的直线到Path
        path.lineTo(endX - r_cycle, 0);
        //二节
        path.quadTo(endX, r_cycle, endX - r_cycle, getHeight()); //赛贝尔曲线画圆弧
        path.lineTo(r_cycle, getHeight());
        path.quadTo(0, r_cycle, r_cycle, 0); //赛贝尔曲线画圆弧
        path.close();//封闭
        return path;
    }
}
```









