title: Android客户端的图形化拖放操作的设计实现
date: 2013-12-23
tags: Android
---
<b>为什么要拖放？</b>
拖放在某些UI交互中可以简化用户操作。

<b>拖放的步骤包括哪些？</b>
“Drag and Drop”，拖放，顾名思义，总共就分三步：
1，	开始拖起来；
2，	正在拖；
3，	放下，进行操作；
在这三步里，贯穿的是数据的传输，从拖的地方传输到放的地方。

现在，我们就以一个简单的拖放删除笔记的App来讲解上面三个步骤都怎么实现的。
App见截图，拽住上面的笔记，然后拖至底下的垃圾桶然后放开，笔记就被删除了。

<img src="http://a.hiphotos.bdimg.com/album/s%3D550%3Bq%3D90%3Bc%3Dxiangce%2C100%2C100/sign=4b2bee88b54543a9f11bfac92e2cfbbf/eac4b74543a9822637ce1b5b8882b9014a90ebbb.jpg?referer=83c60f9d60d9f2d3790611df5559&x=.jpg"/>
 
<b>1，开始拖拽：</b>
开始拖拽要解决三个问题：
1， 什么时候开始？如何开始？
一般是在用户长按一个组件的时候，我们开始拖拽，所以给要被拖拽的View定义它的OnLongClickListener，在其中调用View.startDrag()方法就可以了。
startDrag()方法的定义：

{% code %}
boolean android.view.View.startDrag(
		ClipData data, DragShadowBuilder shadowBuilder, Object myLocalState, int flags)
{% endcode %}

第一个参数ClipData data是拖拽的对象，下面第2条来讲解；第二个参数DragShadowBuilder shadowBuilder是拖拽时的样子，下面的第3条来讲解。

<!-- more --> 

2， ClipData：拖拽的数据实现？
数据定义通过ClipData和ClipData.Item来定义，看一下它们的定义：

{% code %}
/**
* Added in API level 11
* Create a new clip.
* Parameters
* label  Label to show to the user describing this clip.
* mimeTypes  An array of MIME types this data is available as. 
* item  The contents of the first item in the clip.
*/
public ClipData (CharSequence label, String[] mimeTypes, ClipData.Item item) 
{% endcode %}

由此可见，构造一个ClipData对象，作为拖拽的数据对象，它的构造需要三个参数：
1，label 给一个标签；
2，mimeTypes
3，一个ClipData.Item对象
再来看ClipData.Item的定义：
{% code %}
/**
* The types than an individual item can currently contain are: 
* Text: a basic string of text. This is actually a CharSequence, so it can be formatted text supported 
* by corresponding Android built-in style spans. (Custom application spans are not supported and will be 
* stripped when transporting through the clipboard.) 
* Intent: an arbitrary Intent object. A typical use is the shortcut to create when pasting a clipped item 
* on to the home screen. 
* Uri: a URI reference. This may be any URI (such as an http: URI representing a bookmark), 
* however it is often a content: URI. Using content provider references as clips like this allows 
* an application to share complex or large clips through the standard content provider facilities. 
*/
android.content.ClipData.Item 
{% endcode %}

ClipData.item有三类构造方法：
{% code %}
/**
* Create an Item consisting of a single block of (possibly styled) text.
*/
public Item(CharSequence text) {
}

/**
* Create an Item consisting of a single block of (possibly styled) text,
* with an alternative HTML formatted representation.  You must
* supply a plain text representation in addition to HTML text; coercion
* will not be done from HTML formated text into plain text.
*/
public Item(CharSequence text, String htmlText) {
}

/**
* Create an Item consisting of an arbitrary Intent.
*/
public Item(Intent intent) {
}

/**
* Create an Item consisting of an arbitrary URI.
*/
public Item(Uri uri) {
}
{% endcode %}

就是分别可以用Text, Intent和Uri来构造。

3， DragShadowBuilder: 拖拽时的样子：

DragShadowBuilder可以用来定义拖拽时的样子，可以直接用其构造方法DragShadowBuilder(View view)来传递一个View做为样子，也可以自己扩展DragShadowBuilder类。
看一下DragShadowBuilder(View view)的构造方法说明：
{% code %}
/**
* Added in API level 11
* Constructs a shadow image builder based on a View. By default, the resulting drag shadow will have the same appearance and 
* dimensions as the View, with the touch point over the center of the View.
* Parameters
* view  A View. Any View in scope can be used.
*/
public View.DragShadowBuilder (View view) 
  
{% endcode %}

OK，拖拽的第一阶段：开始拖拽就讲完了，看一下完整代码：
{% code %}
//给要被拖拽的View mNote_1添加长按事件
mNote_1.setOnLongClickListener(new View.OnLongClickListener() {
  @Override
  public boolean onLongClick(View v) {
  //在我的代码实现中，给mNote_1设置了一个Text作为其Tag，这里取出来用来构造Item
    ClipData.Item item = new ClipData.Item((CharSequence)v.getTag());
    String[] mimeTypes = {ClipDescription.MIMETYPE_TEXT_PLAIN};
    //构造一个drag data
    ClipData dragData = new ClipData(v.getTag().toString(), mimeTypes, item);
    //构造一个shadow
    View.DragShadowBuilder myShadow = new DragShadowBuilder(mNote_1);
    //发起拖拽
    v.startDrag(dragData, myShadow, null, 0);
    return true;
  }
});
{% endcode %}

<b>2，正在拖拽 </b>

只要给View添加一个OnDragListener，就可以得到整个拖拽过程中的所有回调事件。
OnDragListener的方法onDrag定义如下：
{% code %}
/**
 * Added in API level 11 
 * Called when a drag event is dispatched to a view. This allows listeners to get a chance to override base View behavior.
 * Parameters
 * v  The View that received the drag event. 
 * event  The DragEvent object for the drag event. 
 * Returns
 * true if the drag event was handled successfully, or false if the drag event was not handled. Note that false will trigger the View to call its onDragEvent() handler. 
 */
public abstract boolean onDrag (View v, DragEvent event) 
{% endcode %}

DragEvent定义了所有拖拽中的事件：

<b>ACTION_DRAG_STARTED</b>
只在应用程序调用startDrag()方法，并且获得了拖拽影子后，View对象的拖拽事件监听器才接收这种事件操作。
<b>ACTION_DRAG_ENTERED</b>
当拖拽影子刚进入View对象的边框时，View对象的拖拽事件监听器会接收这种事件操作类型。
<b>ACTION_DRAG_LOCATION</b>
在View对象收到一个ACTION_DRAG_ENTERED事件之后，并且拖拽影子依然还在这个对象的边框之内时，这个View对象的拖拽事件监听器会接收这种事件操作类型
<b>ACTION_DRAG_EXITED</b>
View对象收到一个ACTION_DRAG_ENTERED和至少一个ACTION_DRAG_LOCATION事件之后，这个对象的事件监听器会接受这种操作类型。
<b>ACTION_DROP</b>
当用户在一个View对象之上释放了拖拽影子，这个对象的拖拽事件监听器就会收到这种操作类型。如果这个监听器在响应ACTION_DRAG_STARTED拖拽事件中返回了true，那么这种操作类型只会发送给一个View对象。如果用户在没有被注册监听器的View对象上释放了拖拽影子，或者用户没有在当前布局的任何部分释放操作影子，这个操作类型就不会被发送。如果View对象成功的处理放下事件，监听器要返回true，否则应该返回false。
<b>ACTION_DRAG_ENDED</b>
当系统结束拖拽操作时，View对象拖拽监听器会接收这种事件操作类型。这种操作类型之前不一定是ACTION_DROP事件。如果系统发送了一个ACTION_DROP事件，那么接收ACTION_DRAG_ENDED操作类型不意味着放下操作成功了。监听器必须调用getResult()方法来获得响应ACTION_DROP事件中的返回值。如果ACTION_DROP事件没有被发送，那么getResult()会返回false。

<b>3，放下的操作 </b>
放下就是响应上面说的ACTION_DRAG_ENDED事件。在具体应用中，要明确我们是在哪里放下才采取动作。本文的例子中，就是把作为笔记的TextView拖拽到一个垃圾桶标志的ImageView上放下时，采取删除该TextView的操作。
只要给垃圾桶ImageView添加OnDragListener并且监听到ACTION_DRAG_ENDED事件时采取操作就行了。
{% code %}
trashView.setOnDragListener( new OnDragListener(){
  @Override
    public boolean onDrag(View v,  DragEvent event){
      switch(event.getAction())                   
      {
      case DragEvent.ACTION_DROP:
      Log.d("trash view", "ACTION_DROP event");
      ClipData data = event.getClipData();
      CharSequence noteName = data.getItemAt(0).getText();
      if(noteName.equals(TAG_NOTE_1)) {
        mNote_1.setVisibility(View.GONE);
        Toast.makeText(MainActivity.this, "Note 1 has been removed.", Toast.LENGTH_SHORT).show();
      } else if(noteName.equals(TAG_NOTE_2)) {
        Toast.makeText(MainActivity.this, "Note 2 has been removed.", Toast.LENGTH_SHORT).show();
        mNote_2.setVisibility(View.GONE);
      }
      break;
      default: 
      break;
      }
    return true;
  }
});
{% endcode %}

--------------------------------代码---------------------------------------
MainActivity.java

{% code %}

public class MainActivity extends Activity{
	private TextView mNote_1 = null;
	private TextView mNote_2 = null;
	
	private ImageView trashView = null;
	
   private static final String TAG_NOTE_1 = "Note_1";
   private static final String TAG_NOTE_2 = "Note_2";

   @Override
   public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);

      trashView = (ImageView)findViewById(R.id.main_trash);
      
      mNote_1 = (TextView)findViewById(R.id.main_note_1);
      mNote_1.setTag(TAG_NOTE_1);
      mNote_1.setOnLongClickListener(new View.OnLongClickListener() {
         @Override
         public boolean onLongClick(View v) {
            ClipData.Item item = new ClipData.Item((CharSequence)v.getTag());
            String[] mimeTypes = {ClipDescription.MIMETYPE_TEXT_PLAIN};
            ClipData dragData = new ClipData(v.getTag().toString(), 
            mimeTypes, item);
            View.DragShadowBuilder myShadow = new DragShadowBuilder(mNote_1);
            v.startDrag(dragData, myShadow, null, 0);
            return true;
         }
      });
      
      mNote_2 = (TextView)findViewById(R.id.main_note_2);
      mNote_2.setTag(TAG_NOTE_2);
      mNote_2.setOnLongClickListener(new View.OnLongClickListener() {
         @Override
         public boolean onLongClick(View v) {
            ClipData.Item item = new ClipData.Item((CharSequence)v.getTag());
            String[] mimeTypes = {ClipDescription.MIMETYPE_TEXT_PLAIN};
            ClipData dragData = new ClipData(v.getTag().toString(), mimeTypes, item);
            View.DragShadowBuilder myShadow = new DragShadowBuilder(mNote_2);
            v.startDrag(dragData, myShadow, null, 0);
            return true;
         }
      });
      
      trashView.setOnDragListener( new OnDragListener(){
         @Override
         public boolean onDrag(View v,  DragEvent event){
         switch(event.getAction())                   
         {
            case DragEvent.ACTION_DROP:
               Log.d("trash view", "ACTION_DROP event");
               ClipData data = event.getClipData();
               CharSequence noteName = data.getItemAt(0).getText();
               if(noteName.equals(TAG_NOTE_1)) {
            	   mNote_1.setVisibility(View.GONE);
            	   Toast.makeText(MainActivity.this, "Note 1 has been removed.", Toast.LENGTH_SHORT).show();
               } else if(noteName.equals(TAG_NOTE_2)) {
            	   Toast.makeText(MainActivity.this, "Note 2 has been removed.", Toast.LENGTH_SHORT).show();
            	   mNote_2.setVisibility(View.GONE);
               }
               break;
            default: break;
            }
            return true;
         }
      });
   }
}
{% endcode %}

activity_main.xml
{% code %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/main_note_1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_marginTop="20dip"
		android:layout_centerHorizontal="true"
		android:background="@color/note_bg"
		android:text="@string/note1"
		android:textSize="24sp"
		android:padding="10dip"  />
    
    <TextView
        android:id="@+id/main_note_2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
		android:layout_below="@id/main_note_1"
        android:layout_marginTop="20dip"
		android:layout_centerHorizontal="true"
		android:background="@color/note_bg"
		android:text="@string/note2"
		android:textSize="24sp"
		android:padding="10dip"  />

    <ImageView
        android:id="@+id/main_trash"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
		android:layout_centerHorizontal="true"
		android:layout_marginBottom="20dip"
        android:src="@drawable/trash" />

</RelativeLayout>
{% endcode %}
