# shadowview

## 1 前言
有一个UI设计的朋友，因为涉及到一个组件阴影均匀分布投影的问题，她们公司的程序猿，不愿意去思考如何实现，遂找我帮忙，而我不太会Android，之前稍微看了一点点。经过一番思考，完成了组件的定制，达到了朋友的需求。

## 2 实现

```java
package com.micah.shadowview.view;

/**
 * @Author m.kong
 * @Date 2021/3/27 上午11:57
 * @Version 1
 * @Description
 */

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.*;
import android.graphics.drawable.BitmapDrawable;
import android.os.Build;
import android.util.AttributeSet;
import android.widget.FrameLayout;
import com.micah.shadowview.R;

public class ShadowLayout extends FrameLayout {

    /**
     * 阴影透明度, float类型，取值范围：0 - 1
     */
    private float shadowAlpha;
    /**
     * 阴影颜色
     */
    private int mShadowColor;
    /**
     * 阴影的扩散范围(也可以理解为扩散程度)
     */
    private float mShadowLimit;
    /**
     * 阴影的圆角大小
     */
    private float mCornerRadius;
    /**
     * x轴的偏移量
     */
    private float mDx;
    /**
     * y轴的偏移量
     */
    private float mDy;
    /**
     * 左边是否显示阴影
     */
    private boolean leftShow;
    /**
     * 右边是否显示阴影
     */
    private boolean rightShow;
    /**
     * 上边是否显示阴影
     */
    private boolean topShow;
    /**
     * 下面是否显示阴影
     */
    private boolean bottomShow;


    private boolean mInvalidateShadowOnSizeChanged = true;
    private boolean mForceInvalidateShadow = false;

    public ShadowLayout(Context context) {
        super(context);
        initView(context, null);
    }

    public ShadowLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context, attrs);
    }

    public ShadowLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    @Override
    protected int getSuggestedMinimumWidth() {
        return 0;
    }

    @Override
    protected int getSuggestedMinimumHeight() {
        return 0;
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (w > 0 && h > 0 && (getBackground() == null || mInvalidateShadowOnSizeChanged
                || mForceInvalidateShadow)) {
            mForceInvalidateShadow = false;
            setBackgroundCompat(w, h);
        }
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        if (mForceInvalidateShadow) {
            mForceInvalidateShadow = false;
            setBackgroundCompat(right - left, bottom - top);
        }
    }

    public void setInvalidateShadowOnSizeChanged(boolean invalidateShadowOnSizeChanged) {
        mInvalidateShadowOnSizeChanged = invalidateShadowOnSizeChanged;
    }

    public void invalidateShadow() {
        mForceInvalidateShadow = true;
        requestLayout();
        invalidate();
    }

    private void initView(Context context, AttributeSet attrs) {
        initAttributes(context, attrs);

        int xPadding = (int) (mShadowLimit + Math.abs(mDx));
        int yPadding = (int) (mShadowLimit + Math.abs(mDy));
        int left;
        int right;
        int top;
        int bottom;
        if (leftShow) {
            left = xPadding;
        } else {
            left = 0;
        }

        if (topShow) {
            top = yPadding;
        } else {
            top = 0;
        }


        if (rightShow) {
            right = xPadding;
        } else {
            right = 0;
        }

        if (bottomShow) {
            bottom = yPadding;
        } else {
            bottom = 0;
        }

        setPadding(left, top, right, bottom);
    }

    @SuppressWarnings("deprecation")
    private void setBackgroundCompat(int w, int h) {
        Bitmap bitmap = createShadowBitmap(w, h, mCornerRadius, mShadowLimit, mDx,
                mDy, mShadowColor, Color.TRANSPARENT);
        BitmapDrawable drawable = new BitmapDrawable(getResources(), bitmap);
        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.JELLY_BEAN) {
            setBackgroundDrawable(drawable);
        } else {
            setBackground(drawable);
        }
    }


    private void initAttributes(Context context, AttributeSet attrs) {
        TypedArray attr = getTypedArray(context, attrs, R.styleable.ShadowLayout);
        if (attr == null) {
            return;
        }

        try {
            //默认是显示
            shadowAlpha = attr.getFloat(R.styleable.ShadowLayout_yc_alpha,1);
            leftShow = attr.getBoolean(R.styleable.ShadowLayout_yc_leftShow, true);
            rightShow = attr.getBoolean(R.styleable.ShadowLayout_yc_rightShow, true);
            bottomShow = attr.getBoolean(R.styleable.ShadowLayout_yc_bottomShow, true);
            topShow = attr.getBoolean(R.styleable.ShadowLayout_yc_topShow, true);

            mCornerRadius = attr.getDimension(R.styleable.ShadowLayout_yc_cornerRadius, 0);
            mShadowLimit = attr.getDimension(R.styleable.ShadowLayout_yc_shadowLimit, 0);
            mDx = attr.getDimension(R.styleable.ShadowLayout_yc_dx, 0);
            mDy = attr.getDimension(R.styleable.ShadowLayout_yc_dy, 0);
            mShadowColor = attr.getColor(R.styleable.ShadowLayout_yc_shadowColor,
                    getResources().getColor(R.color.default_shadow_color));
        } finally {
            attr.recycle();
        }
    }

    private TypedArray getTypedArray(Context context, AttributeSet attributeSet, int[] attr) {
        return context.obtainStyledAttributes(attributeSet, attr, 0, 0);
    }

    private Bitmap createShadowBitmap(int shadowWidth, int shadowHeight, float cornerRadius,
                                      float shadowRadius, float dx, float dy,
                                      int shadowColor, int fillColor) {

        //根据宽高创建bitmap背景
        Bitmap output = Bitmap.createBitmap(shadowWidth, shadowHeight, Bitmap.Config.ARGB_8888);
        //用画板canvas进行绘制
        Canvas canvas = new Canvas(output);
        RectF shadowRect = new RectF(shadowRadius, shadowRadius,
                shadowWidth - shadowRadius, shadowHeight - shadowRadius);

        if (dy > 0) {
            shadowRect.top += dy;
            shadowRect.bottom -= dy;
        } else if (dy < 0) {
            shadowRect.top += Math.abs(dy);
            shadowRect.bottom -= Math.abs(dy);
        }

        if (dx > 0) {
            shadowRect.left += dx;
            shadowRect.right -= dx;
        } else if (dx < 0) {
            shadowRect.left += Math.abs(dx);
            shadowRect.right -= Math.abs(dx);
        }

        Paint shadowPaint = new Paint();
        shadowPaint.setAntiAlias(true);
        shadowPaint.setAlpha((int) shadowAlpha*255);
        shadowPaint.setColor(fillColor);
        shadowPaint.setStyle(Paint.Style.FILL);
        if (!isInEditMode()) {
            shadowPaint.setShadowLayer(shadowRadius, dx, dy, shadowColor);
        }
        canvas.drawRoundRect(shadowRect, cornerRadius, cornerRadius, shadowPaint);
        return output;
    }
}

```
属性配置文件`attrs.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="ShadowLayout">
        <!--阴影的圆角大小-->
        <attr name="yc_cornerRadius" format="dimension" />
        <!--阴影的扩散范围(也可以理解为扩散程度)-->
        <attr name="yc_shadowLimit" format="dimension" />
        <!--阴影颜色-->
        <attr name="yc_shadowColor" format="color" />
        <!--x轴的偏移量-->
        <attr name="yc_dx" format="dimension" />
        <!--y轴的偏移量-->
        <attr name="yc_dy" format="dimension" />
        <!--左边是否显示阴影-->
        <attr name="yc_leftShow" format="boolean" />
        <!--右边是否显示阴影-->
        <attr name="yc_rightShow" format="boolean" />
        <!--上边是否显示阴影-->
        <attr name="yc_topShow" format="boolean" />
        <!--下面是否显示阴影-->
        <attr name="yc_bottomShow" format="boolean" />
        <!--阴影透明度-->
        <attr name="yc_alpha" format="float"/>
    </declare-styleable>
</resources>


```

## 3 使用示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        tools:context=".MainActivity">

    <com.micah.shadowview.view.ShadowLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_marginTop="10dp"
            app:yc_dx="0dp"
            app:yc_dy="1dp"
            app:yc_cornerRadius="5dp"
            app:yc_alpha="0.16"
            app:yc_shadowColor="#1F385781"
            app:yc_shadowLimit="4dp">
        <TextView
                android:layout_width="351dp"
                android:layout_height="99dp"
                android:gravity="center"
                android:paddingLeft="10dp"
                android:paddingRight="10dp"
                android:text="这是一个自定义的ShadowLayout"
                android:background="@drawable/ic_view_box"
                android:textColor="#000000" />
    </com.micah.shadowview.view.ShadowLayout>

</RelativeLayout>

```

## 4 效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210328162621964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1ODQzNTE0,size_16,color_FFFFFF,t_70)

