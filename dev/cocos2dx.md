
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


## 编译及最小化.so
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
	刀塔传奇的libhellolua.so 5.4M

如果你是3.0的话： base\ccConfig.h 中可以设置各个宏开关 比如不要物理： 
```
#ifndef CC_USE_PHYSICS
#define CC_USE_PHYSICS 0   //0是不要 1是需要
#endif 
```
>修改makefile去掉 curl 支持，去掉物理引擎，去掉webp支持和sqlite，  libcocos2dcpp.so可以在2.5mb左右 
但是我实际操作apk文件大小极限4.3M， libcocos2dlua.so（12.7M）

- quick如何缩小包体积
	为了制作小包，特意给 quick-cocos2d-x 2.2.5plus 增加了模块化编译能力。
	当禁用所有可选扩展后，libgame.so 的体积缩小到了惊人的 2780K，压缩到 apk 后仅占 1100K 字节（也就是 1M 多一点点）。
	屏蔽的扩展包：curl，jpeg，tiff，filters，ccb，ccstudio，sqllite，physics，webp，dragonbones

	默认空白的项目生产的APK文件大小：3.3M， libgame.so(8.5M)
	
	关闭：curl，tiff，filters，sqllite，physics，webp,  dragonbones,  ccb后生成的APK文件大小：1.8M
	libgame.so(4.8M)

	
>	参考：[quick v2 最后的疯狂，最小化so](http://quick.cocos.org/?p=1678)

世界OL：7.52M(Android)
2048手游：1.9M(Android)，3.5M(iOS)


###eclipse环境编译
cocos compile编译好so文件，再通过eclipse打开android的项目：
1. hello/frameworks/runtime-src/pro.android
2. hello/framework/cocos2d-x/cocos/platform/android/java，或者直接复制源码过去

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


##cocos2dx3.0更新重点

###渲染流水线
* 从渲染器中把场景图形解耦出来。
访问节点时发布图形命令，并把它们添加到一个队列上，但实际上不调用任何OpenGL的渲染代码，由下面的后台线程处理执行。

* 分形几何选择裁剪视图
精灵（Sprite）以及一般的几何图形，如果它们不出现在摄影机的视图范围内，会自动被当前帧移除，不会被选人呢

* 渲染线程 所有渲染执行的执行（如：OpenGL调用）会被移到别的线程中执行，而不是在主线程中执行（这将有助于并行处理，更好的利用多核CPU的能力）

* 自动化批处理
有效地较少绘图调用次数，自动将绘图调用尽可能地成批处理（如：使用同一纹理的精灵）。

* 基于节点的自定义渲染
在当前的Cocos版本中，开发者能够按需在每个节点的基础上自定义渲染，直接调用OpenGL命令，不顾官方的渲染器（不过很可能导致性能减弱）。


> [Cocos2d-x v3.0 渲染流水线 路线图](http://cn.cocos2d-x.org/tutorial/show?id=1558)

###New Label
* 发光、阴影和轮廓效果的支持，边框和荧光需要TTF字库的支持。
* 使用freetype2为标签生成纹理，这种方式提升了生成字体的速度并保证了字体在不同平台下有相同的效果。
2.0版本对文字生成一张图片，没有缓存，性能低下。

> [Cocos2d-x v3.x中的New Label](http://cn.cocos2d-x.org/tutorial/show?id=1446)

##场景
* 尽量少做场景的切换，让切换在层里面完成。
* 场景切换注意，旧的场景onexit()释放，在新场景的init()之后，新场景的onEnterTransitionDidFinish()之前。
（1）像背景图这种只能放到init中，像场景切换时要看到的一些精灵，必须放到init中，不然场景切换时会看不到背景或者一些精灵。
（2）象精灵的一些动画，动作，可以放到onEnterTransitionDidFinish中来初始化。
* 对于有好几个层的场景，在init的时候，先加载第一个层的，而不是把所有层的东西全部加载完，在切换层的时候再加载相对应的层

##层

addChild( Node *child, int zOrder, int tag )

* Child：参数就是节点。对于场景而言，通常我们添加的节点就是层。
* zOrder：先添加的层会被置于后添加的层之下。如果需要为它们指定先后次* 序，可以使用不同的zOrder值。
* tag：是元素的标识号码，如果为子节点设置了tag值，就可以在它的父节点中利用tag值就可以找到它了。
层可以包含任何Node作为子节点，包括Sprites(精灵), Labels(标签)，甚至其他的Layer对象。

tag[重要]，ccs的布局需要用到：
```
sceneGame:addChild(layer2, 2, 2);

local ly = sceneGame:getChildByTag(2)
ly:setPosition(ccp(500,50))
```

##调度器

sharedDirector:getScheduler():scheduleScriptFunc(unsigned int nHandler, float fInterval, bool bPaused)

第一个参数是你要注册的回调函数，第二个是执行该函数循环的秒数，第三个参数是是否暂停。
一般使用例子：scheduleScriptFunc(callbackFunc,0,false)即可。


##通知中心
      
    local function onEnter()
        cclog("ON ENTER")
        CCNotificationCenter:sharedNotificationCenter():registerScriptObserver(ccLayer, testNotificationHandler, LUA_TEST_MSG)
        CCNotificationCenter:sharedNotificationCenter():registerScriptObserver(ccLayer, testNotificationHandler2, LUA_TEST_MSG)
    end
    
    local function onExit()
        cclog("ON EXIT")
        CCNotificationCenter:sharedNotificationCenter():unregisterScriptObserver(ccLayer, LUA_TEST_MSG)
    end


##CCNode

放大缩小：
node:setScale(node:getScale()*2)

倾斜：
node:setSkewX(20)

按钮事件：
```
    node:setTouchEnabled(true)
    node:setTouchSwallowEnabled(false)  --默认ture，不透传
    node:setTouchMode(TOUCH_MODE_ONE_BY_ONE) --默认TOUCH_MODE_ONE_BY_ONE
    node:addNodeEventListener(NODE_TOUCH_EVENT, function (event)
        if event.name == "began" then
             print("spriter began")
             node:setScale(node:getScale()*2)
        elseif event.name == "moved" then
            print("spriter moved")
        elseif event.name == "ended" then
             print("spriter ended")
             node:setScale(node:getScale()/2)
        end
 
        return true
    end)
```

NODE_TOUCH_CAPTURE_EVENT与NODE_TOUCH_EVENT
1.CAPTURE事件用于捕获TOUCH事件，因此优先于TOUCH事件触发；
2.CAPTURE事件不像TOUCH事件按照显示层级来分发，而是按照节点层级关系，多次循环分发；
3.CAPTURE事件began不返回true，则吞噬自身和其子节点的TOUCH事件，但依旧按照节点层级多次循环分发CAPTTURE事件。

简单来说，CAPTURE事件的分发顺序，类似于Node的OnEnter事件，父节点优先触发，然后子节点，最后子子节点。
但又不同于Node事件，CAPTURE事件存在一个循环分发的机制，一个began可能会触发多次。




如何让背景变暗?    以及中心区域变亮，边缘昏暗？
如何让spriter不超出size?

设置记录到json文件?

helper:
  音效，背景音乐
  开关设置
  音量设置（后期）

超链链接文字，RichText斜体

文字出场特效

示例：
http://blog.csdn.net/crayondeng/article/category/1622039

http://code4app.com/ios/%E6%A8%A1%E4%BB%BF%E5%90%88%E9%87%91%E5%BC%B9%E5%A4%B4Demo/50b1d3246803faac0a000000



##ScrollView ScrollTable


http://blog.csdn.net/crayondeng/article/details/12887173

http://cocos2d.9tech.cn/news/2014/0331/40138.html



##Label

- todo: 自定义实现：下划线超链文本控件

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

##音频


| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |



## UI编辑器

cocos studio对应版本
http://www.cocoachina.com/bbs/read.php?tid=182077

## 粒子系统

mac： ParticleDesigner 破解版

> Particle Editor for Cocos2dx：http://games.v-play.net/particleeditor/


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








##quick debug

在lua中使用print()打印log没有任何响应，请教各位大侠怎么解决。
我使用的是mac系统，xcode5.1.

解决办法
1.定义宏COCOS2D_DEBUG=1
2.把Build configuration设为debug 