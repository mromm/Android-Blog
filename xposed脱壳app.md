### 现在的app大多使用了360加固、腾讯加固、爱加固等加固工具，目的是增加代码的安全性，加大破解的难度，不过魔高一丈道高一尺，有些`互联网爱好者`终究耐不住寂寞，喜欢做些挑战的事情，于是市面上各式各样的破解方法、破解工具诞生，幸运的是让我也能一睹其风采，哈哈。以上纯属自娱自乐，下面进入本章正题：
#### 如何使用xposed破解加固apk
 1. 首先准备必要的工具和软件  
   [夜神模拟器6.2.3.0](https://www.yeshen.com/)  
   [xposedInstaller](https://github.com/sunshey/Android-Blog/blob/master/de.robv.android.xposed.installer_v33_36570c.apk)  
   [FDex2_1.1.apk](https://github.com/sunshey/Android-Blog/blob/master/FDex2_1.1.apk)  
   [JustTrustMe](https://github.com/sunshey/Android-Blog/blob/master/JustTrustMe.apk)  
 2. 打开夜神模拟器，将以上apk安装，打开xposed，点击模块，勾选其他两个软件  
 ![xposed截图](https://github.com/sunshey/Android-Blog/blob/master/QQ%E6%88%AA%E5%9B%BE20190418151521.png)
 3. 安装要破解的apk，打开FDEx2，选择要破解的apk，点击,
 ![FDEx2截图](https://github.com/sunshey/Android-Blog/blob/master/QQ%E6%88%AA%E5%9B%BE20190418151741.png)
 4. 点击需要破解的apk后，会弹出一个提示框，记住提示框的路径
 ![提示框路径](https://github.com/sunshey/Android-Blog/blob/master/QQ%E6%88%AA%E5%9B%BE20190418152417.png)
 5. 重启夜神模拟器，再次打开要破解的apk
 6. 通过adb命令连接夜神模拟器，命令如下
 ```
 adb connect 127.0.0.1:62001
 ```
 7. 连接完成后，通过adb命令检查夜神模拟器是否已连接，如连接，则通过一下命令拉取数据：
 ```
 adb devices(检查设备连接)
 adb -s 127.0.0.1:62001 pull /data/data/xxxxx(这里是前面要记住的路径)
 ```
 8. 通过jadx-gui查看dex文件，破解大功告成  
 ![查看文件](https://github.com/sunshey/Android-Blog/blob/master/QQ%E6%88%AA%E5%9B%BE20190418153353.png)
 
 
