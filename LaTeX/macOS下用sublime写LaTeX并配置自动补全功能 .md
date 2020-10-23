来自：[macOS下用sublime写LaTeX并配置**自动补全**功能](http://cache.baiducontent.com/c?m=B7D6AHQEwa7oedB5Z7RGUVNR0YiXUFdlLpUEQ-ambLzv_dPt5r-ZofjhwzvigwFyXIDJGysaU2GJCLnwFXiC2ayYmVZXs7-pf2lMipsHdluuFcldB7N1H1h-CX2uEOXPIVuqHUsqxTuHgfFdeo_ACq&p=9b71c815d9c247be44a2c7710f00&newp=b4759a44d09c50e810be9b7c465792695d0fc20e3ad4d001298ffe0cc4241a1a1a3aecbf2d281000d4c2766d05ad4c5ae1f534703d0034f1f689df08d2ecce7e7099607c&s=cfcd208495d565ef&user=baidu&fm=sc&query=texshop+%D7%D4%B6%AF%B2%B9%C8%AB&qid=b11f1f31000608d8&p1=3)

---



原来大家都有**自动补全**，，就我一直用**TeXShop**手打LaTeX，，所以给我最喜欢的sublime也配一个写LaTeX的环境

------

# 1.安装LaXeT语言环境

```
shift+commond+p`进入框框，输入`install package
```

![图片.png](https://upload-images.jianshu.io/upload_images/843214-ce88cf50df8f8634.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在框里输入`latextools`（因为我已经装好这个插件了所以下图里不再显示）

![图片.png](https://upload-images.jianshu.io/upload_images/843214-2499247ab4705c5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重启sublime，`【菜单栏】-【Tools】-【Build System】`，就可以发现语言环境里已经有LaTeX了

![图片.png](https://upload-images.jianshu.io/upload_images/843214-d4954bb963eb0292.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2.**自动补全**

先像上面一个打开install package的框，输入latexcwl然后点它

![图片.png](https://upload-images.jianshu.io/upload_images/843214-0fca9581d34d9c76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样输入的时候就有快乐补全功能了

![图片.png](https://upload-images.jianshu.io/upload_images/843214-1de2af91429e1f57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 3.配置pdf预览

接下来配置 pdf 预览部分，打开 sublime 设置，`【sublime text-preference-settings】`

![图片.png](https://upload-images.jianshu.io/upload_images/843214-e7ceab5c488cef7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 user 自定义的部分中设置预览器为 mac 自带的预览 preview

```
"viewer": "preview"
```

![图片.png](https://upload-images.jianshu.io/upload_images/843214-a9fdba3d4cdca08b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在打开一个`.tex`文件，就可以用`shift+command+b`自动编译运行了，会跳出来一个下图所示的编译选项，选择对应的即可，比如有中文所以我选的 XeLaTeX

![图片.png](https://upload-images.jianshu.io/upload_images/843214-ce62f9e54813fe0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

撒花～

![图片.png](https://upload-images.jianshu.io/upload_images/843214-922057f3e1cedb86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

---

「完」

