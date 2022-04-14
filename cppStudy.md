# C++ 学习笔记

## Expressions(C++ Primer Chap. 4)

- low-level `const` (常量指针) / top-level `const` (指针常量)

- 显式转换  `static_cast / const_cast / reinterpret_cast / dynamic_cast `

  - 使用方法 `cast-name<type>(expression);`

  - `static_cast` 普适，除了常量指针

    - 可以用于还原 `void*` 中的内容

  - `const_cast` **只适用于**常量指针， **只有它**才能改变`const`

    - 转换的不是常量本身，如果常量指针本身指向的是常量，那么转换后的指针可以修改内容

      ```c++
      const char *pc;
      char  *p  =  const_cast<char*>(pc);  //  ok:but writing through p is undefined
      ```

  - `reinterpret_cast` 

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
  - 因为引用或指针可以指向不同的派生类，所以类型可能会发生改变

- 动态绑定依赖于 `virtual` 虚函数
  - 任何 非 (`static`成员函数以及构造函数) 都可以是虚函数

- 派生类可以**隐式转换**为基类，反之则不行

  - 因为派生类中包含了基类的内容

  - 基类指针指向的派生类的对象也不能重新指向派生类

    ```c++
    Bulk_quote bulk;
    Quote *itemP = &bulk;        // ok: dynamic type is Bulk_quote
    Bulk_quote *bulkP = itemP;   // error: can't convert base to derived
    ```

- 

