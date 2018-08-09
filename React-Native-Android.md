---
title: React-native打包apk
tags:
- react-native 
- android 
- apk
---

通过`create-react-native-app`创建的项目默认推荐使用expo来配合开发，并不需要进行打包直接就能上手开发了很方便，但如果想把开发的成果共享给别人最方便他人的就是打包后发给别人。andorid打包为apk后缀的包后发给别人直接就可以安装了，IOS就麻烦点特别是没有过原生开发经验的打包过程太痛苦了。我就是没有任何APP开发经验， 期间遇到了很多问题整理出这篇文章，给后来着少走点弯路，这里先只介绍Andorid的打包过程，每步操作都给出了参考的文档。

## 执行eject

如果你按照`react-native`官网的教程使用`create-react-native-app`创建的项目，那默认是没有`ios`和`android`这两个目录的，需要使用`eject`来生成这两个目录，如果已经有这两个目录请跳过。 [参考文档](https://github.com/react-community/create-react-native-app/blob/master/EJECTING.md)

```bash
yarn run eject
# 期间会让输入两个东西。第一个是apk安装后显示的名称，第二个是包的标示，已经有前缀com。
```

## 签名配置

Andorid的需要app通过证书进行数字签名，作为业余的APP开发其中的原理并不清楚，只能照做了😅  [参考文档](https://facebook.github.io/react-native/docs/signed-apk-android)

1. 生成签名key

   ```bash
   keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
   # 按提示创建 my-release-key.keystore 文件
   ```

2. 拷贝上一步创建的`my-release-key.keystore`文件至`android/app`目录下

3. 编辑`android/gradle.properties`文件，添加如下内容

   ```bash
   vi android/gradle.properties
   
   MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
   MYAPP_RELEASE_KEY_ALIAS=my-key-alias
   MYAPP_RELEASE_STORE_PASSWORD=******		# 替换为生成key时设置的密码
   MYAPP_RELEASE_KEY_PASSWORD=******		# 替换为生成key时设置的密码
   ```

4. 编辑`android/app/build.gradle`添加签名相关配置

   ```bash
   vi android/app/build.gradle
   
   ...
   android {
       ...
       defaultConfig { ... }
       signingConfigs {
           release {
               if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                   storeFile file(MYAPP_RELEASE_STORE_FILE)
                   storePassword MYAPP_RELEASE_STORE_PASSWORD
                   keyAlias MYAPP_RELEASE_KEY_ALIAS
                   keyPassword MYAPP_RELEASE_KEY_PASSWORD
               }
           }
       }
       buildTypes {
           release {
               ...
               signingConfig signingConfigs.release
           }
       }
   }
   ...
   ```
   

## 生成APK

接下来就可以进行打包了，默认生成的apk位于`android/app/build/outputs/apk/app-release.apk`

```bash
cd android
./gradlew assembleRelease
```

## 后续问题

如果需要集成推送功能，请参考下边的极光推送集成方法。

#### 减小APK包体积

- 根据不同CPU单独打包，编辑`android/app/build.gradle`

  ```bash
  vi android/app/build.gradle
  
  def enableSeparateBuildPerCPUArchitecture = true   # 把该值设置为true
  ```

- 启用混淆功能

  ```bash
  vi android/app/build.gradle
  
  def enableProguardInReleaseBuilds = true  # 把该值设置为true
  ```

#### 处理图标字体文件

如果你使用了`react-native-vector-icons`包的字体图标，例如我这里用了`native-base`依赖了该包，那么启动打包的app后可能提示缺少字体的问题，可以按照如下方法解决 [参考文档](https://github.com/oblador/react-native-vector-icons)

```bash
vi android/app/build.gradle  # 编辑添加如下内容

project.ext.vectoricons = [
    iconFontNames: [ 'Ionicons.ttf' ]
]

apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
```


#### 极光推送

[参考文档](https://github.com/jpush/jpush-react-native)

1. 安装依赖包

   ```bash
   yarn add jpush-react-native jcore-react-native
   ```

2. 自动配置

   ```bash
   ./node_modules/.bin/react-native link	# 过程中需要输入Jpush的AppKey
   ```

3. 手动配置

   [参考文档](https://github.com/jpush/jpush-react-native/blob/master/documents/android_usage.md)

   ```bash
   vi android/app/src/main/java/com/${application}/MainApplication.java	
   # ${application}替换成包的名字
   
   ...
   import cn.jpush.reactnativejpush.JPushPackage;   // <--   导入 JPushPackage
   
   public class MainApplication extends Application implements ReactApplication {
   
       // 设置为 true 将不会弹出 toast
       private boolean SHUTDOWN_TOAST = false;
       // 设置为 true 将不会打印 log
       private boolean SHUTDOWN_LOG = false;
   
       private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
   
   		...
   		
           @Override
           protected List<ReactPackage> getPackages() {
               return Arrays.<ReactPackage>asList(
                       new MainReactPackage(),
                       new JPushPackage(SHUTDOWN_TOAST, SHUTDOWN_LOG)   //  <-- 添加 JPushPackage
                );
           }
       };
   
     ...
   }
   
   ```

4. 配置App.js

   ```bash
   vi App.js
   
   
   import JPushModule from 'jpush-react-native';
   
   ...
   
     componentDidMount() {
   	JPushModule.initPush();
       JPushModule.notifyJSDidLoad(() => {});
       JPushModule.addReceiveCustomMsgListener((message) => {
         this.setState({pushMsg: message});
       });
       JPushModule.addReceiveNotificationListener((message) => {
         console.log("receive notification: " + message);
       })
     }
   
     componentWillUnmount() {
       JPushModule.removeReceiveCustomMsgListener();
       JPushModule.removeReceiveNotificationListener();
     }
   ```

   
