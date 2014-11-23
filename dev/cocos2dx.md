
# cocos2d-x 3.2
##创建项目
官方文档
```
$ cd cocos2d-x
$ ./setup.py
$ source .bash_profile
$ cocos new MyGame -p com.laicoo.mygame -l cpp -d NEW_PROJECTS_DIR
$ cd NEW_PROJECTS_DIR/MyGame
```

也可以进入到目录cocos2d-x-3.2alpha0/tools/cocos2d-console/bin/cocos.py
打开终端运行cocos.py脚本创建文件
```
python cocos.py new hellolua -p com.lacoo.game -l lua -d /workspace/
```
参数说明：

- new 后面接项目名称
- -p后面接包名
- -l后面接开发语言类型，有cpp, lua, js三种类型
- -d后面接项目存放的目录


## 编译
###命令行编译
```
cocos run -p android -j4  (mac air编译时间7分钟)
cocos run -p ios
cocos run -p mac
cocos run -p win32
```

- helloworld项目编译的项目包大小：
	模板例子生成的apk文件大小： 6.5M
		其中还没有压缩的源代码(不含cocos2dx及底层)图片文件大小1.1M
		libcocos2dlua.so：18.2M
		
	删除了curl后，apk文件大小：5.7M
		libcocos2dlua.so：16.5M
	Asset更新用到curl，Asset功能将不能再使用
	
	天天挂机的libcocos2dlua.so文件9.4M
	刀塔传奇的libhellolua.so5M

如果你是3.0的话： base\ccConfig.h 中可以设置各个宏开关 比如不要物理： 
```
#ifndef CC_USE_PHYSICS
#define CC_USE_PHYSICS 0   //0是不要 1是需要
#endif 
```
>修改makefile去掉 curl 支持，去掉物理引擎，去掉webp支持和sqlite，  libcocos2dcpp.so可以在2.5mb左右 


- quick如何缩小包体积
	为了制作小包，特意给 quick-cocos2d-x 2.2.5plus 增加了模块化编译能力。
	当禁用所有可选扩展后，libgame.so 的体积缩小到了惊人的 2780K，压缩到 apk 后仅占 1100K 字节（也就是 1M 多一点点）。
	屏蔽的扩展包：curl，jpeg，tiff，filters，ccb，ccstudio，sqllite，physics，webp，dragonbones

	默认空白的项目生产的APK文件大小：3.3M， libgame.so(8.5M)
	
	关闭：curl，tiff，filters，sqllite，physics，webp, dragonbones, ccb后生成的APK文件大小：1.8M
	libgame.so(4.8M)

	
>	参考：[quick v2 最后的疯狂，最小化so](http://quick.cocos.org/?p=1678)

	

###eclipse环境编译
cocos compile编译好so文件，再通过eclipse打开android的项目：
1. hello/frameworks/runtime-src/pro.android
2. hello/framework/cocos2d-x/cocos/platform/android/java

编译运行即可以跑起来。


##COCOS2DX API风格

- 两阶段构造器及静态create()函数
在Cocos2d-x引擎中，我们使用了两阶段构造器及静态create()函数，构造器由构造函数和init函数组成，由静态函数create()来管理两阶段构造器，同时做自动释放引用计数。

- doSomething()
这是最常见的函数名，在Cocos2d-x引擎中处处都有应用到。第一个字是一个动词，第二个字是一个名词。比如：replaceScene(CCScene*) 和 getTexture()。

- doWithResource()
它是doSomething()方法的变体。在initWithTexture(CCTexture*) 和 initWithFilename(const std::string&)中，你经常可以看见这一函数名。

- onEventCallback()
当你看到类似void onEnter()的函数名时，onAction类型表明这是一个回调函数。当引发一定条件时，其他类将调用这一方法。比如：
```
class Layer
{
public:
    virtual void onEnter();
    virtual void onExit();
    virtual void onEnterTransitionDidFinish();
}
```

- getInstance()/shareClass()
在Cocos2d-x引擎中，如果你没有发现create()，只发现了getInstance()方法，它就属于单例模式类。比如，TextureCache::getInstance()。单例类对应的析构方式是destroyInstance()。 在v3.0之前，单例类的构造方式是CocosClass::sharedCocosClass()，比如TextureCache::sharedTextureCache()。这个方法在v3.0中仍然可以兼容，但不保证在v3.0更后面的版本中仍然保留。
```
	FileUtils::getInstance()
	FileUtils::sharedFileUtils()
	
	FileUtils::destroyInstance()
	FileUtils::purgeFileUtils()
```

- 属性
因为在C++ 和 C++11中没有"property" （“属性”）这个概念，所以我们在Cocos2d-x引擎中使用了许多getter和setters。

	- 如果属性为“只读”，将不会有setProperty(type)方法；
	- 如果属性为一个bool值，将会有setProperty(bool)及 isProperty()方法。 比如：Sprite::isDirty()和Sprite::setDirty(bool bDirty)。
	- 如果属性不是一个bool值，将会有 setProperty(type) 和 getProperty() 方法。比如： void Sprite::setTexture(Texture2D*) 和 Texture2D* CCSprite::getTexture()。


##cocos2dx内存优化

- 一帧一帧载入游戏资源，通过CCTextureCache::sharedTextureCache()->dumpCachedTextureInfo();来观察纹理的内存占用情况
- 减少绘制调用，使用“CCSpriteBatchNode”
- 载入纹理时按照从大到小的顺序
- 避免高峰内存使用
- 使用载入屏幕预载入游戏资源
- 需要时释放空闲资源
- 收到内存警告后释放缓存资源.
- 使用纹理打包器优化纹理大小、格式、颜色深度等
- 使用JPG格式要谨慎！
- 请使用RGB4444颜色深度16位纹理
- 请使用NPOT纹理，不要使用POT纹理
- 避免载入超大纹理
- 推荐使用MP3数据格式的音频文件，因为Android平台和iOS平台均支持MP3格式，音频文件采样率大约在96-128kbps为佳，比特率44kHz就够了。
- 使用的是苹果的工具“Allocation & Leaks”。你可以在Xcode中长按“Run”命令，选择“ Profile ”来启动这两个工具。使用Allocation工具可以监控应用的内存使用，使用Leaks工具可以观察内存的泄漏情况。

##Label
- freetype
3.0cocos2dx开始使用freetype作为文字的处理核心，开始支持：阴影，边框，荧光效果实现，边框和荧光需要TTF字库的支持。以前通过每个文字生成一张图片，没有缓存，性能低下。


- 自定义实现：下划线超链文本控件


- 富文本实现
```
    local _richtext = ccui.RichText:create()

    _richtext:ignoreContentAdaptWithSize(false)  

    _richtext:setContentSize(cc.size(300, 100))
    _richtext:align(display.CENTER, display.cx, display.cy-100)

    local re1 = ccui.RichElementText:create( 1, cc.c3b(255, 255, 255), 255, "This color is white. ", "Helvetica", 20 )  
    local re2 = ccui.RichElementText:create( 2, cc.c3b(255, 255,   0), 255, "And this 中文中文.", "Helvetica", 20 )  
    local reimg = ccui.RichElementImage:create( 6, cc.c3b(255, 255, 255), 255, "lock.png" )  


    _richtext:pushBackElement(re1)  
    _richtext:pushBackElement(re2)  
    _richtext:pushBackElement(reimg) 

    _richtext:addTo(self)   
```



## UI编辑器



## 粒子系统

mac： ParticleDesigner 破解版

> Particle Editor for Cocos2dx：http://games.v-play.net/particleeditor/


##音频


| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

