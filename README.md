# AndroidBugs

原文：[Android Bug总结](http://www.jianshu.com/p/e0b46e8c4580)

1、Android Studio 如何提交代码到 github:
http://blog.csdn.net/u011068702/article/details/49273231#userconsent#

2、Eclipse 导入 android 项目包xml报错未生成 R 文件:
http://jingyan.baidu.com/article/c910274be7536acd361d2dca.html

3、解决 WebView 和 JavaScript 调用混淆导致功能失效:
在 eclipse 的 proguard.cfg 中加入“保持该类下的所有方法不被混淆”

-keep public class com.example.web_01.WebHost
{
    public <methods>;
}
4、eclipse 查看 .class 文件方法:
【1】在项目 libs 目录下新建File: android-suppory-v4.jar.properties
【2】在android-support-v4.jar.properties 里加入 src = E:\Android SDK\sdk\extras\android\support\v4\src ( SDK中对应版本的src位置 )
【3】重启 eclipse

5、genymotion 使用问题
【1】重启ADB方法：在 cmd 的 D:\android\adt-bundle-windows-x86_64-20140702\sdk\platform-tools目录下输入 adb kill-server 和 adb start-server
【2】eclipse 无法识别 genymotion 原因: genymotion 的 API 版本低于项目的 API 版本

6、解决 ADB Server didn't ACK:
打开任务管理器，点击进程，把里面的手机助手进程都结束了。原因是手机助手占用了ADB的5037端口，可在 platform-tools 下用 netstat -ano | findstr "5037" 查看。

7、解决 Unable to resolve target 'android-17' 报错
修改project.properties：把Project target.target=android-17改成Project target.target=android-21 再 clean

8、v4包找不到:
当引用的library中的jar包和本项目的jar包不一致，会导致v4包找不到。可选择删除其中一个，使两个项目中的jar一样

9、自定义Application问题:
自定义Application一定要注册，且通过getApplication()方法获得。

10、SQLite使用问题:
java.lang.IllegalStateException: Couldn't read row 0, col -1 from CursorWindow. Make sure the Cursor is initialized correctly before accessing data from it.
字段不一致。进行修改了数据库的操作，一定要先卸载原应用，再重新安装。为了字段排序，应采用 TreeMap 代替 HashMap 。

11、Layout动态设置高度:

//必须用android.view.ViewGroup.LayoutParams重新设置高度
android.view.ViewGroup.LayoutParams pp = view.getLayoutParams();
pp.height = 200;
view.setLayoutParams(pp);
12、为onClickListener 添加判断标志:
可使用view.setTag()

13、Android Sudio 导入 Eclipse 项目:
http://www.open-open.com/lib/view/open1421580998718.html

14、解决 Android 应用方法数不能超过 65K 的问题 ( 摘自安卓巴士Android开发者门户 ):

作为一名Android开发者，相信你对Android方法数不能超过65K的限制应该有所耳闻，随着应用程序功能不断的丰富，总有一天你会遇到一个异常：
Conversion to Dalvik format failed:Unable toexecute dex: method ID not in [0, 0xffff]: 65536可能有些同学会说，解决这个问题很简单，我们只需要在Project.proterty中配置一句话就Ok啦，dex.force.jumbo=true ，是的，加入了这句话，确实可以让你的应用通过编译，但是在一些2.3系统的机器上很容易出现INSTALL_FAILED_DEXOPT异常 ！
对于以上两个异常，我们先来分析一下原因：
1、Android系统中，一个Dex文件中存储方法id用的是short类型数据，所以导致你的dex中方法不能超过65k

2、在2.3系统之前，虚拟机内存只分配了5M
知道了原因，我们就来一个个的解决上面的问题，首先对于65k的问题，我们在应用层是无法改变android系统的结构的，所以我们无法将数据类型从short改变为int或者其他类型，也就是说一个dex中的方法数不能超过65k是我们无法逾越的鸿沟，我们只能减少一个dex中的方法数，首先最容易想到的方案就是去掉一些无用的Jar包，以及将一些属性设置为public，从而可以去掉get/set方法，这种方法只能临时解决问题，随着时间的推移，总有一天还是会出现方法数超过65k的，毕竟一个应用一般是在加功能，不会减功能。

下面我来向大家介绍两种主流的解决方案，一种是以微信为代表的，将一些功能做成插件，动态加载，另一种方案是以facebook为代表的分包方案，将一个apk中的dex文件分割成多个dex文件，然后动态的去加载dex文件。其实这两种方案的核心思想是一样的，插件是把未来要开发的新功能做成apk和dex动态加载，而分包方案是将已经完成的功能分成多个dex文件动态加载，其实我个人觉得插件方案比分包方案更好的解决了65k的问题，因为插件方案不仅能够解决65k问题，还能让我们的应用体积减小，而分包只能解决65k的问题。
关于插件开发，做成动态加载，我在很早之前一篇文章中就写过其基本思想，有兴趣的同学可以看看《实现Android 动态加载APK（Fragment or Activity实现）》

下面我们重点介绍分包机制
我们知道一个apk文件里面有一个dex文件，这个dex文件里面都是经过优化了的class文件，所谓分包，就是讲一个dex文件分成多个dex文件，这里我们约定一下，第一个dex叫做main.dex,第二个叫做second.dex，通常在分包的时候，我们需要将应用启动就需要使用的类放入到main.dex中，把不是立马就需要使用的类放入到second.dex中，对于Android系统，他只会默认加载main.dex的，second.dex对于他来说可能只是一个资源文件，它是不会主动去加载second.dex,所以我在应用启动的过程中，我们需要为second.dex创建好一个类加载器，便于我在使用second.dex中的类时，能够里面加载该类。
关于如何加载second.dex也有好多做法，用的比较多的主要有一下几种：

1、最简单的做法就是使用DexClassLoader进行加载，并将该DexClassLoader的父加载器设置为PathClassLoader

2、使用DexClassLoader加载，并将DexClassLoader的父加载器设置成PathClassLoader的父加载器，将PahtClassLoader的父加载器设置成DexClassLoader,仔细品味一下1和2的区别

3、将second.dex的路径放入到PathClassLoader的加载路径中
对于第2中方案，在有一种情况下是不能使用的，比如当second.dex通过DexClassLoader加载，但是second.dex中使用了一个类，这个类在main.dex中，这个时候就会抛出类找不到的异常，所以这种方案只能拥有second.dex不会用到main.dex类的时候

以上说的都是理论，下面我们来实战一下
我这里会介绍两种方案，一种是基于gradle构建Android项目，一种是基于Ant构建Android项目
方案一：基于gradle构建Android项目，并实现分包
环境要求：AndroidStudio0.9以上，gradle插件0.14.2以上
1、如果你的工程在eclipse中，那么你需要将该工程导入到Android中，此时需要你升级adt22以上

2、打开你工程的build.gradle文件，检查gradle插件是否是0.14.2版本之后，因为0.14.2之后gradle插件才支持分包、

3、打开工程下某一个Moudle的build.gradle文件，添加对android-support-multidex.jar的依赖

4、去掉第三方jar包中重复的类

5、设置虚拟机堆内存空间大小，避免在编译期间OOM

6、gradle构建项目时，貌似默认是不会将so库加入工程的，所以为了避免此种情况发生，我们需要制定so库目录，对于从eclipse转换过来的工程，还需要制定src和资源文件路径

7、如果你的项目依赖了其他库， 分别在各个库工程中加入 multiDexEnabled = true 和 jniLibs.srcDirs =['libs']两个配置即可

8、如果你的项目没有自定义Application，那么你在AndroidManifest.xml中使用MultiDexApplication即可，如果你的项目有自定义Application,并且是继承是Application，那么只需要改为继承MultiDexApplication即可，如果你的项目时继承的其他Application，那么你需要重写attachBaseContext


经过上述配置，你的项目应该是已经成功分包了。如果分包成功，那么你解压你的apk文件，会发现有两个dex文件，通过上述的配置过程，我们发现此方案我们无法控制哪些类在main.dex中，哪些类在second.dex中，通过此种方案配置分包，可以兼容API4-API20.其加载second.dex采用的是上述方案中的3
下面我们来看看基于Ant构建Android项目，并实现分包过程
在上述方案中，由于我们无法看到gradle构建项目的脚本，所以我们无法控制哪些类在第一个dex，哪些类在第二个dex，此方案中，我们采用Ant构建，Ant是允许用户自己定义构建方案的，比如我们可以通过自定义构建方案，将项目中某些第三方jar包放入到second.dex中，关于这个如何实现，请参考开源项目吧
https://github.com/mmin18/Dex65536.git

由于该项目加载second.dex所采用的方案是上述方案2，比如second.dex中的某些第三方jar包依赖main.dex中的某些类，这种方案就会实现，所以在此我将此方案去掉，换成了方案3，也就是将second.dex的路径设置到PathClassLoader的加载路径中，我只给出Android 4.4中的解决方案，其他系统大同小异
加载second.dex方法！
分包成功后，解压apk文件，进入assert文件夹，我们看到如下结构，libs.apk就是第三方jar编译后形成的dex文件对于上面提到的第二个问题INSTALL_FAILED_DEXOPT，根本原因就是2.3版本之前dalvik虚拟机的内存只有5M,所以无论是插件方案还是分包方案在某些手机上还是会遇到该问题，毕竟我们仅仅是减少了每个dex中包的数量，但是方法总数是没有减少的，所以解决此问题的根本方法就是修改虚拟机内存至8M,这个需求在Java层是无法实现，但是可以在c层实现，具体实现流程可以参考开源项目：https://github.com/viilaismonster/LinearAllocFix.git。至于该方法中用到的一些方法，可以到 android-support-multidex.jar中找到，这里就不都贴出来了，如果那里没有写清楚，欢迎留言讨论...！
15、访问开发API官网的问题:
采用火狐浏览器，设置为脱机工作

16、绑定Service相关问题:
要在注册的Service中，加入android:exported="true"，否则会
产生 java.lang.SecurityException: Not allowed to bind to service Intent
{ act=www.qslx.com.aidl.IRemoteService，这个错误，就是绑定不了这个服务

17、eclipse开发中调换xml布局控件顺序导致findViewById报错:
clean一下project就好了

18、PopupWindow点击外部会消失的解决方法:

setOutsideTouchable(true);// 只是外面可以点击，并不是点击可以消失
setBackgroundDrawable(new BitmapDrawable());// 加上以下句子才可以做到点击外部消失

setTouchInterceptor(new OnTouchListener() {

    @Override
    public boolean onTouch(View v, MotionEvent event) {

        if (event.getAction() == MotionEvent.ACTION_OUTSIDE) {
            dismiss();
            return true;
        }

        return false;
    }
});
19、getView()复用问题:
【1】先重置状态再选中状态
【2】用List或Bean把状态存起来

20、ListView点击Item没反应:
android:descendantFocusability="blocksDescendants"
http://www.cnblogs.com/eyu8874521/archive/2012/10/17/2727882.html

21、使用网络相关问题:
要导入

<uses-permission 
android:name="android.permission.INTERNET"/>
而不是

<uses-permission android:name="ANDROID.PERMISSION.INTERNET"/>
注意测试手机也要联网

22、ListView控件复用导致图片加载位置错误:
【1】为imageView绑定setTag()，在handler中getTag()相同才设置图片
【2】使用成员变量缓存传过来的变量，避免使用Thread导致时序上的错误

23、ListView图片多次加载问题:
使用LruCache<String,Bitmap>

24、ListView滑动卡顿问题:
【1】ListView滑动停止后才加载可见项
【2】ListView滑动时，取消所有加载项
【3】实现AbsListView.OnScrollListener

25、ListView最后一个item被底部布局挡住:
让ListView置于底部布局之上
android:layout_above="@+id/bottom"

26、ListView.setAdapter产生Null Pointer Exception:


Paste_Image.png
// 此处产生Null Pointer Exception闪退，因为mDatas为null
mListView.setAdapter(new ListDirArrayAdapter(context, mDatas));
27、Java无符号数的使用:

public static long getUnsignedIntt(long data) { // 0~4294967295   32为无符号数
        // (0xFFFFFFFF即DWORD)。
        return data & 0x00000000FFFFFFFFL; //L一定不能漏！!!
}
28、字符串equals返回false:
注意大小写，可输出比较

29、eclipse添加工程依赖错误 ( 解决方法同8 ):
jar包不一致，删除其一

30、Android Studio导入module产生gradle报错:
【1】修改build.gradle文件，把compileSdkVersion 、buildToolsVersion、
minSdkVersion、targetSdkVersion修改成一致
【2】修改gradle文件夹的wrapper的gradle-wrapper.properties，
修改gradle-2.4-all.zip
http://doc.okbase.net/x359981514/archive/112744.html

31、failed to find target with hash string 'android-22' :
修改 build.gradle 的 compileSdkVersion 和 buildToolsVersion

32、SVN合并代码冲突:
【1】提前备份代码
【2】用备份代码文件夹直接替换SVN的项目文件夹，然后Commit

33、友盟或 QQ 开放平台常见问题:
【1】友盟不支持Android Studio工程直接配置，需手工配置
【2】不管是友盟，还是QQ开放平台，android:label="@string/app_name"这个app_name要和注册时应用名称保持一致，如QSLXDEMO


Paste_Image.png

文／陆嘉杰（简书作者）
原文链接：http://www.jianshu.com/p/e0b46e8c4580
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。
