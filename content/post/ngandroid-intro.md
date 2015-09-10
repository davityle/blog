+++
Categories = ["Development", "Android"]
Description = "An introduction to the ngAndroid library"
Tags = ["android", "angular"]
date = "2015-02-18T00:05:28-07:00"
menu = "main"
title = "NgAndroid Introduction"
+++

![NgAndroid](/images/ngandroid.png)

<br>

It's finally here. The Android UI framework has become as simple and powerful as [AngularJS](https://angularjs.org/). Sort of.

The project [ngAndroid](https://github.com/davityle/ngAndroid) (which is still in beta) on Github has done something that some might not think is possible; NgAndroid brings Angular type directives and data-binding to Android xml attributes.

If you don't know anything about AngularJS then you probably don't know why this is such an exciting anouncement. You can learn more at Angular's
[website](https://angularjs.org/) but a previous knowledge of AngularJS is not required to revel in the epicness that is about to be laid before your eyes.

<!--more-->


## Two Way Data Binding

When you create a view in XML that the user is going to interact with you usually end up with something that looks like this.
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    ...>

    <TextView
        android:id="@+id/tv_title"
        .../>

    <CheckBox
        android:id="@+id/cb_active"
        .../>

    <EditText
        android:id="@+id/et_text"
        ... />

    <ImageView
        android:id="@+id/iv_ico"
        .../>
</RelativeLayout>
```
```
public class Model {
    private String title;
    private String text;
    private boolean isActive;

    public boolean isActive() {
        return isActive;
    }

    public void setActive(boolean isActive) {
        this.isActive = isActive;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}

// View

private Model model;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_demo);

    TextView tv = (TextView) findViewById(R.id.tv_title);
    tv.setText(model.getTitle());

    CheckBox cb = (CheckBox) findViewById(R.id.cb_active);
    cb.setChecked(model.isActive());
    cb.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
        @Override
        public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
            model.setIsActive(isChecked);
        }
    });


    EditText et = (EditText) findViewById(R.id.et_text);
    et.addTextChangedListener(new TextWatcher() {
        @Override public void beforeTextChanged(CharSequence s, int start, int count, int after) {}
        @Override public void onTextChanged(CharSequence s, int start, int before, int count) {}
        @Override
        public void afterTextChanged(Editable s) {
            model.setText(s.toString());
        }
    });

    findViewById(R.id.iv_ico).setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            // do stuff here
        }
    });

}
```

Most of that code is template code. You are doing the same mundane work every time you make a new view.

>Most templating systems bind data in only one direction: they merge template and model components together into a view. After the merge occurs, changes to the model or related sections of the view are NOT automatically reflected in the view. Worse, any changes that the user makes to the view are not reflected in the model. This means that the developer has to write code that constantly syncs the view with the model and the model with the view.
>
> -AngularJs
>(https://docs.angularjs.org/guide/databinding)

What if you could bind your view to your model in a two way fashion. All of the changes to your model would automatically be propogated to your view and all the changes the user made in the view would automatically be refected in your model.

>The model is the single-source-of-truth for the application state, greatly simplifying the programming model for the developer. You can think of the view as simply an instant projection of your model.
>
> -AngularJs
>(https://docs.angularjs.org/guide/databinding)

With ngAndroid now your view can look like this.
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:ngAndroid="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingPrefix"
    ...>

    <TextView
        android:id="@+id/tv_title"
        ngAndroid:ngModel="model.title"
        .../>

    <CheckBox
        android:id="@+id/cb_active"
        ngAndroid:ngModel="model.isActive"
        .../>

    <EditText
        android:id="@+id/et_text"
        ngAndroid:ngModel="model.text"
        .../>

    <ImageView
        android:id="@+id/iv_ico"
        ngAndroid:ngClick="onIvClick()"
        .../>
</RelativeLayout>
```
```
public interface Model {
    public boolean isActive();
    public void setIsActive(boolean isActive);
    public String getText();
    public void setText(String text);
    public String getTitle();
    public void setTitle(String title);
}

private NgAndroid ngAndroid = NgAndroid.getInstance();
private Model model;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ngAndroid.setContentView(this, R.layout.activity_ng_android);
}

private void onIvClick(){
    // do something with the click
}
```
With just that you have two way databinding. Your model will always reflect your application state. We also get identical funcionality to the previous code. Pretty sweet ey?


[Most](http://www.wintellect.com/devcenter/jlikness/10-reasons-web-developers-should-learn-angularjs) [people](http://code.tutsplus.com/tutorials/5-awesome-angularjs-features--net-25651) [agree](http://anandmanisankar.com/posts/angularjs-best-parts/) that two-way data binding is an amazing, if not the best, feature of AngularJS. I tend to agree with them. I also believe that it is one of the greatest features of NgAndroid but it does not end there.

Stay tuned more more posts on this exciting new library and in the meantime check out the [NgAndroid github repo](https://github.com/davityle/ngAndroid)


---
"[AngularJS logo.svg](https://github.com/angular/angular.js/tree/master/images/logo)" by [AngularJS](https://angularjs.org/) is licensed under <a rel="nofollow" class="external text" href="http://creativecommons.org/licenses/by-sa/3.0/">Creative Commons Attribution-ShareAlike 3.0 Unported License</a>
