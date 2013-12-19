title: Shape + Selector: Make a Shape as one item of the Selector
date: 2013-11-16
tags: Android
---
Generally, I use a selector to select pictures or colors to render the normal and the pressed background for View. 

And use a Shape to draw a fixed background with border, round, gradient and other UI effect for View.

What if you wanna use them both for one View?

Make a Shape as one item of the Selector.

Selector selects background from Shape list.

Below is a sample.

{% code %}
<?xml version=”1.0″ encoding=”UTF-8″?>  
<selector xmlns:android=”http://schemas.android.com/apk/res/android”>  
    <item android:state_pressed=”true”>  
            <shape android:shape=”rectangle”>  
                  <solid android:color=”@color/background_pressed” />  
                  <stroke android:width=”@dimen/gap_2″ android:color=”@color/linecolor” />  
                  <padding android:bottom=”@dimen/gap_2″ android:left=”@dimen/gap_2″                                     
                  android:right=”@dimen/gap_2″ android:top=”@dimen/gap_2″ />  
            </shape>  
     </item>  
  
     <item>  
         <shape android:shape=”rectangle”>  
              <solid android:color=”@color/background_normal” />  
              <stroke android:width=”@dimen/gap_2″ android:color=”@color/linecolor” />  
              <padding android:bottom=”@dimen/gap_2″ android:left=”@dimen/gap_2″                                          
              android:right=”@dimen/gap_2″ android:top=”@dimen/gap_2″ />  
         </shape>  
     </item>  
</selector>  
{% endcode %}

<!-- more -->

