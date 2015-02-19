+++
Categories = ["Development", "Android"]
Description = "An introduction to the ngAndroid library"
Tags = ["android", "angular"]
date = "2015-02-18T00:05:28-07:00"
menu = "main"
title = "NgAndroid Introduction"
draft=true

+++

![Screenshot of stripes](/images/ngandroid.png)

<br>

It's finally here. The Android UI framework has become as simple and powerful as [AngularJS](https://angularjs.org/). Sort of. 

The project [ngAndroid](https://github.com/davityle/ngAndroid) on Github has done something that some might not think is possible; NgAndroid brings Angular type directives and data-binding to Android xml attributes. 

If you don't know anything about AngularJS then you probably don't know why this is such an exciting anouncement. You can learn more at Angular's 
[website](https://angularjs.org/) but a previous knowledge of AngularJS is not required to revel in the epicness that is about to be laid before your eyes. 


## Two Way Data Binding

When you create a view in XML that the user is going to interact with you usually end up with something that looks like this.
```
xml here
```
```
java model and listeners here
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
model
```
```
xml
```
With just that, and NgAndroid, you have two way databinding. Your model will always reflect your application state. 

[Most](http://www.wintellect.com/devcenter/jlikness/10-reasons-web-developers-should-learn-angularjs) [people](http://code.tutsplus.com/tutorials/5-awesome-angularjs-features--net-25651) [agree](http://anandmanisankar.com/posts/angularjs-best-parts/) that two-way data binding is an amazing, if not the best, feature of AngularJS. I tend to agree with them. I also believe that it is one of the greatest features of NgAndroid but it does not end there.





"[AngularJS logo.svg](https://github.com/angular/angular.js/tree/master/images/logo)" by [AngularJS](https://angularjs.org/) is licensed under <a rel="nofollow" class="external text" href="http://creativecommons.org/licenses/by-sa/3.0/">Creative Commons Attribution-ShareAlike 3.0 Unported License</a>