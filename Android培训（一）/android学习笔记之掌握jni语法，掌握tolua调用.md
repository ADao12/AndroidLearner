##Android学习笔记之掌握JNI语法，掌握tolua调用
####From JiaYing.Cheng

---
---

###掌握JNI语法

1. [cocos2d-x中JniHelper类详细使用](https://app.yinxiang.com/shard/s4/sh/aa24d836-4eef-4441-b976-4c77d5910636/d5fceed7ec38e61f8b6a50d4fb53b759)
2. [Cocos2d-x利用jni调用java层代码](https://app.yinxiang.com/shard/s4/sh/54298e0d-6087-4db6-8fe5-9919e88cc68a/fdc65ac323d23b95f7824e5f7f22a7b5)
3. [cocos2d-x 通过JNI实现c/c++和Android的java层函数互调](https://app.yinxiang.com/shard/s4/sh/35de085d-cfe4-477c-b197-6a37ca000bf9/218408fdf6d95c84eaf16f74ebda7b8f)
4. [cocos2d-x中的Jni使用（C++与Andriod方法互调）](https://app.yinxiang.com/shard/s4/sh/530818b8-a717-4aa7-ad91-51ba2eb39e9d/50ff6b4577c1cc4781d926d101779a11)
5. [C/C++调用java，以及在cocos2d-x下的实现](https://app.yinxiang.com/shard/s4/sh/6a3aee21-fddc-4b76-84ba-f468109f9b46/d3a30d82b4305abc2362e3ecca9fc6f4)
6. [cocos2d-x中通过Jni实现Java与C++的互相调用](https://app.yinxiang.com/shard/s4/sh/d9157d03-57db-4e8e-9c9e-82fb4974da6a/efcf1aff8a771c74f399e0af7a7eb055)

以上六篇笔记阅读完后，对Cocos2dx与Java通过JNI互调的流程、使用方法会有一个完整的认识，可以快速应用到实际开发中去。

---

###掌握tolua调用
在Cocos2dx的luaTest中有很多例子，在这里主要讲的是Java代码通过JNI声明后,在C++中实现，把C++接口绑定到lua中，通过lua可以间接调用Java代码的流程和方法。
___
以项目中的分享组件为例子。

`../project/FanRen/proj.android/proj.gradle/FLPlatformLibrary/src/main/java/com/flamingo/flplatform/util/JNIDelegateProxy.java`

```
public class JNIDelegateProxy {

	...
	//Java中实现的分享功能方法
	public static void JNIProxy_share(int type, String content, String imagePath) {
        LogUtil.log("===Java===JNIProxy_share");
        ChuKongSDK.getInstance().JNI_share(type, content, imagePath);
    }
    
    ...
    
}
```

在C++实现要通过JNI调用java方法的类FLSocial(FLSocial.h、FLSocial.cpp)

`../project/FLPlatform/social/FLSocial.h`

```
 #ifndef __FLPlatform__FLSocial__
 #define __FLPlatform__FLSocial__

 #include "../common/FLCommon.h"

class FLSocial : public CCObject
{
public:
    typedef enum{
        kSocialSinaWeibo = 0,
        kSocialWeChat,
    }SocialType;
    /**
     *  分享
     *
     *  @param type      分享的类型
     *  @param text      分享的文字内容
     *  @param imagePath 分享的图片对应的路径，NULL表示不分享图片
     */
    virtual void share( SocialType type, const char* text, const char* imagePath = NULL);
    
    protected:
    friend class FLPlatformManager;
    /**
     *  创建社交分享类。请使用FLPlatformManager::sharedInstance()->getSocial()
     *
     *  @return
     */
    static FLSocial* create();
    ~FLSocial();
    /**
     *  分享的回调，调用share后一定会回调，通知Lua。回调内容中包含了结果
     *
     *  @param params 回调结果的Dictionary
     */
     
    virtual void shareCallback(CCObject* params);
    virtual bool init();
    };
    #endif /* defined(__FLPlatform__FLSocial__) */
```

`../project/FLPlatform/social/FLSocial.cpp`

代码略。

真正的实现在
`../project/FLPlatform/social/android/FLSocial_Android.cpp`

```
void FLSocial_Android::share(SocialType type, const char *text,const char* imagePath)
{
#if (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
    JniMethodInfo t;
    if (JniHelper::getStaticMethodInfo(t, CLASS_NAME, "JNIProxy_share", "(ILjava/lang/String;Ljava/lang/String;)V")) {
        jstring stringArg1 = t.env->NewStringUTF(text);
        jstring stringArg2 = t.env->NewStringUTF(imagePath);
        t.env->CallStaticVoidMethod(t.classID, t.methodID, type, stringArg1, stringArg2);
        t.env->DeleteLocalRef(t.classID);
        t.env->DeleteLocalRef(stringArg1);
        t.env->DeleteLocalRef(stringArg2);
    }
#else
    CCLOG("chl %s","===X=== this platform not support JNI");
#endif
}
```

然后在
`../project/FanRen/proj.android/proj.gradle/Dragon/jni/Android.mk`
里面关联C++的cpp文件，以便在打包Android项目的时候纳入动态库（.so文件）里面

```
LOCAL_SRC_FILES := \
//添加FLSocial_Android.cpp
../../../../../FLPlatform/social/Android/FLSocial_Android.cpp \
//添加FLSocial.cpp
../../../../../FLPlatform/social/FLSocial.cpp \
```

以上流程为C++通过JNI调用Java方法的实现流程

___

以下是把C++绑定到Lua,让Lua通过调用C++来间接调用Java方法

首先，编写，或用脚本生成，XXX.tolua文件，用来声明暴露给Lua的相关接口（一般是常量、公有方法、静态方法）

分享组件的.tolua文件在`project/tools/tolua++/FanrenTolua/FLPlatform/social/FLSocial.tolua`

```
class FLSocial: public CCObject
{
    typedef enum{
        kSocialSinaWeibo = 0,
        kSocialWeChat,
    }SocialType;
    void share( SocialType type, const char* text, const char* path = NULL);
};
```

XXX.tolua文件的内容一般是从XXX.h复制过来，然后删除那些私有的、保护的、不想在lua的暴露的方法声明或常量。

最后通过LuaEngine把C++的类型转化为Lua里面的类型，压入Lua栈中，以便Lua那边的调用。

在项目中，一般会有一个类`../project/FanRen/Classes/LuaFanren/LuaFanren.cpp`集中地对自定类的统一处理C++、Lua的绑定流程。

绑定一个方法的实现细节如下

```
/* method: share of class  FLSocial */
#ifndef TOLUA_DISABLE_tolua_Fanren_FLSocial_share00
static int tolua_Fanren_FLSocial_share00(lua_State* tolua_S)
{
#ifndef TOLUA_RELEASE
 tolua_Error tolua_err;
 if (
     !tolua_isusertype(tolua_S,1,"FLSocial",0,&tolua_err) ||
     !tolua_isnumber(tolua_S,2,0,&tolua_err) ||
     !tolua_isstring(tolua_S,3,0,&tolua_err) ||
     !tolua_isstring(tolua_S,4,1,&tolua_err) ||
     !tolua_isnoobj(tolua_S,5,&tolua_err)
 )
  goto tolua_lerror;
 else
#endif
 {
  FLSocial* self = (FLSocial*)  tolua_tousertype(tolua_S,1,0);
  FLSocial::SocialType type = ((FLSocial::SocialType) (int)  tolua_tonumber(tolua_S,2,0));
  const char* text = ((const char*)  tolua_tostring(tolua_S,3,0));
  const char* path = ((const char*)  tolua_tostring(tolua_S,4,NULL));
#ifndef TOLUA_RELEASE
  if (!self) tolua_error(tolua_S,"invalid 'self' in function 'share'", NULL);
#endif
  {
   self->share(type,text,path);
  }
 }
 return 0;
#ifndef TOLUA_RELEASE
 tolua_lerror:
 tolua_error(tolua_S,"#ferror in function 'share'.",&tolua_err);
 return 0;
#endif
}
#endif //#ifndef TOLUA_DISABLE
```

这里方法绑定的个数理应和FLSocial.tolua中定义的方法的个数一样，不然会让人很困扰。


```
/* Open function */

TOLUA_API int tolua_Fanren_open (lua_State* tolua_S)
{
	tolua_usertype(tolua_S,"FLSocial");
	
	...

//	
tolua_cclass(tolua_S,"FLSocial","FLSocial","CCObject",NULL);
  tolua_beginmodule(tolua_S,"FLSocial");
   tolua_constant(tolua_S,"kSocialSinaWeibo",FLSocial::kSocialSinaWeibo);
   tolua_constant(tolua_S,"kSocialWeChat",FLSocial::kSocialWeChat);
   tolua_function(tolua_S,"share",tolua_Fanren_FLSocial_share00);
  tolua_endmodule(tolua_S);

	...

}
```

- tolua_beginmodule(m_pState, "CTest");是只注册一个模块，比如，我们管CTest叫做"CTest"，保持和C++的名称一样。这样在Lua的对象库中就会多了一个CTest的对象描述，等同于string,number等等基本类型，同理，你也可以用同样的方法，注册你的MFC类。
- 这里要注意，tolua_beginmodule()和tolua_endmodule()对象必须成对出现，如果出现不成对的，你注册的C++类型将会失败。
- tolua_function(m_pState, "SetData", tolua_SetData_CTest);指的是将Lua里面CTest对象的"SetData"绑定到你的tolua_SetData_CTest()函数中去。
- tolua_constant(tolua_S, "ES_AUTOHSCROLL",   ES_AUTOHSCROLL);
这样注册，你就可以在 Lua里面使用ES_AUTOHSCROLL这个常数，它会自动绑定ES_AUTOHSCROLL这个C++常数对象。


____
经过以的上流程，已经把FLSocial.tolua和在LuaFanren.cpp准备好了，其实是已经准备好了FLSocial.tolua的实现。项目中是集中通过命令绑定项目中的Lua和C++,所以我们还需要Fanren.tolua（`../project/tools/tolua++/FanrenTolua/Fanren.tolua`）文件来集中把类似FLSocial.tolua的其他模块的XXX.tolua文件包含在一起以及basic.lua（`../project/tools/tolua++/FanrenTolua/basic.lua`）文件。

Fanren.tolua

```
$pfile "CommandLineHelper.tolua"
$pfile "FRMD5.tolua"
$pfile "FLPlatform/device/FLDevice.tolua"
$pfile "FLPlatform/device/FLKeypadManager.tolua"
$pfile "FLPlatform/platform/FLPlatformManager.tolua"
$pfile "FLPlatform/userSystem/FLUserSystem.tolua"
$pfile "FLPlatform/error/FLErrorCollector.tolua"
$pfile "FLPlatform/application/FLApplication.tolua"
$pfile "FLPlatform/application/FLWebView.tolua"
$pfile "FLPlatform/push/FLPush.tolua"
$pfile "FLPlatform/video/FLVideoPlayer.tolua"

$pfile "FLPlatform/social/FLSocial.tolua"

$pfile "FLPlatform/qrCode/FLQREncode.tolua"
$pfile "std/vector.tolua"
$pfile "rmi/CDELuaMessageHandler.tolua"
//$pfile "rmi/CdlShareObject.tolua"
$pfile "rmi/CLuaCdeOutgoing.tolua"
$pfile "rmi/CLuaCdeSerializestream.tolua"
$pfile "rmi/CLuaSessionManager.tolua"
$pfile "rmi/FRHttpClient.tolua"
$pfile "frUtil/Fanren.tolua"
$pfile "frUtil/FRFile.tolua"
$pfile "frUtil/FREngineUtil.tolua"

```

最后运行FanrenBuild.sh(`../project/tools/tolua++/FanrenTolua/FanrenBuild.sh`)就可以把C++类型和方法绑定到Lua中，可以间接调用Java的方法。

```
#!/bin/bash
tolua++ -L basic.lua -o "../../../FanRen/Classes/LuaFanren/LuaFanren.cpp"  Fanren.tolua
```

当然这个脚本已经由自动化工具经过比较文档版本号自动运行的，我们只需要把上面流程中的每一步做好，提交SVN,喝杯茶稍等片刻就好。