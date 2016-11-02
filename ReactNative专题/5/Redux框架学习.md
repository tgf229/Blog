##Redux

###-redux是什么？
redux是一个用于管理js应用状态的容器。 在开发中随着业务量复杂度提高，需要维护的状态也越来越复杂，redux可以将所有变化进行统一流程处理，尤其是多人开发时极大的提高代码可读性，虽然会增加一些代码量。redux的最终目的就是让状态（state）变化变得可预测可管理，同时提高多人协作组件化开发的可读性。

###-redux三原则
####1 Single source of truth
单一的数据源。。整个应用的state，存储在唯一一个object中，同时也只有一个store用于存储这个object
####2 State is read-only
状态state是只读的。唯一能改变state的方法，就是触发action操作。action是用来描述正在发生事件的一个对象
####3 Changes are made with pure functions
在改变state tree时，用到action，同时也需要编写对应的reducers才能完成改变state的操作。

###-实例
此demo实现通过页面点击切换状态，和fetch获取远程数据并改变状态render页面

###Action：

~js/action/ActionType.js  定义事件常量
```javascript
export const FETCH_LIST = 'FETCH_LIST';
export const RECEIVE_LIST = 'RECEIVE_LIST';
```
~js/action/UserAction.js
```javascript
import * as TYPE from './ActionType';

const URL_ADDR = "http://58.213.145.35/";
const BUS_100201 = "Bus100201";

//远程fetch获取数据事件
export function fetchListData(){
return dispatch => {
dispatch(fetchList());	//先调用fetch的action，用于页面展示ActivityIndicatior
return fetch(URL_ADDR+BUS_100201)
.then(response => {
return response.json();
})
.then(json => {
console.log(json.retcode);
dispatch(receiveList(json.retcode)); //fetch成功后调用receive的action，用于隐藏ActivityIndicator，并展示远程获取的数据
})
.catch(error => {
console.log(error);
dispatch(receiveList('失败')); 
});
}
}

//fetch的action
export function fetchList(){
return {
type:TYPE.FETCH_LIST
}
}
//receive的action
export function receiveList(retcode,jsonPromise){
return {
type:TYPE.RECEIVE_LIST,
retcode:retcode
}
}
```

###Reducer:
~js/reducer/UserReducer.js
```javascript
import * as TYPE from '../action/ActionType';

//最开始的状态，会被注入
const userReducerState = {
loading : false,
retcode : '',
}

export function userReducer(state = userReducerState, action) {

switch (action.type) {
case 'FETCH_LIST':
return Object.assign({}, state, {
loading: true
});
break;
case 'RECEIVE_LIST':
return {
...state,
loading:false,
retcode:action.retcode
}
break;
default:
return state;
}
}
```
~js/reducer/RootReducer.js
```javascript
import {combineReducers} from 'redux';
import {userReducer} from './UserReducer';

//有多个reducer可以在此整合
const rootReducer = combineReducers({
userReducer,
})

export default rootReducer;
```

####Store:
~js/store/RootStore.js
```javascript
'use strict';

import {createStore, applyMiddleware} from 'redux';
import thunkMiddleware from 'redux-thunk';
import rootReducer from '../reducer/RootReducer';

const createStoreWithMiddleware = applyMiddleware(thunkMiddleware)(createStore);

export default function configureStore(initialState) {
const store = createStoreWithMiddleware(rootReducer, initialState);

return store;
}
```

####程序入口 index.ios.js
```javascript
import React, { Component } from 'react';
import {
AppRegistry,
StyleSheet,
Text,
View
} from 'react-native';
import UserContainer from './js/container/UserContainer';
import {Provider} from 'react-redux';
import configureStore from './js/store/RootStore';

//注册store
const store = configureStore();
//订阅监听器,dispatch会触发
store.subscribe(()=>{
console.log(store.getState())
});

export default class reduxDemo extends Component {

render() {
return (
<Provider store={store}>
<UserContainer/>
</Provider>
);
}
}
AppRegistry.registerComponent('reduxDemo', () => reduxDemo);
```

####业务逻辑container 用于实现业务逻辑，整合component，
~js/container/UserContainer.js
```javascript
import React, { Component } from 'react';
import {
AppRegistry,
StyleSheet,
Text,
View
} from 'react-native';
import { connect } from 'react-redux';
import UserComponent from '../component/UserComponent';
import * as TYPE from '../action/ActionType';
import {fetchListData,receiveList} from '../action/UserAction';

class UserContainer extends Component {

constructor(props) {
super(props);
this.state = {

};
}

componentDidMount() {
this.props.dispatch(fetchListData());
}

render() {
const { dispatch,store} = this.props
return (
<UserComponent store={store} onClick={()=>dispatch(receiveList('000001'))}/>
);
}
}

//获取store的state，并将其作为props传入自身
function mapStateToProps(store){
return{
store:store.userReducer,
}
}

//connect函数绑定，当store变化时，会反馈到此组件中
export default connect(mapStateToProps)(UserContainer);
```

####UI组件conponent，只有从container传入的props，事件也通过container的props传导
~js/component/UserComponent.js
```javascript
export default class UserComponent extends Component {

constructor(props) {
super(props);
this.state = {
};
}

render() {
const {store} = this.props
return (
<View style={styles.container}>

{
store.loading?  
<ActivityIndicator/>
:
<Text style={styles.welcome}>
大家好，获取的retcode是{store.retcode}
</Text>
}

<Text style={styles.instructions}>
redux测试，耐心等待接口返回 或 点击下面 ＝ Loading将消失
</Text>
<TouchableHighlight onPress={this.props.onClick}>
<Text style={styles.instructions}>
点击我 可以将Loading消失！！！
</Text>
</TouchableHighlight>
</View>
);
}
}
```

###-坑
遇到2个坑：
1，fetch在rn0.19版本所返回的response（可直接获取response的_bodyText获取返回数据）和rn0.36返回的response（必须链式根据response.json()的promise获取json）不一样，费解了半天，在@晴明大大的指导下，只是fetch内部实现机制的不同，代码正常写没有问题
2，fetch链式回调第二个then不触发，必须点击模拟器才触发，耗时一天，最后发现是chrome调试器的问题，关掉就正常。坑。

###-参考文献
文档
http://cn.redux.js.org    中文文档
http://www.ruanyifeng.com   阮一峰
实例
http://www.jianshu.com/p/2c43860b0532    			
http://www.tuicool.com/articles/7FZreu2

###-本例地址
~study/front-end