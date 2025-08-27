这篇文章主要是我们团队在使用Cocos Creator过程中的一些关于部署方面的实践总结，标题党了一回，严格来说，应该是**《快看漫画游戏研发团队使用Cocos Creator构建部署最佳实践》**，对于其他团队可能并不是。

之所以写这篇文章，一是我刚开始接触Cocos Creator的时候，发现构建部署方面的一些问题，针对性写了3篇优化的方案，随着对Cocos Creator了解的深入，我发现了一些更好的替代方法，二是因为我们团队随着业务发展，又到了缺人的时候，出来刷刷脸，发点招聘广告：[Cocos Creator工程师快到碗里来](http://forum.cocos.com/t/cocos-creator/54157)。

不过你不用担心，本文不会只是炒炒冷饭，这次涉及的内容覆盖了构建部署的整个环节，如果你刚好把代码写好了，你应该看看本文，它会告诉你怎么把你的代码漂亮的部署到线上，并且这里涉及的代码你都可以通过github查看。涉及知识点如下：

1. 如何自定义loading页面
2. 图片部署自动化压缩优化
3. 减少loading页面出现之前的白屏时间
4. 代码混淆与保护
5. 文件资源增加md5版本号
6. cdn缓存

由于我们游戏采用的是1.6版本，所以还保留了md5版本号的优化，新的1.7版本已经比较完美支持md5的功能，但由于没有在实践中使用，所以还是基于1.6版本做一次总结，本质是一样的。

本篇我们会基于[Cocos Creator的官方示](https://github.com/cocos-creator/demo-ui)例做分析，我在原demo的基础上增加了部署的脚本，部署到又拍云和腾讯云。为了展示自定义loading页面的功能，我把这个游戏的loading页面改了，如果有问题，麻烦官方联系我下架。

### **1. 如何自定义loading页面**

这个需求官方其实是有提供解决方案的，官方文档-“[定制项目构建模板](http://docs.cocos.com/creator/manual/zh/publish/custom-project-build-template.html)”的功能就可以实现这个需求，可能是文档描述得不太清楚，我当时并没有把这个功能跟“自定义加载首页”联系起来。

由于这个构建模板功能，我们就可以不用gulp插件轻松实现自定义首页HTML,CSS,JS的功能了。怎么实现可以访问本文的项目地址查看。

**自定义loading页面效果：**

![WX20171213-221317@2x.png](http://upload-images.jianshu.io/upload_images/3360354-2d1f4cfd5015a997.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/360)



### **2. 图片部署自动化压缩优化**

通过gulp工具，在部署之前自动化处理一遍图片压缩流程，在无损压缩的情况下，既能保证图片的输出质量还能减少体积。有团队会手动采用tinypng或者其他压缩工具提前压缩，这样做也是可以的，但流程不好把控，原则上能自动化处理的尽量让机器来做，人是会累的但机器不会，也不会出错。

```
var imagemin = require("gulp-imagemin");
gulp.task("imagemin", function (cb) {
    gulp.src(["./build/web-mobile/**/*.png"])
        .pipe(imagemin([
            imagemin.gifsicle({interlaced: true}),
            imagemin.jpegtran({progressive: true}),
            imagemin.optipng({optimizationLevel: 5})
        ]))
        .pipe(gulp.dest("./build/web-mobile/"))
        .on("end", cb);
});`
```

### **3. 减少loading页面出现之前的白屏时间**

通过gulp-htmlmin插件，把首屏的js，css文件合并到首页html文件，能有效减少网络不稳定情况下进入游戏白屏的时间。

```
var htmlmin = require("gulp-htmlmin");
gulp.task("htmlmin", ["imagemin"], function (cb) {
    gulp.src("./build/web-mobile/*.html")
        .pipe(fileInline())
        .pipe(htmlmin({
            collapseWhitespace: true,
            removeComments: true,
            minifyCSS: true
        }))
        .pipe(gulp.dest("./build/web-mobile/")
            .on("end", cb));
});
```
通过合并操作，首屏loading页面只需要加载index.html文件，在304情况下白屏时间只有142ms！

![WX20171214-101911@2x.png](http://upload-images.jianshu.io/upload_images/3360354-44bbb2aa5dbdde19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **4. 代码混淆，代码保护**

Cocos Creator引擎build后会对代码进行压缩优化，但通过强大的chrome工具格式化代码后，还是能轻松阅读代码的整体逻辑，在竞争激烈的游戏行业，代码保护力度是不足够的。由于代码暴露在前端，H5游戏不存在加密可言，但我们可以做一些工作，增加游戏被破解盗窃的难度。

[gulp-javascript-obfuscator](https://github.com/javascript-obfuscator/javascript-obfuscator)插件可以对代码进行可读性混淆，禁止开启chrome调试，域名绑定等功能，能很大程度保护自己的代码，这个插件还有其他很强大的功能，有兴趣可以访问github了解。但我不建议开启太多功能，毕竟对性能还是有一定影响。

```
var javascriptObfuscator = require("gulp-javascript-obfuscator");
gulp.task("obfuscator", ["htmlmin"], function (cb) {
    gulp.src(["./build/web-mobile/project.js"])
        .pipe(javascriptObfuscator({
            compact: true,
            domainLock: [".zz-game.com"],
            mangle: true,
            rotateStringArray: true,
            selfDefending: true,
            stringArray: true,
            target: "browser"
        }))
        .pipe(gulp.dest("./build/web-mobile")
            .on("end", cb));
});
```

我采用了最轻量的混淆方案，混淆前后对比：

**混淆前：**

```
TabCtrl: [function(t, e, i) {
        "use strict";
        cc._RF.push(e, "62208XJq9ZC2oNDeQGcbCab", "TabCtrl"),
        cc.Class({
            extends: cc.Component,
            properties: {
                idx: 0,
                icon: cc.Sprite,
                arrow: cc.Node,
                anim: cc.Animation
            },
            init: function(t) {
                this.sidebar = t.sidebar,
                this.idx = t.idx,
                this.icon.spriteFrame = t.iconSF,
                this.node.on("touchstart", this.onPressed.bind(this), this.node),
                this.arrow.scale = cc.p(0, 0)
            }
        }),
        cc._RF.pop()
    }
    , {}]
}, {}, ["ItemList", "ItemTemplate", "BackPackUI", "ButtonScaler", "ChargeUI", "EnergyCounter", "HeroSlot", "HomeUI", "PanelTransition", "ShopUI", "SubBtnsUI", "MainMenu", "MenuSidebar", "TabCtrl"]);
```

**混淆后：**

```
'l': [function(b, a, c) {
        'use strict';
        cc[_0xc008('0x12')][_0xc008('0x13')](a, _0xc008('0x98'), 'l'),
        cc['T']({
            'S': cc['U'],
            'O': {
                'idx': 0x0,
                'icon': cc[_0xc008('0x3e')],
                'arrow': cc['Node'],
                'anim': cc['Animation']
            },
            '_': function(a) {
                this[_0xc008('0x68')] = a[_0xc008('0x68')],
                this['idx'] = a[_0xc008('0x7b')],
                this[_0xc008('0x66')][_0xc008('0x47')] = a[_0xc008('0x65')],
                this[_0xc008('0x1b')]['on'](_0xc008('0x99'), this[_0xc008('0x9a')][_0xc008('0x5e')](this), this[_0xc008('0x1b')]),
                this[_0xc008('0x9b')][_0xc008('0x9c')] = cc['p'](0x0, 0x0);
            }
        }),
        cc[_0xc008('0x12')][_0xc008('0x20')]();
    }
    , {}]
}, {}, ['i', 'd', 'f', 'c', 'j', 'b', 'h', 'n', 'a', 'm', 'g', 'k', 'e', 'l']);
```

在不影响性能的前提下，稍微做一些代码保护，还是不错的。如果你想让代码更恶心一点也是可以的：

```
a.DFsJp;
        cc[a[_0xc91c('0x43a')](_0x490d30, a[_0xc91c('0x43b')])][a[_0xc91c('0x43c')](_0x490d30, _0xc91c('0xe5'))](b, a[_0xc91c('0x43c')](_0x490d30, a['\x68\x6d\x41\x4c\x54']), '\x6c'),
        cc['\x54']({
            'S': cc['\x55'],
            'O': {
                'idx': 0x0,
                'icon': cc[_0x490d30(_0xc91c('0x249'))],
                'arrow': cc['\x4e\x6f\x64\x65'],
                'anim': cc[a[_0xc91c('0x43d')]]
            },
            '_': function(b) {
                this[a[_0xc91c('0x43e')](_0x490d30, _0xc91c('0x31c'))] = b[a[_0xc91c('0x43e')](_0x490d30, '\x30\x78\x36\x38')],
                this[a[_0xc91c('0x43f')]] = b[_0x490d30(a[_0xc91c('0x440')])],
                this[_0x490d30(a[_0xc91c('0x441')])][_0x490d30(a['\x6b\x77\x71\x63\x72'])] = b[_0x490d30(a[_0xc91c('0x442')])],
                this[_0x490d30(_0xc91c('0x1c0'))]['\x6f\x6e'](a[_0xc91c('0x43e')](_0x490d30, a[_0xc91c('0x443')]), this[a['\x45\x79\x7a\x47\x44'](_0x490d30, a['\x6a\x64\x44\x61\x6e'])][_0x490d30(_0xc91c('0x2e0'))](this), this[a[_0xc91c('0x444')](_0x490d30, _0xc91c('0x1c0'))]),
                this[a[_0xc91c('0x445')](_0x490d30, _0xc91c('0x446'))][a[_0xc91c('0x445')](_0x490d30, a[_0xc91c('0x447')])] = cc['\x70'](0x0, 0x0);
```

**开启debugProtection功能:**

![image.png](http://upload-images.jianshu.io/upload_images/3360354-1925bf48b3c2a093.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开chrome调试工具就会触发无限循环的dubugger，让chrome调试工具无法使用，增大破解难度。为了方便大家查看学习demo，线上版本我关闭了这个选项。

**开启disableConsoleOutput功能：**

![image.png](http://upload-images.jianshu.io/upload_images/3360354-c638a748e82bfb27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

禁止console.log功能，很多混淆代码，通过断点+console.log，可以方便把翻译后的代码输出，开启disableConsoleOutput，同样增加调试难度。为了把我们的广告无处不在，我同样把它关了。JavaScript obfuscator还有其他不错的功能，这里不再展开。

### 5. 文件资源增加md5版本号

版本号的方案跟之前的文章基本一致，这个流程在1.7版本应该可以忽略了。

```
gulp.task("resRev", ["obfuscator"], function (cb) {
    gulp.src(["./build/web-mobile/**/*.js", "./build/web-mobile/*.png"])
        .pipe(rev())
        .pipe(gulp.dest("./build/web-mobile/"))
        .pipe(rev.manifest())
        .pipe(gulp.dest("./build/web-mobile/")
            .on("end", cb));
});
gulp.task("default", ["resRev"], function (cb) {
    del(["./build/web-mobile/src"]);
    gulp.src(["./build/web-mobile/*.json", "./build/web-mobile/index.html"])
        .pipe(revCollector())
        .pipe(gulp.dest("./build/web-mobile/"));
    gulp.src(["./build/web-mobile/*.json", "./build/web-mobile/main*.js"])
        .pipe(revCollector({
            replaceReved: true
        }))
        .pipe(gulp.dest("./build/web-mobile/")
            .on("end", cb));
});
```
通过md5+强缓存，第二次加载基本是毫秒级，瞬开。

**116个请求的页面，只需要109ms就能渲染出loading页面，完全加载所有资源只需要1.25s：**

![image.png](http://upload-images.jianshu.io/upload_images/3360354-be9a4b02754f9ab1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### **6. cdn缓存**

最后是把代码部署到cdn，现在的云服务都提供cdn分发的功能，通过简单配置，我相信你能折腾出来的，所以不再赘述。

这里主要做不同方案的演示，我部署了两个方案：直接回源和cdn分发。

**首次访问**

在Wifi网络下，回源方案耗时6-10s，cdn分发方案耗时3-6s。

**第二次访问**

由于增加了强缓存，无论是cdn还是回源，第二次访问时间都在1-2s之间。

这个项目本身存在先天不足，例如图片没有合并，导致首次请求有116个，加载速度肯定会受影响。但通过cdn缓存方案，也能基本保证快速加载。

**又拍cdn方案：**

![image.png](http://upload-images.jianshu.io/upload_images/3360354-a5a1a623457f7108.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

腾讯回源方案：

![image.png](http://upload-images.jianshu.io/upload_images/3360354-6855b90e6df88af8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码我已经部署到了又拍云和腾讯云，大家可以点击访问感受加载速度。

[直接回源的部署方案，点击访问](http://tx-ori.zz-game.com/card/index.html)

[采用cdn等优化方案，点击访问](http://yp-cdn.zz-game.com/card/index.html)

cdn是比较好的优化首次访问网络速度的方案，但cdn也不是必然比源站快，大家测试时发现回源更快也不必惊讶，本质上cdn节点就是距离你更近的代理服务器，但也有很多情况导致cdn缓慢，所以部署后还要通过工具多测试各个cdn节点的状况。

### 最后

游戏优化肯定不仅仅这几条，有很多优化要根据实际情况实际分析。但这6点实践，应该可以解决论坛经常提到的缓存刷新，加载速度等部署相关的问题。

**资源md5+cdn+强缓存** 能解决80%H5游戏加载速度的问题，特别是第二次访问，2秒打开轻轻松松，基本已经成为web前端优化的工业标准方案。但现在很多线上的H5游戏还有很多走**url参数+时间戳/md5**的老方案，这种方案有很多弊端。希望通过本文，大家都能在自己的游戏内把**资源md5+cdn+强缓存**的方案贯彻执行起来。其实很简单，特别是Cocos Creator1.7版本后就更方便了。H5游戏的优势就是即点即玩，如果这点都做不到，就没什么优势了。

说了这么多，你可能觉得实践起来很麻烦，业务太多没时间搞这些。

没关系，本文买一送一，既然你看到这里，说明你也是有缘之人，我把代码仓库也赠送给你，例子源码我放在了github [cocos-fly](https://github.com/babyzone2004/cocos-fly)上，有需要大家可以上去下载。

把gulpfile.js和releash.sh扒下来，只需要执行命令：`sh releash`，就可以一键构建出可发布的代码。

线上示例+源码+教程一条龙，还不赶紧引入自己的项目？。