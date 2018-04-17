---
layout:     post
title:      "Android Studio 批量打包"
subtitle:   " \"Android Studio 批量打包 简易版\""
date:       2018-04-17 17:00:00
author:     "spyrx7"
catalog: true
tags:
    - android
---

##   简介
> 我想只要发布国内的安卓市场,需要使用友盟统计的同学们都会遇到这个问题,就是我们在打友盟的便签的时候,需要编译很多次,这个非常的麻烦.如果有一个批量的解决方案,那就完美了.

##   解决方案步骤
- 清单文件里配置变量(AndroidManifest.xml)
- 在项目App Gradle 里写入批量操作
- 执行编译


####   1 清单文件里配置变量(AndroidManifest.xml)

> 我们接入友盟统计通常我们都会在清单配置渠道名 ,我们这一步骤就是使用变量替换它.


``` java
  // 未替换变量
  <meta-data android:value="Google Play" android:name="UMENG_CHANNEL"/>
```

``` java
  // 已替换变量
  <meta-data android:value="${UMENG_CHANNEL_VALUE}" android:name="UMENG_CHANNEL"/>
```

####   2 在项目App Gradle 里写入批量操作

> 接下来我们到app.gradle 文件里(如果没有改名的话,是这个),在Android 这个标签里添加下面的代码.说明一下 UMENG_CHANNEL_VALUE
是在步骤一(AndroidManifest.xml)里定义的那个变量.至于命名,自己看着办吧

``` java

android {

    ...

    productFlavors{
        huawei{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "huawei"]
        }
        xiaomi{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "xiaomi"]
        }
        vivo{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "vivo"]
        }
        oppe{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "oppe"]
        }
        anzhi{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "anzhi"]
        }
        _189{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "_189"]
        }
        smartisan{
            manifestPlaceholders = [UMENG_CHANNEL_VALUE: "smartisan"]
        }
    }
        ...

  }
```

> 或者这样的 , 二选一

``` java
android {

    ...

    productFlavors{
            huawei{
                
            }
            xiaomi{
            
            }
            vivo{
                
            }
            oppe{
            
            }
            anzhi{
            
            }
            _189{
            
            }
            smartisan{
            
            }
    }

    productFlavors.all{
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE:name]
    }

    ...

}

```

####   3 执行编译

> 好的,这一步最简单了,我们Build > Generate Signed APK 然后选完我们要打包的渠道,开始编译,等待就好了.

![](http://junjianliu.cn/img/in-post/post-alitrip-pd/Androidstudiopiliang.png)