##Android学习笔记之Android签名详解（debug和release）
####From JiaYing.Cheng

---
---
###1. 为什么要签名
######1) 发送者的身份认证

由于开发商可能通过使用相同的Package Name来混淆替换已经安装的程序，以此保证签名不同的包不被替换

######2) 保证信息传输的完整性

签名对于包中的每个文件进行处理，以此确保包中内容不被替换

######3) 防止交易中的抵赖发生，Market对软件的要求

###2. 签名的说明
1) 所有的应用程序都必须有数字证书，Android系统不会安装一个没有数字证书的应用程序

2) Android程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证

3) 如果要正式发布一个Android应用，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用adt插件或者ant工具生成的调试证书来发布

4) 数字证书都是有有效期的，Android只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能

5) 签名后需使用zipalign优化程序

6) Android将数字证书用来标识应用程序的作者和在应用程序之间建立信任关系，而不是用来决定最终用户可以安装哪些应用程序

###3. 签名的方法
有三种打包方式：命令行手动打包、ant自动编译打包、eclipse+ADT编译打包，打包步骤如下：

######第一步 生成R.java类文件：

Eclipse中会自动生成R.java，ant和命令行使用android SDK提供的aapt.ext程序生成R.java。

######第二步 将.aidl文件生成.java类文件：

Eclipse中自动生成，ant和命令行使用android SDK提供的aidl.exe生成.java文件。

######第三步 编译.java类文件生成class文件：

Eclipse中自动生成，ant和命令行使用jdk的javac编译java类文件生成class文件。

######第四步 将class文件打包生成classes.dex文件：

Eclipse中自动生成，ant和命令行使用android SDK提供的dx.bat命令行脚本生成classes.dex文件。

######第五步 打包资源文件（包括res、assets、androidmanifest.xml等）：

Eclipse中自动生成，ant和命令行使用Android SDK提供的aapt.exe生成资源包文件。

######第六步 生成未签名的apk安装文件：

Eclipse中自动生成debug签名文件存放在bin目录中，ant和命令行使用android SDK提供的apkbuilder.bat命令脚本生成未签名的apk安装文件。

######第七步 对未签名的apk进行签名生成签名后的android文件：

Eclipse中使用Android Tools进行签名，ant和命令行使用jdk的jarsigner对未签名的包进行apk签名。

 

#####3.1 用eclipse+ADT方式签名
详见：http://jojol-zhou.iteye.com/blog/719428

######a) 调试签名

　　eclipse插件默认赋予程序一个DEBUG权限的签名，此签名的程序不能发布到market上，此签名有效期为一年，如果过期则导致你无法生成apk文件，此时你只要删除debug keystore即可，系统又会为你生成有效期为一年的新签名

######b) 开发者生成密钥并签名

　　右键点击项目名，在菜单中选择Android Tools，然后选择Export Signed Application Package，即可通过eclipse自定义证书并签名

######c) 开发者导出未签名的包

　　右键点击项目名，在菜单中选择Android Tools，然后选择Export Signed Application Package…，即可导出未签名的包，之后可通过命令行方式签名

#####3.2 用命令行方式签名
使用标准的java工具keytool和jarsigner来生成证书和给程序签名。

详见：http://jojol-zhou.iteye.com/blog/729254

######a) 生成签名

$ keytool -genkey -keystore keyfile -keyalg RSA -validity 10000 -alias yan

注：validity为天数，keyfile为生成key存放的文件，yan为私钥，RSA为指定的加密算法(可用RSA或DSA)

######b) 为apk文件签名

$ jarsigner -verbose -keystore keyfile -signedjar signed.apk base.apk yan

注：keyfile为生成key存放的文件，signed.apk为签名后的apk，base.apk 为未签名的apk，yan为私钥

######c) 看某个apk是否经过了签名

$ jarsigner -verify my_application.apk

######d) 优化（签名后需要做对齐优化处理）

$ zipalign -v 4 your_project_name-unaligned.apk your_project_name.apk

#####3.3 在源码中编译的签名
######a) 使用源码中的默认签名

在源码中编译一般都使用默认签名的，在某源码目录中用运行

$ mm showcommands能看到签名命令

Android提供了签名的程序signapk.jar，用法如下：

$ signapk publickey.x509[.pem] privatekey.pk8 input.jar output.jar

*.x509.pem为x509格式公钥，pk8为私钥

build/target/product/security目录中有四组默认签名可选：testkey platform shared media（具体见README.txt），应用程序中Android.mk中有一个LOCAL_CERTIFICATE字段，由它指定用哪个key签名，未指定的默认用testkey.

######b) 在源码中自签名

Android提供了一个脚本mkkey.sh（build/target/product/security/mkkey.sh），用于生成密钥，生成后在应用程序中通过Android.mk中的LOCAL_CERTIFICATE字段指名用哪个签名

######c) mkkey.sh介绍

- 生成公钥

openssl genrsa -3 -out testkey.pem 2048

其中-3是算法的参数，2048是密钥长度，testkey.pem是输出的文件

- 转成x509格式（含作者有效期等）

openssl req -new -x509 -key testkey.pem -out testkey.x509.pem -days 10000 -subj ‘/C=US/ST=California/L=MountainView/O=Android/OU=Android/CN=Android/emailAddress=android@android.com’

- 生成私钥

openssl pkcs8 -in testkey.pem -topk8 -outform DER -out testkey.pk8 -nocrypt

把的格式转换成PKCS #8，这里指定了-nocryp，表示不加密，所以签名时不用输入密码

###4. 签名的相关文件
1) apk包中签名相关的文件在meta_INF目录下

CERT.SF：生成每个文件相对的密钥

MANIFEST.MF：数字签名信息

xxx.SF：这是JAR文件的签名文件，占位符xxx标识了签名者

xxx.DSA：对输出文件的签名和公钥

2) 相关源码

development/tools/jarutils/src/com.anroid.jarutils/SignedJarBuilder.java

frameworks/base/services/java/com/android/server/PackageManagerService.java

frameworks/base/core/java/android/content/pm/PackageManager.java

frameworks/base/cmds/pm/src/com/android/commands/pm/Pm.java

dalvik/libcore/security/src/main/java/java/security/Sign*

build/target/product/security/platform.*

build/tools/signapk/*

###5. 签名的相关问题
一般在安装时提示出错：INSTALL_PARSE_FAILED_INCONSISTENT_CERTIFICATES

1) 两个应用，名字相同，签名不同

2) 升级时前一版本签名，后一版本没签名

3) 升级时前一版本为DEBUG签名，后一个为自定义签名

4) 升级时前一版本为Android源码中的签

######5.1 查看默认签名
　　不同的机子上或不同的设备上，利用eclipse编译出的apk签名是不一样的。eclipse都有一个默认的签名。查看签名路径：

1）打开EclipseàWindowàAndoridàBuild，在这个Build界面，找到Default debug keysore这个编辑框，里面的值则为本台设备中Eclipse的keystore的默认路径。

2）如果出现因签名不同而导致应用程序未安装，可以将原先的keystore替换掉当前设备上的keystore。并重新启动Eclipse。否则只能完全卸载掉移动设备上的apk，重新安装了。

######5.2 无法覆盖安装
1、通过签名的方式生成你的APK，而不是直接从Bin目录底下去拷贝，每个Android可执行程序的APK都有自己的签名，只要签名一致，就可以覆盖安装，而不需要卸载；

2、数据库表结构的变化（增加一个字段，减少一个字段，新表的建立）。正常升级数据库的方法 public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)

3、sharepreferences的数据有改变，这个跟数据库差不多，比如原来的sharepreferences保存的一数据是boolean，在后一版本把保存的数据改为string，问题就出现了。

######5.3 导出签名
1）打开cmd控制台，输入命令：keytool -genkey -alias android.keystore -keyalg RSA -validity 20000 -keystore android.keystore，按照提示依次填写内容，并记住密码，后面会用到；

2）生成好keystore后，就可以导出签名apk了。Eclipse中，右击需要签名的工程àandroid toolsàexport signed application package，location为生成的keystore所在的位置，密码为创建keystore时设置的密码，然后按照提示，next，最后finish，成功导出签名apk。

详见：http://blog.csdn.net/yiwanxinyuefml/article/details/6765129

######5.4 debug签名和release签名的区别
1）debug签名的应用程序不能在Android Market上架销售，它会强制你使用自己的签名；Debug模式下签名用的证书(默认是Eclipse/ADT和Ant编译)自从它创建之日起，1年后就会失效。

2）debug.keystore在不同的机器上所生成的可能都不一样，就意味着如果你换了机器进行apk版本升级，那么将会出现上面那种程序不能覆盖安装的问题，相当于软件不具备升级功能！

###6. Zipalign简单优化
######6.1 为什么要优化
　　Android SDK中包含一个“zipalign”的工具，它能够对打包的应用程序进行优化。在你的应用程序上运行zipalign，使得在运行时Android与应用程序间的交互更加有效率。因此，这种方式能够让应用程序和整个系统运行得更快。我们强烈推荐在新的和已经发布的程序上使用zipalign工具来得到优化后的版本——即使你的程序是在老版本的Android平台下开发的。

######6.2 如何有助于性能改善
　　在Android中，每个应用程序中储存的数据文件都会被多个进程访问；安装程序会读取应用程序的manifest文件来处理与之相关的权限问题；Home应用程序会读取资源文件来获取应用程序的名和图标；系统服务会因为很多种原因读取资源（例如，显示应用程序的Notification）；此外，就是应用程序自身用到资源文件。

　　在Android中，当资源文件通过内存映射对齐到4字节边界时，访问资源文件的代码才是有效率的。但是，如果资源本身没有进行对齐处理（未使用zipalign工具），它就必须回到老路上，显式地读取它们——这个过程将会比较缓慢且会花费额外的内存。

　　对于应用程序开发者来说，这种显式读取方式是相当便利的。它允许使用一些不同的开发方法，包括正常流程中不包含对齐的资源，因此，这种读取方式具有很大的便利性（本段的原始意思请参考原文）。

　　遗憾的是，对于用户来说，这个情况恰恰是相反的——从未对齐的apk中读取资源比较慢且花费较多内存。最好的情况是，Home程序和未对齐的程序启动得比对齐后的慢（这也是唯一可见的效果）。最坏的情况是，安装一些未对齐资源的应用程序会增加内存压力，并因此造成系统反复地启动和杀死进程。最终，用户放弃使用如此慢又耗电的设备。

 

######6.3 如何优化
1）使用ADT：

如果你使用导出向导的话，Eclipse中的ADT插件（从Ver.0.9.3开始）就能自动对齐Release程序包。

使用向导，右击工程属性，选择Android ToolsàExport Signed Application Package。当然，你还可以通过AndroidManifest.xml编辑器的第一页做到。

2）使用Ant：

Ant编译脚本（从Android 1.6开始）可以对齐程序包。老平台的版本不能通过Ant编译脚本进行对齐，必须手动对齐。

从Android 1.6开始，Debug模式下编译时，Ant自动对齐和签名程序包。

Release模式下，如果有足够的信息签名程序包的话，Ant才会执行对齐操作，因为对齐处理发生在签名之后。为了能够签名程序包，进而执行对齐操作，Ant必须知道keystore的位置以及build.properties中key的名字。相应的属性名为key.store和key.alias。如果这些属性为空，签名工具会在编译过程中提示输入store/key的密码，然后脚本会执行签名及apk文件的对齐。如果这些属性都没有，Release程序包不会进行签名，自然也就不会进行对齐了。

3）手动：

为了能够手动对齐程序包，Android 1.6及以后的SDK的tools/文件夹下都有zipalign工具。你可以使用它来对齐任何版本下的程序包。你必须在签名apk文件后进行，使用以下命令：zipalign -v 4 source.apk destination.apk

4）验证对齐：

zipalign -c -v 4 application.apk