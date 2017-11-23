---
title: 你需要知道的 RN 导航知识
layout: post
guid: urn:uuid:14519a52-66d4-4a1b-b9a4-438738b75772
comments: true
tags:
  - react-native
---

作者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

# 预备知识
本文假设您熟悉 React、React Native 以及 Android 开发

# 混编配置
项目默认使用 Native 代码编写，RN 代码只在某些业务中使用

# 业务需求
覆盖 RN 和原生混编场景下不同的导航需求

一、跳转需求  
Native 界面跳转到 N 个 RN 界面，继续跳转到另一个 Native 界面，依次类推。

二、顶部导航栏返回键行为定制需求 (默认行为：返回上一个业务界面)  
1.Native 界面跳转到 RN 界面，又跳转了 N 个 RN 界面，在最后一个 RN 界面中直接返回之前压栈的 Native 界面  
2.Native 界面跳转到 RN 界面，又跳转了 N 个 RN 界面，在最后一个 RN 界面中直接返回某个之前压栈的 RN 界面

三、Android 端特有的物理返回键拦截需求  
我们需要保持顶部导航栏返回键行为和物理返回键行为一致，所以需要该功能。

# 方案调研
市面上主要的方案有三种：react-community 的 [react-navigation](https://github.com/react-community/react-navigation)、airbnb 的 [native-navigation](https://github.com/airbnb/native-navigation) 以及 wix 的 [react-native-navigation](https://github.com/wix/react-native-navigation)

### 实现原理

本文不专门对框架源码进行解析，只做大概介绍。

一、[react-navigation](https://github.com/react-community/react-navigation)  
该方案通过自定义 Component 容器管理所有的 RN 界面，这些界面都需要 Native 层提供壳 Activity 来呈现这些 ReactRootView。框架本身要求宿主 APP 提供 RN 运行环境。

二、[native-navigation](https://github.com/airbnb/native-navigation)  
此方案追求更高的 native 兼容性，所以实现上有一定的侵入性，Native 代码需要使用它封装的一些 Activity 组件和接口以及自定义布局。简单说，它是把所有的 RN 页面都转换成 Fragment，所有 Fragment 都贴在同一个 Activity 容器中，并且由自定义类处理 Fragment 跳转动画和栈的管理。而且顶部导航栏定制性不够高。

三、[react-native-navigation](https://github.com/wix/react-native-navigation)  
本方案实质上是由 Native 层的框架实现，JS 层只做了一些 Native API 的包裹。该框架必须在 JS 端注册所有业务 Screen Component，而且决定启动哪个 Screen 也是在 JS 端控制的。然后通过 RN Bridge 把需要启动的 Screen 参数传到 Native 层，框架的 Native 层解析之后，启动壳 Activity，渲染相应的 Screen Component 到 ReactRootView 上。顶部导航栏定制性同样不够高。

### 对比图

|   名称                   | 优点                            | 缺点                               |
|:------------------------|:--------------------------------|:----------------------------------|
| react-navigation        | 方便混编、纯 JS 实现，无 Native 代码侵入| 由于没有发正式版，Bug 比较多 |
| native-navigation       | 单页管理多个 Fragment，思路有借鉴意义   | Native 端侵入性强，顶部导航栏定制性不高 |
| react-native-navigation | 为纯 RN 编写的应用而生，渲染效果更好 | 强依赖 Native 端框架实现各种效果，顶部导航栏定制性不高|

### 最终决定

采用 [react-navigation](https://github.com/react-community/react-navigation) 比较适合我们混编的场景

# 框架集成

### 使用官网代码

方法一：在主工程目录下执行 `npm install --save react-navigation`  
方法二：直接在 package.json 中的 dependencies 对象中添加 `"react-navigation": "1.0.0-beta.20"`，然后在主工程目录下执行 `npm install` 即可  

### 使用私有仓库

把 package.json 中 dependencies 对象下 `react-navigation` 对应的版本号改为如下两种格式：  

HTTPS 方式：`git+https://git@host/<org or user>/<project>.git#<branch>`  
SSH 方式：`git+ssh://git@host/<org or user>/<project>.git#<branch>`  

以我的仓库为例：  
HTTPS Clone 地址：`https://github.com/johnwatsondev/react-navigation.git`  
SSH Clone 地址：`git@github.com:johnwatsondev/react-navigation.git`  

修改结果如下：  
HTTPS 方式：`git+https://git@github.com/johnwatsondev/react-navigation.git#master`  
SSH 方式：`git+ssh://git@github.com/johnwatsondev/react-navigation.git#master`  

# 功能示例

### react-navigation 提供了三种导航样式：  
StackNavigator --- 和 Android 系统跳转 Activity 方式效果一致 (launchMode = "standard")  
TabNavigator --- 同 Android 系统的 TabActivity 效果类似  
DrawerNavigator --- 与 Android 系统的 DrawerLayout 效果相同

**特别说明：**  
我们仅以 StackNavigator 为例讲解，其他样式暂不解释。

### RN 端构建界面

我们封装了 StackNavigator 的组件如下：

```javascript
Code-Segment-1

class Navigation extends Component {

  constructor(props) {
    super(props)
  }

  render() {
    let Navigator = StackNavigator({
      Home: {
        screen: MyHomeScreen,
      },
      Profile: {
        path: 'people/:name',
        screen: MyProfileScreen,
      },
      Photos: {
        path: 'photos/:name',
        screen: MyPhotosScreen,
      },
      UseSystemBackBehaviourScreen: {
        screen: UseSystemBackBehaviourScreen
      },
    }, 
    {
      initialRouteName: this.props.myProps.SCREEN,
      mode: 'card'
    })
    console.log('Navigation', this.props)
    return (
      <Navigator screenProps={this.props}/>
    )
  }
}
```

上述代码中，`Home/Profile/Photos/UseSystemBackBehaviourScreen` 称为 routeName，每一个 Screen 又叫做 route。

注册到 index.js 中的代码如下：

```javascript
// https://github.com/react-community/react-navigation/issues/876
class MyApp extends React.Component {
  render () {
    console.log('this.props in MyApp', this.props) // This will list the initialProps.
    // StackNavigator **only** accepts a screenProps prop so we're passing
    // initialProps through that.
    return <Navigation myProps={this.props} />
  }
}

AppRegistry.registerComponent('ReactNativeNavigationDemo', () => MyApp);
```

上面的代码中，关键代码是 `<Navigation myProps={this.props} />` ，通过 `myProps` 把 Native 解析 bundle 得到的对象，传递到我们封装的 Navigation 组件中，继而通过 `<Navigator screenProps={this.props}/>` 传递到 StackNavigator 中，也就传递给了每一个业务 Screen。

在每个业务 Screen 中调用我们的参数方法是：`this.props.screenProps.myProps.XXX` (PS: XXX 代表 Native 端 bundle 中传入的 key)

### 一、跳转需求  

![RN 混编跳转图示](http://ozqz8ena7.bkt.clouddn.com/RN%20%E6%B7%B7%E7%BC%96%E8%B7%B3%E8%BD%AC%E5%9B%BE%E7%A4%BA.png)

上图中：
A 为 Native Activity、B/C/D 为 RN Screen、E 为 Native Activity

#### A 启动 B/C/D 任意一个

直接在 Native 启动装载 RN 的壳 Activity 即可。唯一要注意的地方是我们一般要动态启动 B、C、D 中某一个 Activity。那么需要动态传递参数到 RN JS 代码中决定启动哪个 Screen。

```java
public static final String SCREEN = "SCREEN";
public static final String HOME = "Home";

public static Intent callHomeScreenIntent(Context context) {
  Intent intent = callIntent(context);
  intent.putExtra(RNConfig.SCREEN, RNScreens.HOME);
  return intent;
}
```

在 Activity A 中直接调用上面的 Intent，我们会把要启动的 Screen 名称 `Home` 传递到 RN 代码中。通过在 *Code-Segment-1* 代码中的 `initialRouteName: this.props.myProps.SCREEN` 指定初始化 Screen，由此实现动态唤起某个已注册的 RN Screen。

#### B 启动 B/C/D 任意一个

在 RN StackNavigator 内部跳转很简单，在业务 Screen 组件中使用 `this.props.navigation.navigate('Your Screen Route Name', {XXX: 'XXX'}` 即可。

该方法第一个参数意义为：我们在 StackNavigator 中注册的 Screen 的 routeName。例如，我们在 *Code-Segment-1* 中注册的 `Home/Profile/Photos` 其中任何一个。
该方法第二个参数意义为：我们向目标 Screen 传递的参数对象，格式为标准的 JS Object。在业务 Screen 中通过 `this.props.navigation.state.params` 获取该对象。

更多资料请参考 [Screen-Navigation-Prop](https://reactnavigation.org/docs/navigators/navigation-prop#Screen-Navigation-Prop)

#### B/C/D 任意一个启动 D

只需要通过 NativeModule 调用 Native Activity 即可，代码为：`InvokeJumpToApp.navigateToActivityD()`

RN 代码 `InvokeJumpToApp.js` 如下：

```javascript
import {NativeModules} from 'react-native'

const native = NativeModules.InvokeJumpToApp

export default {
  navigateToActivityD: () => {
    return native.navigateToActivityD()
  }
}
```

NativeModule 代码 `InvokeJumpToAppModule.java` 如下：

```java
public class InvokeJumpToAppModule extends ReactContextBaseJavaModule {

  private static final String MODULE_NAME = "InvokeJumpToApp";

  public InvokeJumpToAppModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  @Override public String getName() {
    return MODULE_NAME;
  }

  @ReactMethod public void navigateToActivityD(Promise promise) {
    final Activity currentActivity = getCurrentActivity();

    if (currentActivity == null) {
      PromiseHintUtil.rejectOfActivityDoesNotExist(promise);
      return;
    }

    // This method was invoked in a separate thread instead of UIThread.
    // So we need to invoke startActivity method in UIThread.
    UiThreadImmediateExecutorService.getInstance().execute(new Runnable() {
      @Override public void run() {
        currentActivity.startActivity(ActivityD.callIntent(currentActivity));
      }
    });
  }
}
```

### 二、顶部导航栏返回键行为定制需求 (默认行为：返回上一个业务界面)

框架默认不支持定制返回键的行为，但是支持替换自己的 header。所以，我们只能定制整个 header。

由于代码较多，请直接查看 [ActionBar](https://github.com/johnwatsondev/ReactNativeNavigationDemo/blob/master/app/bar/ActionBar.js)。

我们称需要定制顶部导航栏返回键行为的 Screen 为 目标 Screen。

大体思路是：我们可以在 Native 启动 RN Screen，或者在 RN Screen 启动 RN Screen 时，给目标 RN Screen 中传递参数，然后传入自定义 header 中，由此可以动态控制该 RN Screen 顶部导航栏返回键的行为。

#### Native 界面跳转到 RN 界面，又跳转了 N 个 RN 界面，在最后一个 RN 界面中直接返回之前压栈的 Native 界面

![RN 界面直接返回之前压栈的 Native 界面](http://ozqz8ena7.bkt.clouddn.com/RN%20%E7%95%8C%E9%9D%A2%E7%9B%B4%E6%8E%A5%E8%BF%94%E5%9B%9E%E4%B9%8B%E5%89%8D%E5%8E%8B%E6%A0%88%E7%9A%84%20Native%20%E7%95%8C%E9%9D%A2.png)

目标 RN Screen 代码如下：

```javascript
const UseSystemBackBehaviourScreen = ({navigation}) => (
  <SafeAreaView>
    <SampleText>{'Click back key to return previous native activity'}</SampleText>
  </SafeAreaView>
)

UseSystemBackBehaviourScreen.navigationOptions = props => {
  const {navigation, screenProps} = props

  let sysBack = false
  if (navigation.state.params && typeof navigation.state.params.sysBack !== 'undefined') {
    sysBack = navigation.state.params.sysBack
  }
  return {
    title: 'Photos',
    header: <ActionBar
        nav={navigation}
        screenProps={screenProps}
        data={{
          title: '返回上一个 Native 界面',
          sysBack,
        }}
    />,
    backPressedListener: () => {
      console.log('UseSystemBackBehaviourScreen physical back key pressed event listener invoked...')
      InvokeBackKey.back()
      return true;
    }
  }
}

export default UseSystemBackBehaviourScreen
```

在某个 RN 界面中启动该目标 RN Screen 时调用代码如下：  

```javascript
this.props.navigation.navigate('UseSystemBackBehaviourScreen', {sysBack: true})
```

#### Native 界面跳转到 RN 界面，又跳转了 N 个 RN 界面，在最后一个 RN 界面中直接返回某个之前压栈的 RN 界面

![RN 界面直接返回之前压栈的 RN 界面](http://ozqz8ena7.bkt.clouddn.com/RN%20%E7%95%8C%E9%9D%A2%E7%9B%B4%E6%8E%A5%E8%BF%94%E5%9B%9E%E4%B9%8B%E5%89%8D%E5%8E%8B%E6%A0%88%E7%9A%84%20RN%20%E7%95%8C%E9%9D%A2.png)

这个需求官方已经有了[解决办法](https://reactnavigation.org/docs/navigators/navigation-prop#goBack-Close-the-active-screen-and-move-back)，但是用法上有点特别，例如现在 RN Screen 的导航栈为 `B -> C -> D`,我们要从 D 返回 B，并且 C、D 全部销毁，需要在 D 中调用 `navigation.goBack(SCREEN_KEY_C)`，`SCREEN_KEY_C` 是框架为 C Screen 分配的唯一 key，这个 key 是包含 `id-` 的字符串。

符合使用习惯的方法是：直接传入 B Screen 的 routeName 即可。

所以，我们要修改源码，以便指定某个 routeName 就跳转对应的 Screen。

##### 解决方案：

修改 `StackRouter.js` 的 getStateForAction 方法中一段代码即可：

```javascript
if (action.type === NavigationActions.BACK) {
  const key = action.key;
  let backRouteIndex = null;
  if (key) {
    let foundTargetRouteName = false;
    const backRoute = state.routes.find((route: NavigationRoute) => {
      if (route.routeName === key) {
        foundTargetRouteName = true;
      }
      return route.key === key || route.routeName === key;
    });
    /* $FlowFixMe */
    backRouteIndex = state.routes.indexOf(backRoute);

    if (foundTargetRouteName) {
      // if found the desired route by routeName, we need back to this screen
      backRouteIndex += 1;
    }

    if (backRouteIndex === -1) {
      // if not found the desired route, popping current screen
      backRouteIndex = null;
    }
  }
  if (backRouteIndex == null) {
    return StateUtils.pop(state);
  }
  if (backRouteIndex > 0) {
    return {
      ...state,
      routes: state.routes.slice(0, backRouteIndex),
      index: backRouteIndex - 1,
    };
  }
}
```

##### 调用方式

和原先一样，只要把传入的 key 变为 routeName 即可。  
例如，我们要跳转到 *Code-Segment-1* 中声明的 MyHomeScreen，调用 `this.props.navigation.goBack('Home')` 就可以跳转到该 Screen。

### 三、Android 端特有的物理返回键拦截需求

官方源码在 [createNavigationContainer.js](https://github.com/react-community/react-navigation/blob/master/src/createNavigationContainer.js#L155) 的 `componentDidMount()` 方法中添加了事件监听器。默认情况下，点击物理返回键会返回上一个 RN Screen。

```
this.subs = BackHandler.addEventListener('hardwareBackPress', () =>
  this.dispatch(NavigationActions.back())
);
```

我们没有更好的办法，只能修改源码，去掉这个容器层的默认处理，交由每个 Screen Component 来管理。

一个有意思的问题是：如果现在启动了三个 RN 界面，如果每个 Screen Component 都去监听 BackHandler 物理返回键事件，谁优先去响应呢？所以，合适的思路是我们在 Screen Component 的某个上层组件拦截物理返回键事件，然后根据当前的导航状态来确定当前正在展示的目标 Screen Component，把这个事件分发给该 Screen Component。这个上层组件就是 `CardStack`，因为每一个 StackNavigator 组件都会包含一个 `CardStack`。在 `CardStack` 中我们可以获得正确的目标 Screen Component。

#### 解决方案：

1.修改 `createNavigationContainer.js` 中 `componentDidMount()` 和 `componentWillUnmount()` 两个方法：

```javascript
componentDidMount() {
  if (!this._isStateful()) {
    return;
  }

  // FIXME: 21/11/2017 remove default back pressed process behaviour
  // this.subs = BackHandler.addEventListener('hardwareBackPress', () =>
  //   this.dispatch(NavigationActions.back())
  // );

  Linking.addEventListener('url', this._handleOpenURL);

  Linking.getInitialURL().then(
    (url: ?string) => url && this._handleOpenURL({ url })
  );
}

componentWillUnmount() {
  Linking.removeEventListener('url', this._handleOpenURL);
  // this.subs && this.subs.remove();
}
```

2.在 CardStack.js 中添加两个方法即可：  

```javascript
componentDidMount() {
  if (Platform.OS === 'android') {
    this.subs = BackHandler.addEventListener('hardwareBackPress', () => {
      const { navigation, scene } = this.props;
      const { backPressedListener } = this._getScreenDetails(scene).options;

      // 被嵌套的 CardStack 组件应该优先响应返回按键，例如 CardStack A 包含 CardStack B，那么应该由 B 响应。
      // 如何区别这个 CardStack 是否是最内层的呢？根据 scene.route.routes 是否存在判断。
      // 如果是最内层 CardStack，没有 routes 对象，否则存在该对象。

      if (!scene.route.routes) {
        if (typeof backPressedListener === 'function') {
          return backPressedListener();
        } else {
          return navigation.dispatch(NavigationActions.back());
        }
      }
    });
  }
}

componentWillUnmount() {
  if (Platform.OS === 'android') {
    this.subs && this.subs.remove();
  }
}
```

#### RN 中的业务 Screen 如何拦截呢？  
只需要在 Screen Component 中的 静态 `navigationOptions` 对象中指定给 `backPressedListener` 一个 function 即可。
代码如下：

```javascript
YourRNScreenComponent.navigationOptions = ({navigation}) => {
  return {
    backPressedListener: () => {
      // navigation.goBack(null);
      // return true if you want process physical back key pressed event
      return true;
    }
  }
};
```

或者另一种写法  

```javascript
class YourRNScreenComponent extends Component {
  constructor (props) {
    super(props)
  }
  
  static navigationOptions = ({navigation, screenProps}) => {
    return {
      backPressedListener: () => {
        // navigation.goBack(null);
        // return true if you want process physical back key pressed event
        return true;
      }
    }
  }
  
  render () {
    return (<View></View>)
  }
}
```

**注意：** backPressedListener 的值必须是 function，而且要返回布尔类型。如果我们要处理该事件就返回 true，否则返回 false。

# StackNavigator 工作原理

我们定义的每一个 `StackNavigator` 都是由 N 个 route 页面构成的。而我们所有的页面都存储在导航容器的静态 router 中。我们可以理解成 router 保存了所有我们要渲染的 Screen Component 所有信息。

![StackNavigator Component 封装结构](http://ozqz8ena7.bkt.clouddn.com/StackNavigator%20Component%20%E5%B0%81%E8%A3%85%E7%BB%93%E6%9E%84.png)

整个导航库可以理解成一个俄罗斯套娃，其中使用了大量的 HOC (Hight Order Component，可以理解为 Java 中的抽象类)，每一层 HOC 封装一些功能。比如 `NavigationContainer` 封装了每个 route 中使用的 navigation 对象，我们所有的命令都是通过该对象来接收和响应的。`Navigator` 主要是把 router 对象传递给具体的实现Component。`CardStackTransitioner` 使用组合的方式封装了实现导航 Screen 之间动画的 `Transitioner`，还有 `Transitioner` 中渲染的 `CardStack`。简单说，`CardStackTransitioner` 渲染抽象的 `Transitioner`，`Transitioner` 渲染具体的 `CardStack`。`CardStack` 中渲染 `Card`，`Card` 中通过 `SceneView` 来真正渲染我们传入的 Screen Component。

当然每次不是渲染所有的 Screen Component。框架会通过 Action 计算出我们需要渲染的 Screen。

最外层的 navigation 对象中包含一个 state 对象，该对象会在 navigation 对象的 dispatch 方法中更新，该 state 对象会保存需要渲染的所有 Screen Component 信息。

`NavigationContainer` 分为有状态和无状态两种，如果该容器是顶层 Component ，那么它就是有状态的。我们知道这个导航库支持嵌套，有的 `NavigationContainer` 会作为一个 Screen Component 存在，此时这个 `NavigationContainer` 就是无状态的，它不会创建自己的 navigation 对象来管理导航状态。从 React Component 的角度来理解，子组件的状态只能来自父组件，也就是说，无论是 `NavigationContainer` 还是 Nested `NavigationContainer` 中的某个 Screen 要执行某个 Action ，这个 Action 只能由 最外层的 `NavigationContainer` 中创建的 navigation 对象的 dispatch 方法来接收，由此触发整个 Component Tree 的状态刷新。

# 代码示例
[ReactNativeNavigationDemo](https://github.com/johnwatsondev/ReactNativeNavigationDemo)

# 致谢
感谢您阅读本文，如果您有任何问题，请留言或者发送到 `johnwatsondev.com@gmail.com`。