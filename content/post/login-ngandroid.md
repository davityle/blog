+++
Categories = ["Development", "Android"]
Description = "Android login screen with NgAndroid"
Tags = ["android", "ngandroid"]
date = "2015-09-09T21:15:59-06:00"
menu = "main"
title = "Login Screen with NgAndroid"

+++

#### Edit*

With the release of the official [Android data binding library](http://developer.android.com/tools/data-binding/guide.html) NgAndroid has been deprecated.

-----

<br>
<br>
I really don't like writing user interface glue code. Enough said about that.

#### TL;DR

[NgAndroid](https://github.com/davityle/ngAndroid) is an <strong>effecient compile time annotation processor</strong> that generates <strong>two data binding</strong> and layout controllers for Android <strong>MVC</strong>. [NgAndroid](https://github.com/davityle/ngAndroid) is currently unstable.

The code for a login screen can be found below or [here](https://github.com/davityle/NgAndroid-Login-Demo).
<!--more-->

###### Controller Scope
{{< highlight java>}}
@NgScope(name="Login")
public class LoginScope {
    @NgModel
    User user;
    @NgModel
    NetworkCall call;

    void onSubmit(Context context) {
        call.setActive(true);
        Toast.makeText(context, user.getUsername() + " : " +user.getPassword(), Toast.LENGTH_SHORT ).show();
    }
}
{{< /highlight>}}
###### User Model
{{< highlight java>}}
public class User {
    private String username = "", password = "";

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
{{< /highlight>}}
###### Layout
{{< highlight xml>}}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:x="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="60dp"
    tools:context=".LoginActivity"
    tools:ignore="MissingPrefix"
    x:ngScope="Login">

    <EditText
        android:id="@+id/username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="text"
        android:layout_above="@+id/password"
        x:ngModel="user.username"/>

    <EditText
        android:id="@+id/password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="textPassword"
        android:layout_above="@+id/submit"
        x:ngModel="user.password"/>

    <Button
        android:id="@+id/submit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="@string/submit"
        x:ngDisabled="user.username.length() &lt; 6 || user.password.length() &lt; 6"
        x:ngClick="onSubmit($view.context)"
        x:ngInvisible="call.active"/>


    <ProgressBar
        android:id="@+id/progress"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        x:ngInvisible="!call.active"/>

</RelativeLayout>
{{< /highlight>}}
###### Activity
{{< highlight java>}}
public class LoginActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        LoginScope scope = new LoginScope();
        ActivityLoginController controller = new ActivityLoginController(new NgOptions.Builder().build(), scope);
        controller.attach(findViewById(android.R.id.content));
    }

}
{{< /highlight>}}

###### Product
![Working login](/images/working_login.gif)

#### Explanation

Here is how to create an Android login screen using [NgAndroid](https://github.com/davityle/ngAndroid).

I'm going to assume that you know how to set up an Android project. If you don't you can start [here](http://developer.android.com/training/basics/firstapp/index.html)

Create a new Blank Activity called LoginActivity and open activity_login.xml

![Android Studio create activity view](/images/new_activity.png)

Change the root view of the layout to `RelativeLayout` and change the padding to `android:padding="60dp"`.

Add a `Button` and two `EditText` views.

{{< highlight xml>}}
<EditText
    android:id="@+id/username"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="text"
    android:layout_above="@+id/password"/>

<EditText
    android:id="@+id/password"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="textPassword"
    android:layout_above="@+id/submit"/>

<Button
    android:id="@+id/submit"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Submit"
    android:layout_centerInParent="true"/>
{{< /highlight>}}

If you run your app you should have something that looks like this (on Android Lollipop).

![App screenshot](/images/not_disabled.png)

Now lets add in NgAndroid. If you open up your project build.gradle file you will see something like this.


{{< highlight groovy>}}
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.0'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

{{< /highlight>}}

In buildscript.dependencies add a new classpath `classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'` and in allprojects.repositories add
{{< highlight groovy>}}
maven {
    url 'http://oss.sonatype.org/content/repositories/snapshots'
}
{{< /highlight>}}
So your build.gradle would then look like this.

{{< highlight groovy>}}
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.0'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()

        maven {
            url 'http://oss.sonatype.org/content/repositories/snapshots'
        }
    }
}
{{< /highlight>}}
The maven.url is only necessary because you are going to be using a snapshot build of NgAndroid.

Now open up your app's build.gradle and add `apply plugin: 'com.neenbedankt.android-apt'` below `apply plugin: 'com.android.application'` and add
{{< highlight groovy>}}
compile 'com.github.davityle:ngandroid:1.0.10-SNAPSHOT'
apt 'com.github.davityle:ng-processor:1.0.10-SNAPSHOT'
{{< /highlight>}}
to your dependencies.

Now create a class called LoginScope. Your scope is important. It is the base reference for all of your bindings.
To make LoginScope a scope that NgAndroid will recognize you need to add `@NgScope(name="Login")` to the class declaration.

{{< highlight java>}}
@NgScope(name="Login")
public class LoginScope {

}
{{< /highlight>}}

Now create a User model class for your data.

{{< highlight java>}}
public class User {

    private String username, password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
{{< /highlight>}}

Add a user field to `LoginScope` annotated with `@NgModel`.

{{< highlight java>}}
@NgScope(name="Login")
public class LoginScope {
    @NgModel
    User user;
}
{{< /highlight>}}

By annotating a class with the `@NgModel` annotation you are telling NgAndroid to create a subclass of that class that will handle all of the view bindings.
It also means that NgAndroid will inject that field. There is no need to instantiate it.

Let's create some bindings. Open up activity_login.xml again and add `xmlns:x="http://schemas.android.com/apk/res-auto"`
below `xmlns:android="http://schemas.android.com/apk/res/android"` in your `RelativeLayout`.
Then declare the scope of the xml file by adding `x:ngScope="Login"` as well. You should have something that looks like this.

{{< highlight xml>}}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:x="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="60dp"
    x:ngScope="Login"
    tools:ignore="MissingPrefix" >

    ...


</RelativeLayout>
{{< /highlight>}}

I also like to add `tools:ignore="MissingPrefix"` to the root layout.
Otherwise Android Studio will complain about adding non android prefix attributes to standard views.

You can now reference any methods or models declared in your scope directly in xml attributes.

Now bind your `User` to your username and password `EditText` views by adding `x:ngModel="user.username"` and `x:ngModel="user.password"` respectively.
Also, make your submit button only be enabled if the lengths of the username and password are both greater than 6 by adding
`x:ngDisabled="user.username.length() &lt; 6 || user.password.length() &lt; 6"` to your submit button. All together it should look
like this.

{{< highlight xml>}}
<EditText
    android:id="@+id/username"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_above="@+id/password"
    android:inputType="text"
    x:ngModel="user.username" />

<EditText
    android:id="@+id/password"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:inputType="textPassword"
    android:layout_above="@+id/submit"
    x:ngModel="user.password"/>

<Button
    android:id="@+id/submit"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="Submit"
    android:layout_centerInParent="true"
    x:ngDisabled="user.username.length() &lt; 6 || user.password.length() &lt; 6"/>
{{< /highlight>}}

The `&lt;` is the symbol for `<` in an xml attribute. `&gt;` is the symbol for `>`.

Now run your app.

![App screenshot](/images/not_disabled.png)

But wait, why isn't the submit button disabled until after you start typing?
If you check your logs you'll see `Unable to get initial value for view 'submit' because of null pointer`
That is just saying that when NgAndroid tried finding whether or not the submit button should be disabled something was `null`.
Initializing `username` and `password` will fix that. `private String username = "", password = "";`.
If you run it again it'll be disabled like expected.

![App screenshot](/images/disabled.png)

If you type in 6 or more characters in both the username and the password fields than you will see that the submit button is enabled.

In order to do something when the submit button is clicked create a method in your scope.

{{< highlight java>}}
void onSubmit(Context context) {
    Toast.makeText(context, user.getUsername() + " : " +user.getPassword(), Toast.LENGTH_SHORT ).show();
}
{{< /highlight>}}

Then create an event binding in your layout file.

{{< highlight xml>}}
<Button
    android:id="@+id/submit"
    ...
    x:ngClick="onSubmit($view.context)"/>
{{< /highlight>}}

You'll notice the `$view` syntax. `$` is a symbol that allows you to reference predefined NgAndroid variables.
$view is a reference to the view that will be bound to the click event. You can use `.context` as a shortcut to
`.getContext()`.

Now you have a working login screen. If you want to add an animation to that it's extremely easy.
Add the `ProgressBar` to your layout with an `ngInvisible` attribute to make it only visible while
your network call is active.

{{< highlight xml>}}
<ProgressBar
    android:id="@+id/progress"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_centerInParent="true"
    x:ngInvisible="!call.active"/>
{{< /highlight>}}

Add `x:ngInvisible="call.active"` to your submit button to hide it when your call is active.

Finally add your `NetworkCall` model to your `LoginScope` and set it as active.

{{< highlight java>}}
public class LoginScope {
    //...
    @NgModel
    NetworkCall call;

    void onSubmit(Context context) {
        call.setActive(true);
        //...
    }
}
{{< /highlight>}}

It's that easy.

![Working login](/images/working_login.gif)
