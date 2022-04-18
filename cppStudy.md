# C++ 学习笔记

## Variables and Basic Types

- 类型别名
  - `typedef TYPE ALIAS;`
  - `using ALIAS = TYPE;`

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

- 无参默认构造函数 或 析构函数 可以写为 `~/someclass() = default;`
  
  - `=default` 如果在类内 那么则默认是 `inline`
  
- 类内 定义 的函数默认是`inline`，类内 声明 的函数可以使用`inline`， **类内没有声明`inline`的可以在类外定义时增加`inline`（推荐）**

- 初始化列表的使用

  - 在初始化 `const`，引用或者没有**默认无参构造函数**的`class`时务必要使用初始化列表

    ```c++
    // class1没有默认无参构造函数
    class class1
    {
        class1(int i);        
    };
    
    class class2
    {
    public:
        class2(int i);
    private:
        class1 c1; // 无法初始化
    };
    ```

    *因为没有无参默认构造函数，c1无法初始化*

## Dynamic Memory (C++ Primer Chap. 12)

- 智能指针`shared_ptr / unique_ptr`，用于内存泄露问题

  - 用法 添加头文件`<memory>` `smart_ptr<type> p;`

  - `shared_ptr` 允许多个指针指向同一个对象，内部存在一个计数器，当计数器为0（当最后一个引用该对象的指针销毁）后销毁对象

    - 一般用法

      ```c++
      shared_ptr<string> p1;    // shared_ptr that can point at a string
      shared_ptr<list<int>> p2; // shared_ptr that can point at a list of ints
      ```

    - 创立对象时就采用，采用`make_shared`方法，可以采用`auto`接收

      ```c++
      // shared_ptr that points to an int with value 42
      shared_ptr<int> p3 = make_shared<int>(42);
      // p4 points to a string with value 9999999999
      shared_ptr<string> p4 = make_shared<string>(10, '9');
      // p5 points to an int that is value initialized (§ 3.3.1 (p. 98)) to 0
      shared_ptr<int> p5 = make_shared<int>();
      ```

    - 拷贝/赋值（右值）/传参/返回 时计数器会++，赋值（左值）时计数器会--直至0，之后销毁对象

      ```c++
      auto q = make_shared<int> ();
      auto r = make_shared<int>(42); // int to which r points has one user
      r = q;  // assign to r, making it point to a different address
              // increase the use count for the object to which q points
              // reduce the use count of the object to which r had pointed
              // the object  r  had  pointed  to  has  no  users;  that  object is automatical freed
      ```

      

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

- 派生类的 作用域 嵌套在 基类的作用域中，因此派生类的成员变量会覆盖基类的成员变量

  - 派生类有与基类同名的函数会自动覆盖，要想使用基类的同名函数需要加基类名称的作用域
    - 覆盖后 子类 不能直接通过 `./->` 运算符访问基类对象的函数

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

- 派生类没有继承基类的私有成员变量，但是可以通过接口来访问或设置

- `friend` 可以使得类外的函数访问类的私有/保护成员

  - 派生类的友元只属于派生类，不能访问基类对象
  - 友元是单向的

- 对于私有继承，使用 `using`可以改变访问属性

  ```c++
  class Base {
  public:
      std::size_t size() const { return n; }
  protected:
      std::size_t n;
  };
  class Derived : private Base {    //  note: private inheritance
  public:
      // maintain access levels for members related to the size of the object
      using Base::size;
  protected:
      using Base::n;
  };
  ```

  

### 动态绑定

> The key idea behind OOP is **polymorphism**. Polymorphism is derived from a Greek
> word meaning “many forms.” We speak of types related by inheritance as
> polymorphic types, because we can use the “many forms” of these types while
> ignoring the differences among them. The fact that the *static and dynamic types*
> *of references and pointers can differ* is the **cornerstone** of how C++ supports
> polymorphism.

- 又名 运行时绑定(run-time binding)

- **只有当使用 基类的引用或者指针  调用虚函数时 才会发生动态绑定**
  
- 因为引用或指针可以指向不同的派生类，所以类型可能会发生改变
  - 声明的类型是静态类型，而动态绑定后的为动态类型
  - 如果直接将一个 派生类对象 赋值给 基类对象，那么 基类对象 会调用 基类虚函数，没有发生动态绑定，因为基类对象在创建之初就已经确定了
  
- 动态绑定依赖于 `virtual` **虚函数**

  - 任何 非 (`static`成员函数以及构造函数) 都可以是虚函数

  - 普通函数可以只有声明，而虚函数必须要定义，因为编译器不知道函数是否被调用

  - **派生类虚函数的参数 必须与 对应基类虚函数的参数 一致**（否则派生类方法会被覆盖），但是返回值可以是对方（如果转换允许）

    - 派生类中允许存在和基类虚函数 名字相同参数不一致的函数，在编程过程中如果想要重载基类虚函数，由于参数不一致重载不会发生，可能造成后续错误，因此在派生类中如果想要重载可以在函数体开始前添加`override`关键字以便检测此错误

      ```c++
      struct B {
          virtual void f1(int) const;
          virtual void f2();
          void f3();
      };
      struct D1 : B {
          void f1(int) const override; // ok: f1 matches f1 in the base
          void f2(int) override; // error: B has no f2(int) function
          void f3() override;    // error: f3 not virtual
          void f4() override;    // error: B doesn't have a function named f4
      };
      ```

    - 如果不希望函数被重载，可以在后面添加`final`关键字

      ```c++
      struct D2 : B {
         // inherits f2() and f3() from B and overrides f1(int)
         void  f1(int)  const  final; // subsequent  classes  can't  override  f1 (int)
      };
      struct D3 : D2 {
          void f2();          // ok: overrides f2 inherited from the indirect base,
      B
          void f1(int) const; // error: D2 declared f2 as final
      };
      ```

  - 虚函数如果有默认参数那么在调用时会使用基类默认参数

    ```c++
    class A {
    public:
    	virtual void foo (int i = 1) {
    		cout << "foo in class A\n";
    		cout << "i = " << i << endl << endl;
    	}
    	virtual ~A() = default;	// 如果不显示声明，会提示A有一个非虚的默认析构函数
    };
    
    class B : public A {
    public:
    	void foo(int i = 2) override {
    		cout << "foo in class B\n";
    		cout << "i = " << i << endl << endl;
    	}
    };
    
    A* p1 = new B;	p1->foo(); 
    // 输出结果
    // foo in class B
    // i = 1
    ```

- 包含纯虚函数的类（包括继承而来）称之为 纯虚抽象类， 不能实例化

- 派生类可以**隐式转换**为基类，反之则不行

  - 因为派生类中包含了基类的内容

  - 基类指针指向的派生类的对象也不能重新指向派生类

    ```c++
    Bulk_quote bulk;
    Quote *itemP = &bulk;        // ok: dynamic type is Bulk_quote
    Bulk_quote *bulkP = itemP;   // error: can't convert base to derived
    ```

- 虚函数表属于类，在编译期间就创建了

  - 虚函数指针属于对象，在生成对象时建立，在运行时动态绑定

## 关键字

### `static`

`static` 将变量分配到全局区（静态区），程序结束后才释放

- C 中的 `static`
  1. 静态局部变量，跳出局部作用域后也不会被销毁，但只能在作用域内访问
  2. 静态全局变量，对外部文件可以隐藏当前变量，可以在不同文件中重复定义
  3. 静态函数，与静态全局变量类似

- C++ 中的 `static` 还有

   1. 静态成员变量

      - 类变量，可以通过类名访问，所有实例共享一份数据
      - 类内声明，类外实现

   2. 静态成员函数

      - 可以访问静态成员和静态成员函数
      - 不可以访问非静态成员变量和非静态成员函数

      *非静态成员函数可以访问静态成员函数/变量*

- `class` 里的静态变量无法使用 `this`访问，因此静态成员函数后面不能加`const`

### `const`

const 对象的核心是  所有操作都不能改变对象本身，其余和普通对象没有区别

- `const` *默认* 和 `static` 一样可以在多个文件中重复定义
  - 如果想在其他文件中使用该变量，需要使用`extern`

- 在编译过程中，编译器会将使用到const对象的名字的地方替换成 对象的内容， 所以**一定要初始化**

  ```c++
  const int bufSize = 512; 
  ```

  *编译器会将代码中 `byfSize` 替换为 `512`*

- 非`const`的引用 **不能**绑定到  `const`对象

  ```c++
  const int ci = 1024;
  const int &r1 = ci;   // ok: both reference and underlying object are const
  r1 = 42;              // error: r1 is a reference to const
  int &r2 = ci;         // error: non const reference to a const object
  ```

  *non const $\rightarrow$ const，反之则不行*

- 每一个对象内部会有一个`this`指针指向该对象，不能改变其指向，`this` $\leftrightarrow$ `ClassName* const this`

  ```c++
  ClassName::func (ClassName* const this) 
  {
      ...
  }
  ```

  成员函数后面加`const`后 改成员函数不能修改对象的内容

  ```c++
  ClassName::func(const ClassName* const this) const 
  {
      ...
  }
  ```

  此时的`this` $\leftrightarrow $ `const ClassName* const obj`

  - `const`对象只能调 `const`成员函数，非`const`对象均可

- 关于`const`的重载

  - 只有当传递到参数是 引用或者指针 这些可能改变参数本身的类型时，才有可能发生`const`重载
    - 因为 值传递 相当于创建了一个新的变量，对其的修改和传入变量无关

      ```c++
      void ClassName::func (int num);
      void ClassName::func (const int num);
      ```

      *二者本质上是一致的，所以编译报错*

  - 调用函数时会优先调用和参数匹配的函数

    - 有`const`的会调用有`const`的，没有会调用没有的

      ```c++
      // [1] 
      void ClassName::func (int& num);
      // [2] 
      void ClassName::func (const int& num);
      --------------------------------------
      ClassName obj;
      int nonconstNum = 0;
      const int constNum = 0;
      obj.func(nonconstNum);		// [1]
      obj.func(constNum);			// [2]
      ```

    - 如果没有定义 非`const` 的函数，那么外部调用时会出错

      ```c++
      // [1] 
      // void ClassName::func (int& num);
      // [2] 
      void ClassName::func (const int& num);
      --------------------------------------
      ClassName obj;
      int nonconstNum = 0;
      const int constNum = 0;
      obj.func(nonconstNum);		// [1]
      obj.func(constNum);			// error
      ```

      *因为 `const` 不能向 非`const` 转换*

    

    





