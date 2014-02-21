title: ListView滑动位置精准记忆
date: 2014-02-21
tags: Android ListView
---
需求场景：
一个ListView页面，滑动阅读到某一位置，然后退出页面，下次再进入页面的时候，想要直接滑动到上次阅读的位置。

方案1：
页面退出的时候，ListView.getFirstVisiblePosition()来获取当前可见的第一个Item的position并记录，下次再进入页面的时候通过ListView.setSelection(int position)把ListView直接滑动到记忆的position。

此方案记忆的ListView的位置不够精准，因为position指定的是ListView的Item的index，setSelection(int pos)只能把index为pos的item作为第一个可见的item来显示，所以item总是顶头显示的，不会显示滑出屏幕一半的item，所以ListView的位置只能定位到某个item的开始位置，并不精准。

方案1的升级：
在方案1的基础上，再记录FirstVisiblePosition item的top/bottom等位置参数，然后恢复的时候ListView再scrollTo一下。

此方案仍然不行，ListView的scrollTo没效果。

终极方案：
退出页面的时候：
{% code %}
Parcelable listState = listView.onSaveInstanceState();
{% endcode %}
记住listState对象；
再次进入页面的时候：
{% code %}
listView.onRestoreInstanceState(listState);
{% endcode %}

记忆的位置分毫不差。

要注意：listView的状态记忆后，还要保证其数据在两次进入页面时的一致性；
另：ListView的header会影响其状态对象，不过这个是小问题。
