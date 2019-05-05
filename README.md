# react-native-notes

## 0. 简介

* 从2018/7月开始接触RN及其社区插件，个人填坑记录

## 1. react-native

### 1.1 RN常见报错警告收集

#### 1.1.1 Prevent React setState on unmounted Component

* Warning: Can only update a mounted or mounting component. This usually means you called setState, replaceState, or forceUpdate on an unmounted component. This is a no-op.

* Warning: Can’t call setState (or forceUpdate) on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.

todo https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component/

## 2. Android Studio

## 3. RN社区组件

### 3.1 [react-native-AlphabetListView](https://github.com/elizond0/react-native-alphabetlistview)

* react-native-AlphabetListView，此控件是利用RN内置的Section和ListView组件，模仿的手机上通讯录的UI，所用版本号0.3.0，目前[原作者](https://github.com/i6mi6/react-native-alphabetlistview)已经不更新，因此fork后直接修改源码作为项目备份。

* BUG-1：在右侧字母列表不是垂直撑满的情况下，手势拖拽滑动到列表外会产生红屏报错
1. 原因：没有判断变量是否存在就,length
2. 解决方案：components\SectionList.js/#63

* BUG-2：在左侧列表项内容小于容器高度时，点击右侧字母列表后，左侧内容跳会转到底部，预期应该不进行滚动
1. 原因：源码计算高度有负数
2. 解决方案：components/SelectableSectionsListView.js/#125

### 3.2 [react-native-image-gallery](https://github.com/elizond0/react-native-image-gallery)

* BUG-1：组件的initialPage属性设置后，组件没有更新列表状态，
1. 原因：即没有触发DidmountHandler中的操作
2. 解决方案：src/libraries/ViewPager/index.js/#125

* BUG-2：第一次滑动后图片回到初始加载页面
1. 原因：runAfterInteractions传入的方法，并没有在页面初始化后立即执行，而是在第一次左右滑动时才执行
2. 临时解决方案：src/libraries/ViewPager/index.js/#247 去除runAfterInteraction的包装

* BUG-3：仅安卓出现此bug IOS正常，未开始渲染的元素通过initialPage打开时，页面只会显示已经加载的图片
1. 原因：安卓下Gallery组件无法正确计算未渲染图片的宽度offset
2. 解决方案：gallery组件传入getItemLayout方法，预先传入宽度，同时pageMargin属性会影响布局
3. 后续问题-3.1：传入getItemLayout会使flatlist触发bug，无法正常渲染列表项
4. 后续问题-3.1的解决方案：使用mobx触发组件再次渲染，进入页面时重新绘制

* 性能问题-4：安卓左右滑动有厚重感
1. 临时解决方案：MIN_FLING_VELOCITY是速率，降低速率提高灵敏度，src/libraries/ViewPager/index.js/#247

## 4. [react-navigation](https://github.com/react-navigation/react-navigation)

* 由于react-navigation是react官方御用导航插件，因此提取出来单独记录

* 需求-1：ios和android的页面切换动画保持一致，都采用水平切换的方式，默认是ios水平，安卓垂直
1. 环境："react-navigation": "^2.5.5","react": "16.0.0","react-native": "0.51.0",
2. 解决方案：在2.5.5中官方虽未在文档中列出，但react-navigation的社区中merge了相关的解决方案
3. 后续问题-1.1：ios中从页面最边上水平滑动可以实现浏览器向前向后的操作，而android中则没有这个滑动事件的手势
4. 后续问题-1.1的解决方案：主要是机型的问题，部分安卓机有左划回退的手势

```js
// 首先在路由文件中引入
import StackViewStyleInterpolator from "react-navigation/src/views/StackView/StackViewStyleInterpolator";
// 然后在路由配置选项中进行配置
const RootNavigation = StackNavigator(
    {
        ...pageRoots
    },
    {
        transitionConfig: ()=> {
            // 解决需求：mode.card属性，安卓默认是上下方向，IOS默认是左右，这里统一成水平方向
            return {screenInterpolator: StackViewStyleInterpolator.forHorizontal}
        },
    }
)

```

* BUG-2：react-navigation会对路由进行层级管理并具有缓存策略，如果页面跳转时等同于goback操作，那页面不会进行重绘
1. 解决方案：官方提供了监听事件this.props.navigation.addListener('willFocus',() => {// 重新获得数据的方法体})，在willmount句柄中挂载可以对导航栏的变化进行监听
2. 后续问题-2.1：在navigation进行页面切换时，如果点击速度较快，在导航动画没有完成时就进行了页面跳转，那addListener事件willFocus会挂载失败
3. 后续问题-2.1的解决方案：在willmount句柄中调用两次重新取数据的方法，并且通过自定义的status避免执行两次。goback的页面中有接收参数的，也需要对接受参数并赋值的方法进行限制，避免页面闪烁，如果用到了promise的then()方法，可以设置多个status根据需要来解决

```js
constructor(props){
    this.isPageFirstRendering=true
}

componentWillMount(){
    this.pageInit()
    this.props.navigation.addListener('willFocus',() => {
        this.pageInit()
    })
}

pageInit(){
    if(this.isPageFirstRendering){
        this.isPageFirstRendering=false
        this.getData()
    }
}

getData(){
    // 获取数据的方法
}
```

* BUG-3：react-navigation页面跳转时来回点击跳转，会导致生命周期执行bug
1. 具体现象：例如A => B => A，正常的生命周期执行应该是A-willUnmount => B-willmount。点击间隔小于800ms时：A-willUnmount => B-willmount =>A-willunmount。
2. 过程分析：从A跳转到B页面时，A准备开始卸载，然后B开始挂载，正确生命周期与错误bug复现过程的区别在于，A页面在B开始挂载后又执行了一次卸载操作。
3. 临时解决方案：对页面跳转进行控制，经过测试可得800ms是安全间隔，怀疑是react-navigation的动画过程引起的。
