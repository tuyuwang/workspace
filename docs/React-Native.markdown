---
layout: default
title: React Native
nav_order: 10
---

## 随记

#### 导航栏

[react-navigation](https://reactnavigation.org/docs/en/getting-started.html)

#### 报错

Error: listen EADDRINUSE: address already in use:
~~~
sudo lsof -i:8081

sudo kill -9 $pid
~~~


Could not get BatchedBridge, make sure your bundle is packaged properly:
~~~
react-native start

react-native run-ios
~~~

react-native-getsure-handler:
~~~
npm install react-native-gesture-handler@latest --save
react-native link react-native-gesture-handler
~~~

指定模拟器:
~~~
 react-native run-ios --simulator="iPhone 11"
~~~

版本不对应: null is not an object (evaluating'_RNGestureHandlerModule.default.Direction'):
~~~
cd ios
pod install
cd ..
~~~