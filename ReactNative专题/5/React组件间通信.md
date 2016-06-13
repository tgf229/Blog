
##组件间通信
###1)前言
---
因为RN的高度组件化特性，在开发的过程中，我们一定会碰到组件间是怎么通信的。
目前我碰到的基本是两种，一种是A包含B，即A页面中使用了B作为其子组件；一种是A打开B，作为Navigator
的一个新页面;

###2)分析
---
####2.1)A包含B

A中包含B，可传递props，也可传递state。
若传递的是state，则当A setState()的时候，B中的数据内容也会更新。
这在某些场景下很实用，如：A页面中包含顶部Banner，中间静态按钮和底部ListView。 
当整个页面需要下拉刷新时，即可在A中setState修改数据源，命令Banner组件跟进刷新。
```javascript
import B from './B';
export default class A extends Component {
    static defaultProps = {
        name: '涂高峰',
    }; 
    constructor(props){
        super(props);
        this.state = {
            a:'1',
        }
    }

    onClick(){
        this.setState({a:'2'});
    }

    render() {
        return (
        <View 
            style={styles.container}>
            <Text>我是A</Text>
            <B s={this.state.a} p={this.props.name} />
            <TouchableHighlight
                onPress={() => this.onClick()}
                underlayColor='#fcfcfc'
                style={{backgroundColor:'#ff902d',width:200,height:40,justifyContent:'center',alignItems:'center'}}>
                <Text>点击改变A的state</Text>
            </TouchableHighlight>
        </View>
        );
    }
}
```

####2.2)A打开B

A打开B，类似Android的startActivity()，使用Navigator即可简单的在打开B的同时传递参数。
``` javascript
this.props.navigator.push({
    component:B,
    params:{
        x:'11',
    }
});
```
但是，Navigator中使用pop()退出时无法传参，那项目中遇到需要在B中刷新A中的数据时应该如何操作呢？
类似Android的startActivityForResult()。
```javascript
this.props.navigator.pop();
```
这里提供两种方案：
方案一：A push 到B 的时候，将A的this通过props传递至B。
```javascript
refreshByData(){
    this.setState({a:'2'});
    console.log('xxx')
}

onClick(){
    this.props.navigator.push({
        component:B,
        params:{
            objA:this,
        }
    });
}
```
当B pop的时候，在B中用A对象setState重新render A。
```javascript
//    this.props.objA.refreshByData();  
this.props.objA.setState({a:'2'}); 
this.props.navigator.pop();
```

方案二：A定义方法 refreshByData(){}，当A push到B的时候传参params:{refreshA:this.refreshByData}
```javascript
refreshByData(){
    hello.setState({a:'2'});
    console.log('xxx')
}

onClick(){
    this.props.navigator.push({
        component:B,
        params:{
            refreshA:hello.refreshByData,
        }
    });
}
```
当B pop之前，在B中this.props.refreshA();
```javascript
onClick(){
    this.props.refreshA();
    this.props.navigator.pop();
}
```

###3) 进阶Redux
http://cn.redux.js.org/index.html
