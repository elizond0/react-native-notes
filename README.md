# react-native-notes

## 0. 简介

* 从2018/7月开始接触RN及其社区插件，个人填坑记录

## 1. react-native

## 2. Android Studio

## 3. RN社区组件

### 3.1 react-native-AlphabetListView

* react-native-AlphabetListView，此控件是利用RN内置的Section和ListView组件，模仿的手机上通讯录的UI，所用版本号0.3.0，目前[原作者](https://github.com/elizond0/react-native-alphabetlistview)已经不更新，因此[fork](https://github.com/elizond0/react-native-alphabetlistview)后直接修改源码作为项目备份。

* BUG-1：在右侧字母列表不是垂直撑满的情况下，手势拖拽滑动到列表外会产生红屏报错
1. 原因：没有判断变量是否存在就,length
2. 解决方案：components\SectionList.js/#63

* BUG-2：在左侧列表项内容小于容器高度时，点击右侧字母列表后，左侧内容跳会转到底部，预期应该不进行滚动
1. 原因：源码计算高度有负数
2. 解决方案：components/SelectableSectionsListView.js/#125
