#1: 理解`bind`方法

```JavaScript
// 这是Prototype.js中的`.bind`方法
Function.prototype.bind = function(){ 
  // fn绑定调用的函数；args保存着函数参数转为数组对象；object保存着第一个参数（即为绑定`this`的对象）
  var fn = this, args = Array.prototype.slice.call(arguments), object = args.shift(); 
  return function(){ 
    //注：这里是返回一个函数，所以有使用call/apply使用的时候也不会影响起先bind的`this`值
    return fn.apply(object, 
      args.concat(Array.prototype.slice.call(arguments))); 
  }; 
};
// 实际运用中
var name = "window"

var baiji = {
  name:"baiji"
}

function sayName(){
  console.log(this.name)
}
sayName()    //logs window
sayName.bind(baiji)() //logs baiji
sayName.bind(baiji).call(null)  //logs baiji
```
