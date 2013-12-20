title: Android初学笔记
date: 2012-12-10
tags: Android
---
Eclipse 【ADT】 源

https://dl-ssl.google.com/android/eclipse

-------------------------------------------------------------------------

Notice that no matter what scenario causes the activity to stop, the system always calls onPause() before calling onStop().

Although the onPause() method is called before onStop(), you should use onStop() to perform larger, more CPU intensive shut-down operations, such as writing information to a database.

the system calls onStart() both when it creates your activity and when it restarts the activity from the stopped state.

Resumed

In this state, the activity is in the foreground and the user can interact with it. (Also sometimes referred to as the "running" state.)

Paused

In this state, the activity is partially obscured by another activity—the other activity that's in the foreground is semi-transparent or doesn't cover the entire screen. The paused activity does not receive user input and cannot execute any code. 

Stopped

In this state, the activity is completely hidden and not visible to the user; it is considered to be in the background. While stopped, the activity instance and all its state information such as member variables is retained, but it cannot execute any code.

<!-- more -->

----------------------------------------------------------------------

//保证当前的版本可以执行某段代码

{% code %}
if ( Build.VERSION.SDK_INT >=Build.VERSION_CODES.HONEYCOMB ) {   

}
{% endcode %}
------------------------------------------------------------------------

onCreate<-->onDestroy，执行一次；

onStart<-->onStop，循环；

onResume<-->onPause，循环；

----------------------------------------------------------------------

In order for the Android system to restore the state of the views in your activity, each view must have a unique ID, supplied by the android:id attribute.

----------------------------------------------------------------------

As the system begins to stop your activity, it calls onSaveInstanceState() (1) so you can specify additional state data you'd like to save in case the Activity instance must be recreated. If the activity is destroyed and the same instance must be recreated, the system passes the state data defined at (1) to both the onCreate() method (2) and the onRestoreInstanceState() method (3).

When your activity is recreated after it was previously destroyed, you can recover your saved state from the Bundle that the system passes your activity. Both the onCreate() and onRestoreInstanceState() callback methods receive the same Bundle that containes the instance state information.

Because the onCreate() method is called whether the system is creating a new instance of your activity or recreating a previous one, you must check whether the state Bundle is null before you attempt to read it. If it is null, then the system is creating a new instance of the activity, instead of restoring a previous one that was destroyed.

Instead of restoring the state during onCreate() you may choose to implement onRestoreInstanceState(), which the system calls after the onStart() method. The system calls onRestoreInstanceState() only if there is a saved state to restore, so you do not need to check whether the Bundle is null.

------------------------------------------------------------------

从.java中和从.xml中load resource

// Get a string resource from your app's Resources

{% code %}
String hello =getResources().getString(R.string.hello_world);
{% endcode %}

// Or supply a string resource to a method that requires a string

{% code %}
TextView textView =newTextView(this);

textView.setText(R.string.hello_world);
{% endcode %}

{% code %}
<TextView

    android:layout_width="wrap_content"

    android:layout_height="wrap_content"

    android:text="@string/hello_world" />
{% endcode %}

------------------------------------------------------------------

【Supporting different screens】

There are four generalized sizes: small, normal, large, xlarge

And four generalized densities: low (ldpi), medium (mdpi), high (hdpi), extra high (xhdpi)

Also be aware that the screens orientation (landscape or portrait) is considered a variation of screen size, so many apps should revise the layout to optimize the user experience in each orientation

Android automatically scales your layout in order to properly fit the screen. Thus, your layouts for different screen sizes don't need to worry about the absolute size of UI elements but instead focus on the layout structure that affects the user experience (such as the size or position of important views relative to sibling views).

By default, the layout/main.xml file is used for portrait orientation.

If you want a provide a special layout for landscape, including while on large screens, then you need to use both the large and land qualifier:

MyProject/ 

        res/ 

                layout/main.xml                                  # default (portrait)  

                layout-land/main.xml            # landscape

                layout-large/main.xml            # large (portrait) 

                layout-large-land/main.xml       # large landscape



xhdpi: 2.0

hdpi: 1.5

mdpi: 1.0 (baseline)

ldpi: 0.75

This means that if you generate a 200x200 image for xhdpi devices, you should generate the same  resource in 150x150 for hdpi, 100x100 for mdpi, and 75x75 for ldpi devices.

--------------------------------------------------------------

如果用Android SDK Manager下载了Support lib，则无论创建mini SDK version为多高的project，android-support-v4.jar都会被自动拷贝到lib下，并且关联起来。

The Android Support Library provides a JAR file with an API library that allow you to use some of the more recent Android APIs in your app while running on earlier versions of Android.

If you decide for other reasons that the minimum API level your app requires is 11 or higher, you don't need to use the Support Library and can instead use the framework's built in Fragment class and related APIs. 

---------------------------------------------------------------

activity与其对应的layout/*.xml并不需要名字对应

而是，得在*.xml中添加的元素中至少有一个@+id，这样才会让R注册这个*.xml作为一个layout.

------------------------------------------------------------------

Your app's internal storage directory is specified by your app's package name in a special location of the Android file system.

Tip: Although apps are installed onto the internal storage by default, you can specify the android:installLocation attribute in your manifest so your app may be installed on external storage. Users appreciate this option when the APK size is very large and they have an external storage space that's larger than the internal storage. For more information

Caution: Currently, all apps have the ability to read the external storage without a special permission. However, this will change in a future release. If your app needs to read the external storage (but not write to it), then you will need to declare the READ_EXTERNAL_STORAGE permission. To ensure that your app continues to work as expected, you should declare this permission now, before the change takes effect.

{% code %}
 <manifest ...>

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

    ...

</manifest>
{% endcode %}

However, if your app uses the WRITE_EXTERNAL_STORAGE permission, then it implicitly has permission to read the external storage as well.

You don’t need any permissions to save files on the internal storage. Your application always has permission to read and write files in its internal storage directory.

-----------------------------------------------------------------------

Remember that getExternalFilesDir()creates a directory inside a directory that is deleted when the user uninstalls your app. If the files you're saving should remain available after the user uninstalls your app—such as when your app is a camera and the user will want to keep the photos—you should instead use getExternalStoragePublicDirectory().

Note: When the user uninstalls your app, the Android system deletes the following:

 •All files you saved on internal storage

 •All files you saved on external storage using getExternalFilesDir().

However, you should manually delete all cached files created with getCacheDir() on a regular basis and also regularly delete other files you no longer need.

------------------------------------------------------------------------

Just like files that you save on the device's internal storage, Android stores your database in private disk space that's associated application. Your data is secure, because by default this area is not accessible to other applications.

-------------------------------------------------------------------------

Eclispe 【Git】 plug-in:

http://download.eclipse.org/egit/updates

http://www.eclipse.org/egit/download/

---------------------------------------------------------------------------

MPC is included in all of the packages available from the Eclipse download page (except the Classic Package).        

-------------------------------------------------------------------------

【subversion SVN】 eclipse plugin subeclipse source:

http://subclipse.tigris.org/update_1.0.x

------------------------------------------------------------------------

【Activities组成一个Stack】

不同应用程序之间的Activities共同组成一个Stack的Task，call的过程是1.onCreate-1.onStart-1.onResume-1.onPause-2.onCreate-2.onStart-2.onResume-1.onStop-2.onPause-3.onStart-... ...  back的过程是3.onPause-2.onRestart-2.onStart-2.onResume-3.onStop-3.onDestory..

可见back的过程中出栈的activity是被destory掉了；默认在call的过程中不会destory掉上一个activity，但如果调用了fininsh，则在back的过程中由于此activity已经被destory了，不会出现在back链上。

假如一个Activity的manifest定义其theme是diaolog，则状态链条中会少一个onStop，只是onPause.

--------------------------------------------------------------------

在layout.xml中和在.java中声明组件的意义

在layout.xml中声明的组件可以在任何地方被R.layout.id调用到；

而在.java中声明的组件是.java私有的。

--------------------------------------------------------------------

如果在setContentView为layout.xml之前调用findViewById，则返回null

----------------------------------------------------------------------

getCacheDir()方法用于获取/data/data/<package name>/cache目录
getFilesDir()方法用于获取/data/data/<package name>/files目录

-----------------------------------------------------------------------

【探测键盘是否呼出的一种不靠谱方法】

{% code %}
android:windowSoftInputMode="adjustResize"
{% endcode %}

{% code %}
final View root = findViewById(R.id.editnotelayout);

root.getViewTreeObserver().addOnGlobalLayoutListener(

new OnGlobalLayoutListener() {

@Override

public void onGlobalLayout() {

Rect r = new Rect();

root.getWindowVisibleDisplayFrame(r);

int heightDiff = root.getRootView().getHeight() - (r.bottom - r.top);

if (heightDiff > 100) {

System.out.println("1: ....Keyboard show");

} else {

System.out.println("1: ....Keyboard hidden");

}

}
});
{% endcode %}