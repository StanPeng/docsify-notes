## 概念

### CSS选择器分类

1. 选择器   css声明块前的标签、类名等
2. 选择符 后代关系 空格( )，父子关系 (>) ,相邻兄弟关系(+),兄弟关系(~),列关系(||)
3. 伪类 前面有一个冒号(:)，和用户行为相关联  :hover
4. 伪元素 两个冒号(::),如::before,::after,::firsr-letter,::first-line

### 无效CSS选择器

无效CSS选择器和浏览器支持的CSS选择器写在一起时，会导致整个选择器无效

```css
/* IE浏览器不支持导致全部失效*/
.example:hover,
.example:focus-within{
    color:red
}
```

可修改为渐进增强的写法

```css
/* IE浏览器可识别 */
.example:hover{
    color:red
}

/* IE浏览器不可识别 */
.example:focus-within{
    color:red
}
```

如果加上-webkit-私有前缀，浏览器就可以识别

```css
/* 无效 */
div,span::whatever{
    background:grey
}

/* 生效 */
div,span::-webkit-whatever{
    background:grey
}
```

## 优先级

1. 0级：通配符（*），选择符（+，>,~,空格,| |），逻辑组合伪类(:not(), :is() , :where )
2. 1级：标签选择器
3. 2级：类选择器，属性选择器和伪类。
4. 3级：id选择器
5. 4级：style属性内联
6. 5级：!important

### 优先级计算规则

数值计数法：每一段CSS选择器对应一个具体的数值 ，出现一个0级0级+0 ，1级+1，2级+10

```css
a:not([rel=nofollow]) /* 11=1+0+10,1个1级标签选择器(a),1个0级否定伪类(:not()) 1个2级属性选择器([rel=nofollow)*/
li.foo.bar /* 21=10+10=1 2个2级类名选择器(.foo .bar),1个标签选择器(li)*/
```

当CSS选择器优先级数值一样时，后渲染的选择器优先级更高

小技巧：

要增加CSS选择器优先级时，如果采用嵌套，会增加耦合，降低代码的可维护性，可以选择重复选择器自身，`.foo.foo{}`,或借助存在的属性选择器`.foo[class]{}`,`#foo[id]{}`

**注意**

数值计数法不严谨，不同等级的选择器之间的差距是不可跨越的存在。

## 选择器命名

| 选择器类型        | 示例       | 是否大小写敏感 |
| :---------------- | ---------- | -------------- |
| 标签选择器        | div{}      | 不敏感         |
| 属性选择器-纯属性 | [attr]     | 不敏感         |
| 属性选择器        | [attr=val] | 属性值敏感     |
| 类选择器          | .container | 敏感           |
| ID选择器          | #container | 敏感           |

### 选择器设计的最佳实践

1. 不要用ID选择器

   原因：（1）优先级太高，重置样式困难

   ​			（2）和JavaScript耦合

2. 不要嵌套选择器

   原因：（1）渲染性能糟糕 ,CSS选择器从右往左进行匹配渲染  `.box>div`先匹配页面所有的`<div>`元素，再匹配`.box`类名元素

   ​		   （2）优先级混乱，重置某些样式困难，优先级选择的原则：尽可能保持低优先级

   ​		   （3）样式布局脆弱  调整HTML结构时整个CSS文件要同步修改

## CSS选择符

### 后代元素选择符空格（  ）

有意思的题

```html
<div class="lightblue">
    <div class="darkblue">
        <p>
            1.颜色是?
        </p>
    </div>
</div>
<div class="darkblue">
    <div class="lightblue">
        <p>
            2.颜色是?
        </p>
    </div>
</div>
```



```css
.lightblue p{color:lightblue}
.darkblue  p{color:darkblue}
```

1和2将都是深蓝色

```css
:not(.darkblue) p {color:lightblue;}
.darkblue p {color:darkblue}
```

1和2都是深蓝色

### 子选择符箭头（>)

### 相邻兄弟选择符（+）

相邻兄弟选择符会忽略文本节点与注释

相邻兄弟选择器实现聚焦框后面有提示

```html
<input><span class="cs-tips">不超过10个字符</span>
```

```css
:focus + .cs-tips{
    visibility:visible
}
```

### 随后兄弟选择符(~)

相邻兄弟选择符只会匹配它后面的第一个兄弟元素，而随后兄弟选择符会匹配后面的所有兄弟元素

## 元素选择器

包括两类，一类是标签选择器，一类是通配符选择器

### 级联语法

1. 元素选择器是唯一不能重复自身的选择器

   ```css
   /*有效*/
   .foo.foo{}
   #foo#foo{}
   [foo][foo]{}
   
   /*无效*/
   foo*foo{}
   ```

2. 级联使用的时候元素选择器必须写在最前面

   ```css
   /*无效*/
   [type='radio']input{}
   
   /*有效*/
   input[type='radio']{}
   ```

小技巧

借助<html>和<body>元素提高优先级

通配选择器(*)匹配所有元素，比较消耗性能，且容易出现一些意料之外的样式问题，谨慎使用

## 属性选择器

### id选择器和类选择器

### 属性直接匹配选择器

1. [attr]

   只要包含指定属性就匹配

   适用于HTML布尔属性，只要有属性值，无论值是什么都认为是true

   如`disabled`

   判断输入框是否禁用，直接使用`[disabled]{}`,无需关系具体属性值

   与之相对应的是`checked`属性，表单控件元素在`checked`状态变化时不会同步修改`checked`属性的值

   ```js
   checkbox.checked=false;
   checkbox.disabled=false
   ```

   在html中将会

   `<input id="checkbox" type="checkbox" checked>`

   `disabled`消失了，`checked`还在

   实际开发中一般使用`:checked`这些伪类

2. [attr="val"]

   属性值完全匹配选择器

   不区分单引号和双引号

   引号可以省略，如果属性值包含空格，则需要转义

   `<input value="20">`

   `[value="20"]{}` 可匹配

   如果改变输入值为10，无论手动输入还是`JavaScript`更改，依旧按照`[value="20"]`渲染

   除非使用`input.setAttribute('value'),10`,属性选择器会按照`[value="10"]`渲染

3. [attr~="val"]

   属性值单词完全匹配选择器，专门用来匹配属性中的单词

   最佳实践

   ```html
   <div data-align="left top"></div>
   <div data-align="top"></div>
   <div data-align="right top"></div>
   <div data-align="right"></div>
   <div data-align="right bottom"></div>
   <div data-align="bottom"></div>
   <div data-align="left bottom"></div>
   <div data-align="left"></div>
   <div data-align="center"></div>
   ```

   ```css
   [data-align]{left:50%;top:50%;}
   [data-align~="top"]{top:0;}
   [data-align~="right"]{right:0;}
   [data-align~="bottom"]{bottom:0;}
   [data-align~="left"]{left:0;}
   ```

4. [attr|="val"]

   属性值起始片段完全匹配选择器

   表示具有`attr`属性的元素，其值要么正好是`val`,要么以`val`,要么以`val`外加短横线-(U+002D)开头

```html
<!--匹配-->
<div attr="val"></div>
<!--匹配-->
<div attr="val-ue"></div>
<!--匹配-->
<div attr="val-ue bar"></div>
<!--不匹配-->
<div attr="value"></div>
<!--不匹配-->
<div attr="val bar"></div>
<!--不匹配-->
<div attr="bar val-ue"></div>
```

实践：

匹配中文类或英文类语言

```css
/* 匹配中文类语言 */
[lang|="zh"]
/* 匹配英文类语言 */
[lang|="en"]
```

### AMCSS开发模式

`Attribute Modules for CSS`的缩写，表示借助`HTML`属性来进行`CSS`相关开发

主流开发模式

`<button class="cs-button cs-button-large cs-button-blue">按钮</button>`

AMCSS则是基于属性控制，例如

`<button button="large blue">按钮</button>`

为了避免属性冲突可以给属性添加一个统一前缀

`<button am-button="large blue">按钮</button>`

选择器可表示为

```css
[am-button]{}
[am-button~="large"]{}
[am-button~="blue"]{}
```

### 属性值正则匹配选择器

#### [attr^="val"]

表示匹配`attr`属性值以字符`val`开头的元素

```html
<!--匹配-->
<div attr="val"></div>
<!--不匹配-->
<div attr="text val"></div>
<!--匹配-->
<div attr="value"></div>
<!--匹配-->
<div attr="val-ue"></div>
```

实践

```css
/*链接地址*/
[href^="http"],[href^="ftp"],[href^="//"]{}
/*网页内锚链*/
[href^="#"]{}
/*手机和邮箱*/
[href^="tel:"]{}
[href^="mailto:"]{}
```

#### [attr$="val"]

```html
<!--匹配-->
<div attr="val"></div>
<!--匹配-->
<div attr="text val"></div>
<!--不匹配-->
<div attr="value"></div>
<!--不匹配-->
<div attr="val-ue"></div>
```

实践

```css
/* 指向PDF文件 */
[href$=".pdf"]{}

/* 下载zip压缩文件*/
[href$=".zip"]{}

/* 图片链接 */
[href$=".png"]{}
```

#### [attr*="val"]

匹配属性值中含有字符`val`的元素

```htm
<!--匹配-->
<div attr="val"></div>
<!--匹配-->
<div attr="text val"></div>
<!--匹配-->
<div attr="value"></div>
<!--匹配-->
<div attr="val-ue"></div>
```

### CSS属性选择器搜索过滤技术

```js
var eleStyle=document.createElement('style');
document.head.appendChild(eleStyle);
//文本框输入
input.addEventListener("input",function(){
    var value=this.value.trim();
    eleStyle.innerHTML=value?'[data-search]:not([data-search*="'+ value '"]){display:none;}':''
})
```

### 忽略大小写

使用i或者I作为运算符值忽略大小写

[attr~="val" i]{}

## 用户行为伪类

## URL定位伪类

### :link

用来匹配页面上`href`链接没有访问过的`<a>`元素

