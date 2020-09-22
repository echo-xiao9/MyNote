# lab1

[TOC]



## 错误修复

- 39行const 只能初始化，不能赋值，因此去掉const

<img src="/Users/kangyixiao/Desktop/截屏2020-09-18 下午2.05.21.png" alt="截屏2020-09-18 下午2.05.21" style="zoom:50%;" width=900 />

​	解决方案：使用初始化列表

<img src="/Users/kangyixiao/Library/Application Support/typora-user-images/截屏2020-09-18 下午2.02.57.png" alt="截屏2020-09-18 下午2.02.57" style="zoom:50%;" width=900 />

​	构建成功

<img src="/Users/kangyixiao/Library/Application Support/typora-user-images/截屏2020-09-18 下午2.07.52.png" alt="截屏2020-09-18 下午2.07.52" style="zoom:50%;"  width=900 />

## 知识摘记

### 符号表示

顶部的隔间包含类的名称。它以粗体和居中的形式打印，第一个字母大写。
中间的部分包含类的属性。它们是左对齐的，第一个字母是小写。
底部的隔间包含类可以执行的操作。它们也是左对齐的，第一个字母是小写的。

| `+`  | Public    |
| ---- | --------- |
| `-`  | Private   |
| `#`  | Protected |
| `~`  | Package   |

<img src="/Users/kangyixiao/Downloads/2560px-Uml_classes_en.svg.png" alt="2560px-Uml_classes_en.svg" style="zoom:50%;" width=900 />



**association**

In generic terms, the [causation](https://en.wikipedia.org/wiki/Causality) is usually called "sending a message", "invoking a [method](https://en.wikipedia.org/wiki/Method_(computer_science))" or "calling a [member function](https://en.wikipedia.org/wiki/Member_function)" to the controlled object. 

**Aggregation**

When the container is destroyed, the contents are usually not destroyed

### substr(a,b)

string sub2 = s.substr(5, 3); //从下标为5开始截取长度为3位：sub2 = "567"



### 继承构造函数

> 参考：https://blog.csdn.net/zhaolianyun/article/details/45271727?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param

### 引用、指针传递

引用即声明的时候是引用，指针即声明的时候是指针。

引用传参传本身，因为需要一个别名，指针传参传的是地址。

```C++
#include<iostream>
using namespace std;
/*
	author: James-J
	time: 2019/02/28
*/
// 传值 
void fun_1(int num){
	num = 100;
	cout<<"In function 1 num = "<<num<<endl;
} 
// 传指针 
void fun_2(int *num){
	*num = 200;  // *num就是根据指针num找原来位置的变量
	cout<<"In function 2 num = "<<*num<<endl;
}
// 传引用 
void fun_3(int &num){
	num = 300;  // 引用的操作看起来就像是直接赋值一样
	cout<<"In function 3 num = "<<num<<endl;
}
 
int main(){
	int num = 0;
	cout<<"num = "<<num<<endl<<endl;
	// 传值 
	cout<<"Before function 1 num = "<<num<<endl;
	fun_1(num);
	cout<<"After function 1 num = "<<num<<endl<<endl;
	// 传指针
	cout<<"Before function 2 num = "<<num<<endl;
	fun_2(&num);  // 地址传过去
	cout<<"After function 2 num = "<<num<<endl<<endl;
	// 传引用
	cout<<"Before function 3 num = "<<num<<endl;
	fun_3(num);
	cout<<"After function 3 num = "<<num<<endl<<endl;
	return 0;
} 
```



## question



