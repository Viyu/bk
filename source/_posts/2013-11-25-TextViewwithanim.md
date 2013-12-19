title: Implement a TextView with an animation in its left side
date: 2013-11-25
tags: Android
---
In my case, I want to write a TextView with an animation in its left side.

ImageView + TextView could work but it’s not frugal enough.

TextView with drawableLeft would be the best option.

{% code %}
<TextView  
     … …  
     android:drawablePadding=”10dip”  
     android:drawableLeft=”@drawable/loadingicon” />  
{% endcode %}
     
But above implementation just show a icon in the left of TextView, not an animation.

How to show an animation instead of a icon?

<!-- more -->

Assume you have 4 pictures which would be an animation if display them one after one. Then define animation xml in res/anim folder in your project.

loading_animation.xml:

{% code %}
<animation-list xmlns:android=”http://schemas.android.com/apk/res/android”  
      android:oneshot=”false” >  
      <item  
           android:drawable=”@drawable/loading_1″  
           android:duration=”500″>  
       </item>  
       <item  
            android:drawable=”@drawable/loading_2″  
            android:duration=”500″>  
        </item>  
        <item  
             android:drawable=”@drawable/loading_3″  
             android:duration=”500″>  
        </item>  
</animation-list>  
{% endcode %}

“android:oneshot” determines whether or not play the animation just once.
“android:duration” determines the duration of pictures switching.

Then in layout xml, use above animation like this:

{% code %}
<TextView  
      … …  
      android:drawablePadding=”10dip”  
      android:drawableLeft=”@anim/loading_animation” />  
{% endcode %}

Bingo? no, no, no, the animation can not switching yet.

You need to do more.

In java/Activity code, you must get the animation drawable and start it.

{% code %}
Drawable[] draws = locateAreaTextView.getCompoundDrawables();  
if (draws != null && draws.length > 0 && draws[0] instanceof AnimationDrawable) {  
       loadingAnimation = (AnimationDrawable) draws[0];  
       loadingAnimation.start();  
}  
{% endcode %}

In above, we get index 0 of the drawable array since drawableLeft is in 0 position of the array.

Now, the animation works. But there is one more trap.

If you put above codes in onCreate() of Activity, draws will be a [null, null, null, null] array.

You must put it in onResumse() of Activity.
