# C++ 学习笔记

## Class (C++ Primer Chap. 7)

无参默认构造函数 或 析构函数 可以写为 `~/someclass() = default;`

## OOP (C++ Primer Chap. 15)

> The key ideas in object-oriented programming are **data abstraction**, **inheritance**, and **dynamic binding**.
>
> Using **data abstraction**, we can define classes that separate interface from implementation. 
>
> Through **inheritance**, we can define classes that model the relationships among similar types.
>
> Through **dynamic binding**, we can use objects of these types while ignoring the details of how they differ.

### 继承

- `struct`默认 `public` 继承， `class`默认`private`继承

- `static`静态成员 只有一份 基类和派生类的对象或者类本身都可以访问

- 派生类的 基类部分 和 派生类新定义部分 物理上未必连续

- 派生类的声明 **不能** 带上继承的类，因为声明只需要说明对象的类型

  ```c++
  class  Bulk_quote  :  public  Quote;  //  error:  derivation  list  can't appear here
  class Bulk_quote;                // ok:  right  way  to  declare  a derived class
  ```

- 基类必须**定义**后才能被继承，因为派生类必须知道继承了什么东西

- 不允许继承 需要在类名 后面增加 `final` 关键字

  ```c++
  class NoDerived final { /*  */ }; // NoDerived can't be a base class
  class Base { /*  */ }; 
  // Last is final; we cannot inherit from Last
  class Last final : Base { /*  */ }; // Last can't be a base class
  class Bad : NoDerived { /*  */ };   // error: NoDerived is final
  class Bad2 : Last { /*  */ };       // error: Last is final
  ```

  

### 动态绑定

- 又名 运行时绑定(run-time binding)

- **只有当使用 基类的引用或者指针  调用虚函数时 才会发生动态绑定**

- 动态绑定依赖于 `virtual` 虚函数
  - 任何 非 (`static`成员函数以及构造函数) 都可以是虚函数

