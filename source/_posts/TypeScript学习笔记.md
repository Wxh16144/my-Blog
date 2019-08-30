---
title: TypeScript学习笔记
date: 2018-09-27 16:15:36
tags: ['typescript']
---

转发自：[Typescript 学习笔记](https://blog.52chinaweb.com/blog/tsnote.html) 

<!--more-->

TypeScript 是由微软开发的一款开源的编程语言。TypeScript 是Javascript 的超集，遵循最新的ES6、Es5 规范。TypeScript 扩展了JavaScript的语法。TypeScript 更像后端java、C#这样的面向对象语言可以让js 开发大型企业项目。谷歌也在大力支持Typescript 的推广，谷歌的angular2.x+就是基于Typescript 语法。最新的Vue 、React 也可以集成TypeScript。

## 安装

---

```javascript
# 1. 全局安装typescript 
npm install -g typescript

# 2. 查看版本
tsc -v
```

> 本篇无特别说明，都是在ts文件中书写代码

## 数据类型

---

typescript中为了使编写的代码更规范，更有利于维护，增加了类型校验,写ts代码必须指定类型

>  <span data-type="color" style="color:#676f7d">  布尔类型（boolean）</span>
> <span data-type="color" style="color:#676f7d">   数字类型（number）</span>
> <span data-type="color" style="color:#676f7d">   字符串类型(string)</span>
> <span data-type="color" style="color:#676f7d">   数组类型（array）</span>
> <span data-type="color" style="color:#676f7d">   元组类型（tuple）</span>
> <span data-type="color" style="color:#676f7d">   枚举类型（enum）</span>
> <span data-type="color" style="color:#676f7d">   任意类型（any）</span>
> <span data-type="color" style="color:#676f7d">   null 和 undefined</span>
> <span data-type="color" style="color:#676f7d">   void类型</span>
> <span data-type="color" style="color:#676f7d">   never类型</span>

### 布尔类型
```typescript
let isshow:boolean = true
```
### 数字类型
```typescript
let num:number=123;
```
### 字符串类型
```typescript
let title:string = 'Hello Chang Jun'
```
### 数组类型
```typescript
# 定义数组中所有的元素都必须是number类型
let list:number[]=[1,2,3] 
# 等同于上面
let arr:Array<number>=[1,2,3];
```
### 元组类型(数组的一种)
```typescript
# 数组合并了相同类型的对象，而元组（Tuple）合并了不同类型的对象。 
let arr:[number,string]=[123,'this is ts'];
```
### 枚举
<span data-type="color" style="color:rgb(51, 51, 51)"><span data-type="background" style="background-color:rgb(255, 255, 255)">枚举（Enum）类型用于取值被限定在一定范围内的场景，比如一周只能有七天，颜色限定为红绿蓝等。</span></span>
```typescript
enum Color {blue,red,orange}
let c:Color = Color.red
# 如果标识符没有赋值，它的值就是下标
console.log(c)  // 1

enum Err {'undefined'=-1,'null'=-2,'success'=1};
var e:Err=Err.success;
console.log(e); //1
```
### 任意类型（any）
```typescript
let num:any=123;
num='str';
num=true;
console.log(num) //true
```
### null 和 undefined
```typescript
let num:number;
console.log(num)  //输出：undefined   ts报错

let num:undefined
console.log(num)  //输出：undefined   ts正确

let num:number | undefined;
num=123;
console.log(num);   // 123

let num:number | undefined;
console.log(num);   // undefined

# 一个元素可能是 number类型 可能是null 可能是undefined
let num:number | null | undefined;
num=1234;
console.log(num) //1234
```
### void类型
typescript中的void表示没有任何类型，一般用于定义方法的时候方法没有返回值。
```typescript
# 方法没有返回任何类型
function run():void{
    console.log('run')
}
run();

# 方法返回string类型
function getTitle():string{
    return '1234'
}
getTitle();
```
## never类型
是其他类型 （包括 null 和 undefined）的子类型，代表从不会出现的值。这意味着声明never的变量只能被never类型所赋值。
```typescript
let a:never;
a=(()=>{
  throw new Error('错误')
})()
```

## typeScript中的函数

---

> 1. <span data-type="color" style="color:#676f7d">函数的定义</span>
> 2. <span data-type="color" style="color:#676f7d">可选参数</span>
> 3. <span data-type="color" style="color:#676f7d">默认参数</span>
> 4. <span data-type="color" style="color:#676f7d">剩余参数</span>
> 5. <span data-type="color" style="color:#676f7d">函数重载</span>
> 6. <span data-type="color" style="color:#676f7d">箭头函数</span>

### 函数的定义
```typescript
function getTitle():string{
  return '123';
}
function getInfo(name:string,age:number):string{
  return `${name} --- ${age}`;
}
```
### 可选参数
```typescript
# 可选参数必须配置到参数的最后面
function getInfo(name:string,age?:number):string{
  if(age){
     return `${name} --- ${age}`;
  }else{
     return `${name} ---年龄保密`;
  }
}
```
### 默认参数
```typescript
function getInfo(name:string,age:number=20):string{
  if(age){
    return `${name} --- ${age}`;
  }else{
    return `${name} ---年龄保密`;
  }
}
```
### 剩余参数
```typescript
# 三点运算符 接受新参传过来的值
function sum(...result:number[]):number{          
   var sum=0;
   for(var i=0;i<result.length;i++){
      sum+=result[i];  
   }
   return sum;
}
sum(1,2,3,4,5,6);
```
### 函数重载
```typescript
function getInfo(name:string):string;
function getInfo(name:string,age:number):string;
function getInfo(name:any,age?:any):any{
  if(age){
    return '我叫：'+name+'我的年龄是'+age;
  }else{
    return '我叫：'+name;
  }
}
alert(getInfo('zhangsan'));  // 正确
alert(getInfo(123));  错误
alert(getInfo('zhangsan',20));// 正确
```

## typeScript中的类继承修饰符

---

> 1. ts中定义类
> 2. ts中实现继承 extends super
> 3. ts中类的修饰符
> 4. 静态属性和静态方法
> 5. 抽象类和多态

### ts中定义类
```typescript
class Person{
    name:string;
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
}
let p=new Person('ChangJun');
p.setName('ChangXiaoJun');
```
### ts中实现继承
```typescript
class Person{
    name:string;
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
}
class Web extends Person{
     constructor(name:string){
       super(name);
    }
}
let p=new Person('ChangJun');
alert(p.run());

# 如果子类方法和父类方法一致，那么执行子类方法
```
### ts中的修饰符(默认是共有 public)
> <span data-type="color" style="color:#676f7d">public :公有          在当前类里面、 子类  、类外面都可以访问</span>
> <span data-type="color" style="color:#676f7d">protected：保护类型    在当前类里面、子类里面可以访问 ，在类外部没法访问</span>
> <span data-type="color" style="color:#676f7d">private ：私有         在当前类里面可以访问，子类、类外部都没法访问</span>

```typescript
class Person{
    protected name:string;
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
}
let p=new Person('ChangJun');
alert(p.name); // 报错

class Person{
    private name:string;
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
}
let p=new Person('ChangJun');
alert(p.name); // 报错

```
### ts中的静态属性和静态方法
```typescript
class Person{
    name:string;
    static sex="男";
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
    static print(){
        print('Hello ChangJun')
    }
}

# 静态属性不需要new 即可访问
# 静态方法不需要new 即可访问
Person.sex
Person.print()
```
### ts中多态
多态：父类定义一个方法不去实现，让继承它的子类去实现，每一个子类有不同的表现 
```typescript
class Person{
    name:string;
    constructor(name:string){
       this.name=name;
    }
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
    getScore():string{
        console.log('分数是多少')
    }
}

class Per extends Person{
     constructor(name:string){
       super(name)
    }
    getScore():string{
        return   this.name+'分数'
    }
}
```
### <span data-type="color" style="color:#676f7d">抽象类</span>
typescript中的抽象类：它是提供其他类继承的基类，不能直接被实例化
用abstract关键字定义抽象类和抽象方法，抽象类中的抽象方法不包含具体实现并且必须在派生类中实现
abstract抽象方法只能放在抽象类里面
抽象类和抽象方法用来定义标准 
```typescript
abstract class Animal{
    
    public name:string;

    constructor(name:string){
        this.name=name;
    }

    abstract eat():any;  //抽象方法不包含具体实现并且必须在派生类中实现。    
    run(){
        console.log('其他方法可以不实现')
    }
}

class Dog extends Animal{

    //抽象类的子类必须实现抽象类里面的抽象方法
    constructor(name:any){
        super(name)
    }
    eat(){
        console.log(this.name+'吃肉')
    }
}

let h=new Dog('哈士奇');
h.eat();

```

## Typescript中的接口
> 1. <span data-type="color" style="color:#676f7d">属性类接口</span>
> 2. <span data-type="color" style="color:#676f7d">函数类型接口</span>
> 3. <span data-type="color" style="color:#676f7d">可索引接口</span>
> 4. <span data-type="color" style="color:#676f7d">类类型接口</span>
> 5. <span data-type="color" style="color:#676f7d">接口扩展</span>

接口的作用：在面向对象的编程中，接口是一种规范的定义，它定义了行为和动作的规范，在程序设计里面，接口起到一种限制和规范的作用。接口定义了某一批类所需要遵守的规范，接口不关心这些类的内部状态数据，也不关心这些类里方法的实现细节，它只规定这批类里必须提供某些方法，提供这些方法的类就可以满足实际需要。 typescrip中的接口类似于java，同时还增加了更灵活的接口类型，包括属性、函数、可索引和类等。

### 属性类接口
```typescript
// ts中自定义方法传入参数,对json进行约束
function printLabel(labelInfo:{label:string}):void {
    console.log('printLabel');
}

printLabel({label:'Chang Jun'}); // 正确


// 对批量方法传入参数进行约束
interface FullName{
    firstName:string;
    secondName:string;
}
# 传入的对象必须包含firstName secondName否则报错,参数顺序可不一样
function printName(name:FullName){
  // 必须传入对象  firstName  secondName
  console.log(name.firstName+'--'+name.secondName);
}

# 可选属性，表示传入对象可以没有secondName属性
interface FullName{
    firstName:string;
    secondName？:string;
}

```
### 函数类型接口
对方法传入的参数以及返回值进行约束
```typescript
// 加密的函数类型接口
interface encrypt{
    (key:string,value:string):string;
}
let md5:encrypt=function(key:string,value:string):string{
    //模拟操作
    return key+value;
}
console.log(md5('name','zhangsan'));
```

### 可索引接口
数组、对象的约束（不常用）
```typescript
# 对数组的约束
interface UserArr{
   [index:number]:string
}
let arr:UserArr=['aaa','bbb']; // 正确
let arr:UserArr=[123,'bbb']; // 错误

# 对对象的约束
interface UserObj{
   [index:string]:string
}
var arr:UserObj={name:'张三'};
```

### 类类型接口
```javascript
interface Animal{
    name:string;
    eat(str:string):void;
}

class Dog implements Animal{
   name:string;
   constructor(name:string){
      this.name=name;
   }
   eat(){
      console.log(this.name+'吃粮食')
   }
}

class Cat implements Animal{
    name:string;
    constructor(name:string){
        this.name=name;
    }
    eat(food:string){
        console.log(this.name+'吃'+food);
    }
}

let c=new Cat('猫');
c.eat('老鼠');
```

## typeScript中的泛型
软件工程中，我们不仅要创建一致的定义良好的API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。
在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。
通俗理解：泛型就是解决 类 接口 方法的复用性、以及对不特定数据类型的支持(类型校验)

### 泛型属性
```typescript
// 泛型:可以支持不特定的数据类型
// T表示泛型，具体什么类型是调用这个方法的时候决定的

function getData<T>(value:T):T{
    return value
}
getData<number>(123)
getDat<string>(123) // 错误
```

### 类的泛型
```typescript
# 最小堆算法
class MinClas<T>{
    public list:T[]=[];

    add(value:T):void{
        this.list.push(value);
    }

    min():T{        
        var minNum=this.list[0];
        for(var i=0;i<this.list.length;i++){
            if(minNum>this.list[i]){
                minNum=this.list[i];
            }
        }
        return minNum;
    }
}

var m1=new MinClas<number>();   /*实例化类 并且制定了类的T代表的类型是number*/
m1.add(11);
m1.add(3);
m1.add(2);
alert(m1.min())


var m2=new MinClas<string>();   /*实例化类 并且制定了类的T代表的类型是string*/

m2.add('c');
m2.add('a');
m2.add('v');
alert(m2.min())
```

### 泛型接口
```typescript
interface ConfigFn<T>{
    (value:T):T;
}

function getData<T>(value:T):T{
    return value;
}

let myGetData:ConfigFn<string>=getData;
myGetData('20'); // 正确
myGetData(20)  // 错误
```
