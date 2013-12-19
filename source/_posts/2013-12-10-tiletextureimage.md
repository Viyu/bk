title: How to tile small texture image onto page as its background
date: 2013-12-10
tags: Android
---
You don’t need to set a big size image as the background of pages if the image is texture or uniform color.

How to tile small texture image onto big page/area as its background?

First, define drawable xml in res/drawable,

bg_texture_image.xml:

{% code %}
<?xml version=”1.0″ encoding=”utf-8″?>  
<bitmap xmlns:android=”http://schemas.android.com/apk/res/android”  
      android:src=”@drawable/bg_image”  
      android:tileMode=”repeat” />  
{% endcode %}
      
Second, use it as the background,

{% code %}
android:background=”@drawable/bg_texture_image”  
{% endcode %}
