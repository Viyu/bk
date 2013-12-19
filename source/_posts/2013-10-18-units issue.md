title: Units Problem: How to read text size as custom attr from xml and set it to TextView in java code
date: 2013-10-18
tags: Android
---
Here is this topic’s background:

I defined a custom View which extends FrameLayout and contains a TextView, calledMyView here. And I defined custom attribute “myviewtextsize” in attrs.xml for MyView so that clients can set different text size in layout xml for the TextView of MyView.

So far, clients code can set text size like this:

{% code %}
<MyView  
     android:…  
     …
     my:myviewtextsize=”@dimen/textsize_24″  
     …  
/> 
{% endcode %}

The problem is: how to read the client’s text size number and set it to the TextVew of MyView?

In MyView.java,

{% code %}
float textSize = typedArray.getDimension(R.MyView_myviewtextsize, -1);  
  
this.textView.setTextSize(textSize).  
{% endcode %}

Above code goes wrong. The text size is bigger than it’s supposed to.

Why? It is mixed units problem.

The default method setTextSize(float) assumes you’re inputting sp units (scaled pixels), while the typedArray.getDimension() method returns an exact pixel size.

It can be fixed this by using the alternate <b>setTextSize(TypedValue, float)</b>, like below:
 
{% code %}
this.textView.setTextSize(TypedValue.COMPLEX_UNIT_PX, size);  
{% endcode %}

This will make sure you’re working with the same units.


<!-- more -->

