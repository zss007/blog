---
title: css 之 Sass 入门二
categories:
- css
---
继续研究 sass 的运算、控制命令、函数、@ 规范等。
<!--more-->
### 一、运算
#### 1、加法、减法
```
.box {
  width: 20px + 8px;
}
```
携带不同类型的单位时，在 Sass 中计算会报错
#### 2、乘法
```
.box {
  width: 10px * 2;
}
```
当一个单位同时声明两个值时会有问题；携带不同类型的单位时，在 Sass 中计算会报错
#### 3、除法
```
.box {
  width: (100px / 2);  
}
```
如果数值或它的任意部分是存储在一个变量中或是函数的返回值、数值被圆括号包围、数值是另一个数学表达式的一部分才会生效。
#### 4、颜色运算
所有算数运算都支持颜色值，并且是分段运算的。也就是说，红、绿和蓝各颜色分段单独进行运算。
如：`color: #010203 + #040506;` ==> `color: #050709;`
#### 5、字符运算
可通过加法符号 “+” 来对字符串进行连接。
如：`p:before {content: "Foo " + Bar;}` ==> `p:before {content: "Foo Bar";}`
### 二、控制命令
#### 1、@if
@if 可一个条件单独使用，也可以和 @else 结合多条件使用
#### 2、@for
@for `$i` from <start> through <end> (包括 end)
@for `$i` from <start> to <end> (不包括 end)
其中 $i 表示变量，start 表示起始值，end 表示结束值
#### 3、@while
和 @for 指令很相似，只要 @while 后面的条件为 true 就会执行
#### 4、@each
@each 循环就是去遍历一个列表，然后从列表中取出对应的值 (@each `$var` in <list>)
#### 5、三目判断
if(`$condition`, `$if_true`, `$if_false`)。三个参数分别表示：条件，条件为真的值，条件为假的值。
### 三、字符串函数
#### 1、unquote、quote
unquote：删除一个字符串中的引号；quote：给字符串添加引号
#### 2、to-upper-case、to-lower-case
to-upper-case：将字符串小写字母转换成大写字母；to-lower-case：将字符串转换成小写字母
### 四、数字函数
1、percentage(`$value`)：将一个不带单位的数转换成百分比值；
2、round(`$value`)：将数值四舍五入，转换成一个最接近的整数；
3、ceil(`$value`)：将大于自己的小数转换成下一位整数；
4、floor(`$value`)：将一个数去除他的小数部分；
5、abs(`$value`)：返回一个数的绝对值；
6、min(`$numbers`…)：找出几个数值之间的最小值；
7、max(`$numbers`…)：找出几个数值之间的最大值；
8、random(): 获取随机数
### 五、列表函数
1、length(`$list`)：返回一个列表的长度值；
2、nth(`$list`, `$n`)：返回一个列表中指定的某个标签值
3、join(`$list1`, `$list2`, [$separator])：将两个列给连接在一起，变成一个列表；
4、append(`$list1`, `$val`, [$separator])：将某个值放在列表的最后；
5、zip(`$lists`…)：将几个列表结合成一个多维的列表；
6、index(`$list`, `$value`)：返回一个值在列表中的位置值。
### 六、Introspection 函数
1、type-of(`$value`)：返回一个值的类型
2、unit(`$number`)：返回一个值的单位
3、unitless(`$number`)：判断一个值是否带有单位
4、comparable(`$number`-1, `$number`-2)：判断两个值是否可以做加、减和合并
### 七、Sass Maps 函数
1、map-get(`$map`,`$key`)：根据给定的 key 值，返回 map 中相关的值。
2、map-merge(`$map1`,`$map2`)：将两个 map 合并成一个新的 map。
3、map-remove(`$map`,`$key`)：从 map 中删除一个 key，返回一个新 map。
4、map-keys(`$map`)：返回 map 中所有的 key。
5、map-values(`$map`)：返回 map 中所有的 value。
6、map-has-key(`$map`,`$key`)：根据给定的 key 值判断 map 是否有对应的 value 值，如果有返回 true，否则返回 false。
7、keywords(`$args`)：返回一个函数的参数，这个参数可以动态的设置 key 和 value。
### 八、RGB 颜色函数
1、rgb(`$red`,`$green`,`$blue`)：根据红、绿、蓝三个值创建一个颜色；
2、rgba(`$red`,`$green`,`$blue`,`$alpha`)：根据红、绿、蓝和透明度值创建一个颜色；
3、red(`$color`)：从一个颜色中获取其中红色值；
4、green(`$color`)：从一个颜色中获取其中绿色值；
5、blue(`$color`)：从一个颜色中获取其中蓝色值；
6、mix(`$color`-1,`$color`-2,[$weight])：把两种颜色混合在一起。
### 九、Opacity 函数
1、alpha(`$color`) /opacity(`$color`)：获取颜色透明度值；
2、rgba(`$color`, `$alpha`)：改变颜色的透明度值；
3、opacify(`$color`, `$amount`) / fade-in(`$color`, `$amount`)：使颜色更不透明；
4、transparentize(`$color`, `$amount`) / fade-out(`$color`, `$amount`)：使颜色更加透明。
### 十、@ 规则
#### 1、@import
sass中如导入其他sass文件，最后编译为一个css文件，优于纯 css 的 @import
#### 2、@extend
用来扩展选择器或占位符
#### 3、@at-root
@at-root 从字面上解释就是跳出根元素。当你选择器嵌套多层之后，想让某个选择器跳出，此时就可以使用 @at-root
#### 4、@debug
@debug 在 Sass 中是用来调试的，当你的在 Sass 的源码中使用了 @debug 指令之后，Sass 代码在编译出错时，在命令终端会输出你设置的提示 Bug
#### 5、@warn
@warn 和 @debug 功能类似，用来帮助我们更好的调试 Sass
#### 6、@error
@error 和 @warn、@debug 功能是如出一辙
