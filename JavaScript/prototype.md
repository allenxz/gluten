## 1. 理解原型的几个要点

1、所有的引用类型（数组、函数、对象）可以自由扩展属性（除null以外）。

2、所有的引用类型都有一个`__proto__`属性(也叫隐式原型，它是一个普通的对象)。

3、所有的函数都有一个`prototype`性(这也叫显式原型，它也是一个普通的对象)。

4、所有引用类型，它的`__proto__`属性指向它的构造函数的`prototype`属性。

5、当试图得到一个对象的属性时，如果这个对象本身不存在这个属性，那么就会去它的`__proto__`属性(也就是它的构造函数的`prototype`属性)中去寻找。



























> 参考：
>
> * https://blog.csdn.net/qq_36996271/article/details/82527256