## 单例模式

定义：保证一个类只有一个实例，并提供一个访问它的全局访问点

用例：登录浮窗，单击登录按钮时，页面中会出现登录浮窗，无论点击多少次登录按钮，浮窗只会创建一次

实现：

第一版

```js
var Singleton=function(name){
    this.name=name;
    this.instance=null;
}

Singleton.prototype.getName=function(){
    alert(this.name);
}

Singleton.getInstance=function(name){
    if(!this.instance){
        this.instance=new Singleton(name);
    }
    return this.instance;
}

var a=Singleton.getInstance('Doe');
var b=Singleton.getInstance('John');

alert(a===b);   //true;
```

