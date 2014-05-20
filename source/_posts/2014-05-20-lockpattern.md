title: Android图案锁实现
date: 2014-05-20
tags: Android LockPattern
---
<img src="http://g.hiphotos.bdimg.com/album/s%3D550%3Bq%3D90%3Bc%3Dxiangce%2C100%2C100/sign=94f295ef13dfa9ecf92e561252eb863e/562c11dfa9ec8a130506a013f503918fa0ecc021.jpg?referer=bfd815e96259252dfa00293420ff&x=.jpg"/>

很多安全要求高的App都会有个图案/手势/LockPattern解锁的模块，比如支付宝钱包等。

Android系统本身就有这个东西，叫LockPatternView，所以在自己的App中不用自己实现，但也不能直接调用，因为Android的LockPatternView不是给App用的，
得从\AndroidSdk\sources\android-19\com\android\internal\widget下拷贝出来改改再用。
具体步骤：
1， 拷贝两个文件LockPatternView和LockPatternUtils；
2， LockPatternView.java中报错的地方，都是找不到string/drawable/attr等，按照名字去sdk相应目录下拷出这些资源来或者新建；
3， 删掉LockPatternUtils.java中所有代码，除了LockPatternView中要调用的两个方法stringToPattern和patternToString；

现在这个LockPatternView就可以用了，在自己的布局中使用这个View，并且给其setOnPatternListener(...).
OnPatternListener有个方法: 
{% code %}
public void onPatternDetected(List<Cell> pattern);
{% endcode %}
pattern就是九宫格绘制出来的点的一个list.

要真正实现一个设置图案锁\解锁的模块，还有一些工作要自己做，利用上面的LockPatternView生成的结果，去两次匹配、存储、解锁时再匹配等。

<!-- more --> 

<a href="http://mosthink.net/LockPattern/">这个project</a>实现了这些。 
 
