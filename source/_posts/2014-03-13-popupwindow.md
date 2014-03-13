title: 使用PopupWindow
date: 2014-03-12
tags: PopupWindow
---
 PopupWindow可以用来实现弹出任意位置的菜单，比Context Menu和Option Menu灵活性更高。
 Android中弹出一个PopupWindow基本有两个方法：
 
 {% code %}
//Display the content view in a popup window anchored to the bottom-left corner of the anchor view.
public void showAsDropDown(View anchor, int xoff, int yoff);
{% endcode %}
这个方法是弹出的窗口在anchor view的bottom-left,一般Android应用的菜单都在右上，实现思路就是以app中右上的view作为anchor，然后用屏幕宽度减去popup window的宽度作为xoff.
 
 还有一个方法：
{% code %}
 public void showAtLocation(IBinder token, int gravity, int x, int y);
{% endcode %}
 
再来说如何实现一个PopupWindow，步骤基本如下，下面代码时extends了PopupWindow的子类的实现：

<!-- more --> 

{% code %}
//inflate一个content view并设置给PopupWindow
mContentView = (View) mInflater.inflate(R.layout.popup_window, null);
setContentView(mContentView);

//给组件定义事件
mQuitView = (TextView) mContentView.findViewById(R.id.popup_window_quit);
mQuitView.setOnClickListener(this);
... ...

//设置popup window的背景，如果设置了非null，PopupWindow内部会将其包起来作为root view展示
//如果设置了null，则setOutsideTouchable(true)不起作用
setBackgroundDrawable(...);

//设置在popup window之外点击dismiss window
setOutsideTouchable(true);

//如果设置为true, popup window打开的话，系统menu键就不响应了，back键还可以响应，因为PopupWindow内部接收了back键但没关menu键
//所以假如要用menu键来控制popup window的打开和关闭的话，就需要额外的实现，后面会讲。
setFocusable(true);

//setWidth和setHeight是必须的，不然window没尺寸，但又不想hardcoded尺寸的话怎么办？用如下的方法
mContentView.measure(View.MeasureSpec.UNSPECIFIED, View.MeasureSpec.UNSPECIFIED);
setWidth(mContentView.getMeasuredWidth());
setHeight(mContentView.getMeasuredHeight());
{% endcode %}
 
 再来讲刚才说到的用系统menu来控制popup window的方法:
1, 自定义自己的content view;
2, 在自定义的content view中接收menu key event(dispatchKeyEvent);
到此为止，如果popupWindow.setFocusable(true)，则自定义content view的dispatchKeyEvent不会被执行，必须加上：
3, setFocusableInTouchMode(true);
 
详细代码：

<script src="https://gist.github.com/Viyu/9521561.js"></script>
