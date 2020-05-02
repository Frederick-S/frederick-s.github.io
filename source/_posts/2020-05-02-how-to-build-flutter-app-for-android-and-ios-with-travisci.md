title: 使用 Travis CI 为 Flutter 项目打包 Android/iOS 应用 
tags:
- Flutter
---

[Building Flutter APKs and IPAs on Travis](https://medium.com/@yegorj/building-flutter-apks-and-ipas-on-travis-98d84d8e9b4) 这篇文章详细介绍了如何在 Travis CI 上为 Flutter 项目打包 Android/iOS 应用，不过实际构建时存在几个问题，原文中的 `.travis.yml` 配置如下：

```
matrix:
  include:
    - os: linux
      language: android
      licenses:
        - 'android-sdk-preview-license-.+'
        - 'android-sdk-license-.+'
        - 'google-gdk-license-.+'
      android:
        components:
          - tools
          - platform-tools
          - build-tools-25.0.3
          - android-25
          - sys-img-armeabi-v7a-google_apis-25
          - extra-android-m2repository
          - extra-google-m2repository
          - extra-google-android-support
      jdk: oraclejdk8
      sudo: false
      addons:
        apt:
          # Flutter depends on /usr/lib/x86_64-linux-gnu/libstdc++.so.6 version GLIBCXX_3.4.18
          sources:
            - ubuntu-toolchain-r-test # if we don't specify this, the libstdc++6 we get is the wrong version
          packages:
            - libstdc++6
            - fonts-droid
      before_script:
        - wget http://services.gradle.org/distributions/gradle-3.5-bin.zip
        - unzip -qq gradle-3.5-bin.zip
        - export GRADLE_HOME=$PWD/gradle-3.5
        - export PATH=$GRADLE_HOME/bin:$PATH
        - git clone https://github.com/flutter/flutter.git -b alpha --depth 1
      script:
        - ./flutter/bin/flutter -v build apk

    - os: osx
      language: generic
      osx_image: xcode8.3
      before_script:
        - pip install six
        - brew update
        - brew install --HEAD libimobiledevice
        - brew install ideviceinstaller
        - brew install ios-deploy
        - git clone https://github.com/flutter/flutter.git -b alpha --depth 1
      script:
        - ./flutter/bin/flutter -v build ios --no-codesign

cache:
  directories:
    - $HOME/.pub-cache
```

## Android
### wget - 403 Forbidden
这个错误发生在执行 `wget http://services.gradle.org/distributions/gradle-3.5-bin.zip` 的时候，把 `gradle` 的下载路径替换成 `https` 即可。

### Remote branch alpha not found in upstream origin
这个错误发生在下载 `Flutter` 代码的阶段，原文中的配置会下载 `Flutter` 的 `alpha` 分支代码，但是目前 `Flutter` 的仓库已经没有 `alpha` 分支，切换到 `stable` 分支即可，即：`git clone https://github.com/flutter/flutter.git -b stable --depth 1`。

### Failed to install the following Android SDK packages as some licences have not been accepted
详细错误信息如下：

```
[        ] > Failed to install the following Android SDK packages as some

licences have not been accepted.

[        ]      build-tools;28.0.3 Android SDK Build-Tools 28.0.3

[        ]      platforms;android-29 Android SDK Platform 29

[        ]   To build this project, accept the SDK license agreements and

install the missing components using the Android Studio SDK Manager.
```

这个错误是由于没有同意 `Android SDK` 的许可证协议，在 `before_script` 中加入如下配置即可：

```
yes | sdkmanager "platforms;android-29"
yes | sdkmanager "build-tools;28.0.3"
```

## iOS
### pip: command not found
这个错误在执行 `pip install six` 时遇到，经过实际验证构建 `iOS` 应用时并不需要此行配置，所以删掉即可。

### Xcode 11.0 or greater is required to develop for iOS
原文中的配置使用的是 `Xcode 8.3`，最后打包时会提示此错误，将 `osx_image` 设置为 `xcode11` 即可。

最后完整可用的 `.travis.yml` 配置如下：

```
matrix:
  include:
    - os: linux
      language: android
      licenses:
        - 'android-sdk-preview-license-.+'
        - 'android-sdk-license-.+'
        - 'google-gdk-license-.+'
      android:
        components:
          - tools
          - platform-tools
          - build-tools-25.0.3
          - android-25
          - sys-img-armeabi-v7a-google_apis-25
          - extra-android-m2repository
          - extra-google-m2repository
          - extra-google-android-support
      jdk: oraclejdk8
      sudo: false
      addons:
        apt:
          # Flutter depends on /usr/lib/x86_64-linux-gnu/libstdc++.so.6 version GLIBCXX_3.4.18
          sources:
            - ubuntu-toolchain-r-test # if we don't specify this, the libstdc++6 we get is the wrong version
          packages:
            - libstdc++6
            - fonts-droid
      before_script:
        - wget https://services.gradle.org/distributions/gradle-3.5-bin.zip
        - unzip -qq gradle-3.5-bin.zip
        - export GRADLE_HOME=$PWD/gradle-3.5
        - export PATH=$GRADLE_HOME/bin:$PATH
        - git clone https://github.com/flutter/flutter.git -b stable --depth 1
        - yes | sdkmanager "platforms;android-29"
        - yes | sdkmanager "build-tools;28.0.3"
      script:
        - ./flutter/bin/flutter -v build apk

    - os: osx
      language: generic
      osx_image: xcode11
      before_script:
        - brew update
        - brew install --HEAD libimobiledevice
        - brew install ideviceinstaller
        - brew install ios-deploy
        - git clone https://github.com/flutter/flutter.git -b stable --depth 1
      script:
        - ./flutter/bin/flutter -v build ios --no-codesign

cache:
  directories:
    - $HOME/.pub-cache
```

参考：

- [Building Flutter APKs and IPAs on Travis](https://medium.com/@yegorj/building-flutter-apks-and-ipas-on-travis-98d84d8e9b4)
- [Failed to install the following Android SDK packages as some licences have not been accepted in jitpack](https://stackoverflow.com/questions/54273412/failed-to-install-the-following-android-sdk-packages-as-some-licences-have-not-b)
