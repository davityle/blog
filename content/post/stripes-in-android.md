+++
Categories = ["android"]
Description = "Easily add stripes to any of your views"
Tags = ["dev", "android", "ui", "animation"]
date = "2015-02-07T11:30:53-07:00"
title = "Creating a striped view in Android"

+++

Have you ever said to yourself, 'Wow, this view needs some stripes, but there is no android:stripes="true" attribute and I don't know how to draw stripes using the canvas'? If so, this is the perfect place for you. Or, if you need to draw stripes and figured you'd check the internet for an easy solution before you tried it yourself than you are also in the right place.

For those of you who just want the code here it is. 

```
public class StripedView extends View {

    private static final float STRIPE_WIDTH_PERCT = .05f;
    private static final float STRIPE_OFFSET_PERCT = .2f;
    private Path stripespath = new Path();
    private Paint blackPaint = new Paint();

    public StripedView(Context context) {
        super(context);
    }

    public StripedView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public StripedView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public StripedView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
    //init()
    {
        blackPaint.setAntiAlias(true);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        drawStripes(canvas, 0, getHeight(), 0, getWidth(), blackPaint);
    }

    private void drawStripes(Canvas canvas, float topy, float bottomy, float startx, float endx, Paint p){
        if(bottomy - topy < 1){
            return;
        }
        stripespath.reset();
        float width = (bottomy - topy) * STRIPE_WIDTH_PERCT;
        float offset = (bottomy - topy) * STRIPE_OFFSET_PERCT;
        for(float xPos = startx - offset; xPos <= endx; xPos += offset + width){
            stripespath.moveTo(xPos, topy);
            stripespath.lineTo(xPos + width, topy);
            stripespath.lineTo(xPos + offset + width, bottomy);
            stripespath.lineTo(xPos + offset, bottomy);
            stripespath.lineTo(xPos, topy);
        }
        canvas.drawPath(stripespath, p);
    }
}
```
Which will result in something that looks like this.

![Screenshot of stripes](/images/device-2015-02-07-125851.png)


Drawing stripes is pretty straight forward. You need a defined width of your stripe and an offset between the top and the bottom of the stripe. If the [Canvas](http://developer.android.com/reference/android/graphics/Canvas.html) class in Android had a draw polygon class then we could then just draw a bunch of Polygons. Instead we have to use the [Path](http://developer.android.com/reference/android/graphics/Path.html) object. Fortunately, the Path object is fairly simple and intuitive to use.

Grab your width of your stripe. I'm using a percentage of height of the view but any width is fine.
```
float width = (bottomy - topy) * STRIPE_WIDTH_PERCT;
```
Next grab an offset for each stripe. 
```
float offset = (bottomy - topy) * STRIPE_OFFSET_PERCT;
```
Now you need to loop across the width of your view. For this example I start off of the view by the width of the offset in order to get the end of the first stripe at the bottom left of the view. 
```
float xPos = startx - offset
```
You'll want to increment the x position by the offset plus the width so that you get evenly spaced stripes. This is important if you want to animate it and have it look good.

When you draw your stripes you can actually use a single path for all of the stripes. You 'move' the path to the top left corner of the stripe which is just (x,y) then you draw a line across to (x + stripeWidth, y), down to (x + stripeWidth + offset, y + height), over to (x + offset, y + height, and then back to (x,y). Then you increment x by the width and the offset and repeat the process until your x position is greater than the width of your view.

You're code will look something like this.
```
for(float xPos = startx - offset; xPos <= endx; xPos += offset + width){
    stripespath.moveTo(xPos, topy);
    stripespath.lineTo(xPos + width, topy);
    stripespath.lineTo(xPos + offset + width, bottomy);
    stripespath.lineTo(xPos + offset, bottomy);
    stripespath.lineTo(xPos, topy);
}
```

After that drawing it is as simple as 
```
canvas.drawPath(stripespath, p);
```
Now lets have some fun with it. What would happen if we changed our starting position based off of what time it was.
```
private static final long ANIMATION_SPEED = 30;

float xPos = startx - (System.currentTimeMillis()/ANIMATION_SPEED) % (long)(offset + width)
```
and then redrew the view at a regular interval.
```
private static final long REFRESH_RATE = 33;
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    drawStripes(canvas, 0, getHeight(), 0, getWidth(), blackPaint);
    postDelayed(invalidateRunnable, REFRESH_RATE);
}

private final Runnable invalidateRunnable = new Runnable() {
    public void run() {
        invalidate();
    }
};
```

You would get this.

![Animated Stripes](/images/imageedit_3_3395727418.gif)


Or what if you only want to draw the stripes in a small area of your view. Then you can clip the canvas to only draw in the section that you have clipped.
```
canvas.save();
canvas.clipRect(20, 20, getWidth() - 20, getHeight() - 20);
drawStripes(canvas, 20, getHeight() - 20, 20, getWidth() - 20, redPaint);
canvas.restore();
```

![Clipped Stripes](/images/device-2015-02-07-153205.png)

You can flip the direction of the stripes by swapping the y values.
```
stripespath.moveTo(xPos, bottomy);
stripespath.lineTo(xPos + width, bottomy);
stripespath.lineTo(xPos + offset + width, topy);
stripespath.lineTo(xPos + offset, topy);
stripespath.lineTo(xPos, bottomy);
```
![Swapped Values](/images/device-2015-02-07-153631.png)

And finally you can draw stripes on any view that you want. Such as a Button.
```
public class StripedButton extends Button {
    private static final float STRIPE_WIDTH_PERCT = .1f;
    private static final float STRIPE_OFFSET_PERCT = .15f;
    private Path stripespath = new Path();
    private Paint redPaint = new Paint();

    public StripedButton(Context context) {
        super(context);
    }

    public StripedButton(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public StripedButton(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public StripedButton(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }
    //init()
    {
        redPaint.setAntiAlias(true);
        redPaint.setColor(0xFFFF5555);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        drawStripes(canvas, getPaddingLeft(), getHeight() - getPaddingBottom(), getPaddingLeft(), getWidth() - getPaddingRight(), redPaint);
        super.onDraw(canvas);
    }

    private void drawStripes(Canvas canvas, float topy, float bottomy, float startx, float endx, Paint p){
        if(bottomy - topy < 1){
            return;
        }
        stripespath.reset();
        float width = (bottomy - topy) * STRIPE_WIDTH_PERCT;
        float offset = (bottomy - topy) * STRIPE_OFFSET_PERCT;
        for(float xPos = startx - offset; xPos <= endx; xPos += offset + width){
            stripespath.moveTo(xPos, bottomy);
            stripespath.lineTo(xPos + width, bottomy);
            stripespath.lineTo(xPos + offset + width, topy);
            stripespath.lineTo(xPos + offset, topy);
            stripespath.lineTo(xPos, bottomy);
        }
        canvas.drawPath(stripespath, p);
    }
}
```
![Button With Stripes](/images/device-2015-02-07-154645.png)
