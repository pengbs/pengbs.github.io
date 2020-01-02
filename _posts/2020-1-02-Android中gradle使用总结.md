---
layout:     post
title:      "Android中gradle使用总结"
subtitle:   ""
date:       2020-01-02 15:19:00
author:     "HeyBoy"
tags:
    - Android
    - Gradle
---

## 概述
android中使用gradle的一些要点总结，做下记录.

## 全局统一库版本
开发一个app，肯定会引入很多第三方库或者module，如果里面引入的第三方库版本不一致就会导致冲突报错，一般情况下是会在报错的那里exclude排除一下重复的，如下：

    implementation (group: 'com.facebook.react', name: 'react-native-fast-image', version:'5.1.1-jitsi-877932'){
        exclude group: 'com.facebook.react'
        exclude group: 'com.android.support'
        exclude group: 'com.github.bumptech.glide'
    }

但是这样就会导致每引入一个第三方库可能就得排除一下，非常麻烦,同步是官方的`com.android.support`库；其实可以在gradle里面配置一下，强制使用统一的版本，在项目的根`build.gradle`里面添加如下配置：


	subprojects {
    	configurations.all {
        	resolutionStrategy {
           	  eachDependency { details ->
                // Force all of the primary support libraries to use the same version.
                if (details.requested.group == 'com.android.support'
                        && details.requested.name != 'multidex'
                        && details.requested.name != 'multidex-instrumentation') {
                    details.useVersion rootProject.ext.version["androidSupportSdkVersion"]
                }
                // Force all the error-prone dependencies to use the same version.
                if (details.requested.group == 'com.squareup.okhttp3') {
                    details.useVersion rootProject.ext.version["okhttpVersion"]
                }
             }
          }
       }
    }

这样全局的support和okhttp就是使用的统一版本,其他的也可以照此仿照;

## 统一管理第三方库版本
多module项目场景下，每个module需要引入各自的第三方库依赖，有可能就依赖了重复的第三方库，这样不方便使用统一的版本和管理，为此可以做如下操作：
1. 在项目根目录新建一个`config.build`文件，里面定义参考如下：

       ext {
    	android = [
               compileSdkVersion       : 27,
               buildToolsVersion       : "28.0.2",
               minSdkVersion           : 21,
               targetSdkVersion        : 27,
               versionCode             : 34,
               versionName             : "1.1.25"
    	]
     	version = [
                   androidSupportSdkVersion: "27.1.1",
                   glideSdkVersion         : "4.8.0",
                   espressoSdkVersion      : "3.0.1",
                   canarySdkVersion        : "1.5.4",
                   okhttpVersion           : "3.12.3",
        ]
    	dependencies = [
                //support
                "appcompat-v7"             : "com.android.support:appcompat-v7:${version["androidSupportSdkVersion"]}",
                "design"                   : "com.android.support:design:${version["androidSupportSdkVersion"]}",
                "support-v4"               : "com.android.support:support-v4:${version["androidSupportSdkVersion"]}",
                "cardview-v7"              : "com.android.support:cardview-v7:${version["androidSupportSdkVersion"]}",
                "annotations"              : "com.android.support:support-annotations:${version["androidSupportSdkVersion"]}",
                "recyclerview-v7"          : "com.android.support:recyclerview-v7:${version["androidSupportSdkVersion"]}",
                "exifinterface"            : "com.android.support:exifinterface:${version["androidSupportSdkVersion"]}",

                //network
                "okhttp3"                  : "com.squareup.okhttp3:okhttp:${version["okhttpVersion"]}",
                "okhttp-urlconnection"     : "com.squareup.okhttp3:okhttp-urlconnection:${version["okhttpVersion"]}",
                "glide"                    : "com.github.bumptech.glide:glide:${version["glideSdkVersion"]}",
                "glide-compiler"           : "com.github.bumptech.glide:compiler:${version["glideSdkVersion"]}",
                "glide-loader-okhttp3"     : "com.github.bumptech.glide:okhttp3-integration:${version["glideSdkVersion"]}",
                "picasso"                  : "com.squareup.picasso:picasso:2.5.2",

                //view
                "photoview"                : "com.github.chrisbanes:PhotoView:2.1.4",
                "subsamplingScaleView"     : "com.davemorrissey.labs:subsampling-scale-image-view:3.10.0",
                "android-gif-drawable"       : "pl.droidsonroids.gif:android-gif-drawable:1.2.15",

                //tools
                "eventbus"                 : "org.greenrobot:eventbus:3.1.1",
                "gson"                     : "com.google.code.gson:gson:2.8.5",
                "fastjson"                 : "com.alibaba:fastjson:1.1.70.android",
                "multidex"                 : "com.android.support:multidex:1.0.2",
         ]
	   }

2. 在项目根目录的`build.gradle`的最上方加入下面这行，引用这个`config.gradle`文件:

		apply from: "config.gradle"

3. 在各自module里面引用第三方库的时候如下引用即可：

		#引用版本		
		compileSdkVersion rootProject.ext.android["compileSdkVersion"]
        #引用v7库
		implementation rootProject.ext.dependencies['appcompat-v7']

这样以后要升级第三方库的版本就可以统一在`config.build`文件里面修改

## 依赖下载失败处理

因为墙和网络的原因，gradle依赖同步每次都很慢而且非常容易失败，这种我们可以在gradle中配置国内阿里云的镜像源来规避，在项目的根`build.gradle`里面配置如下：

	buildscript {
    	repositories {
        	maven {url "https://maven.aliyun.com/repository/public"}
        	maven {url "https://maven.aliyun.com/repository/google"}
        	mavenCentral()
        	maven {
            	url 'https://www.jitpack.io'
        	}
        	google()
        	jcenter()
    	}
	}

加入

	maven {url "https://maven.aliyun.com/repository/public"}
    maven {url "https://maven.aliyun.com/repository/google"}

这两行即可，具体可参考:[https://maven.aliyun.com/mvn/view](https://maven.aliyun.com/mvn/view "阿里云镜像")
