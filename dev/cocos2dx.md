
# cocos2d-x 3.2
##创建项目
官方文档
```
$ cd cocos2d-x
$ ./setup.py
$ source .bash_profile
$ cocos new MyGame -p com.your_company.mygame -l cpp -d NEW_PROJECTS_DIR
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
cocos run -p android -j4  (编译时间7分钟)
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


- quick如何缩小包体积
为了制作小包，特意给 quick-cocos2d-x 2.2.5plus 增加了模块化编译能力。
当禁用所有可选扩展后，libgame.so 的体积缩小到了惊人的 2780K，压缩到 apk 后仅占 1100K 字节（也就是 1M 多一点点）。
屏蔽的扩展包：




###eclipse环境编译




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

Particle Editor for Cocos2dx：http://games.v-play.net/particleeditor/


##音频





