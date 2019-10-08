# std::shared_ptr usage

With `std::shared_ptr`, clients need not to concern with managing life-time of *pointed-to* objects (as garbage collector). Beside, the timing of the objects' destruction is deterministic.

* Using a shared pointer is like a normal raw pointer
* *Reference count* is how a shared pointer knows if it is the last one pointing to an object.

```c++
#include <memory>
#include <iostream>

int main()
{
    std::shared_ptr<int> sp1 = std::make_shared<int>(1);
    std::shared_ptr<int> sp2 = std::make_shared<int>(2);
    std::cout << "use_count = " << sp1.use_count() << std::endl;
    sp2 = sp1;
    
    //dereference a shared pointer like a raw pointer
    std::cout << "*sp1= " << *sp1 << ", *sp2= " << *sp2 << std::endl;
    *sp1 += 1;
    std::cout << "*sp1= " << *sp1 << ", *sp2= " << *sp2 << std::endl;   
    
    //check number of pointers point to the same object
    if (sp1.use_count() == sp2.use_count())
        std::cout << "use_count = " << sp2.use_count() << std::endl;
    else
        std::cout << "something wrong" << std::endl;  
    
    return 0;
}
Start

use_count = 1
*sp1= 1, *sp2= 1
*sp1= 2, *sp2= 2
use_count = 2

0

Finish
```
We rework on the above example to show how copy assignment `sp2 = sp1` makes `sp2` destroys its object and start pointing to same object with `sp1`. We also shorten the code with `auto`.

```c++
#include <iostream>
#include <memory>

struct Widget
{
    std::string name = "";
    Widget(std::string n) : name{n} {}
    
    ~Widget()
    {
        std::cout << "Destroy.." << name << std::endl;
    }
};

int main()
{
    auto sp1 = std::make_shared<Widget>("sp1");
    auto sp2 = std::make_shared<Widget>("sp2");
    
    std::cout << "use_count = " << sp1.use_count() << std::endl;
    sp2 = sp1; //copy assignment causes sp2 free its object and point to same object with sp1
    
    sp1->name = "common shared pointer";
    std::cout << "sp1->name " << sp1->name << ", sp2->name= " << sp2->name << std::endl;   
    
    //check number of pointers point to the same object
    if (sp1.use_count() == sp2.use_count())
        std::cout << "use_count = " << sp2.use_count() << std::endl;
    else
        std::cout << "something wrong" << std::endl;  
    
    return 0;
}//destroy only one time by sp1 because of sp1 and sp2 on stack

Start

use_count = 1
Destroy..sp2
sp1->name common shared pointer, sp2->name= common shared pointer
use_count = 2
Destroy..common shared pointer

0

Finish
```
**Reference count**
* Constructor increments *reference count*
* Destructor decrements *reference count*
* Copy assignment (as first example above) also decrements *reference count* to object `int` of value 2 and increments *reference count* of object `int` of value 1.
* When `shared_ptr` sees *reference count* is `0` after decrement, it destroys resource.
* Increment and decrement of a *reference count* must be atomic
* For `std::shared_ptr`: move constructors are faster than copy constructors; move assignments are faster than copy assignments. Because move operators seem not care about reference counts. So, "move when moveable, copy when otherwise".

# `shared_ptr` vs. `unique_ptr`
1. `shared_ptr`s are twice the size of a raw pointer because it stores
* a raw pointer to resource
* a raw pointer to reference count (but actually later we see it is a block of memory, called *control block*)

2. `shared_ptr`s are more flexible with custom deleter, compared to `unique_ptr`s.
This is because share pointers need defining custom deleter as input of constructor while unique pointers need to define custom deleter in template parameter.

```c++
#include <memory>
#include <vector>
#include <iostream>

struct Widget
{
    int a;
};

void makeLogEntry(Widget* pw)
{
    std::cout <<__PRETTY_FUNCTION__<< pw->a << std::endl;
}

void makeSpecialLogEntry(Widget *pw)
{
    std::cout << __PRETTY_FUNCTION__ << pw->a << std::endl;
}

auto customDeleter = [](Widget* pw)            //custom deleter writes log before destroying resource
                      {
                        makeLogEntry(pw);
                        delete pw;
                      };

auto customSpecialDeleter = [](Widget* pw)    //custom deleter with special logging
                      {
                        makeSpecialLogEntry(pw);
                        delete pw;
                      };
int main()
{
    std::unique_ptr<Widget, decltype(customDeleter)> 
                                  p(new Widget, customDeleter); // define type of deleter in template
    
    std::shared_ptr<Widget> sp1(new Widget, customDeleter);     // define type of custom deleter in ctor
    std::shared_ptr<Widget> sp2(new Widget, customSpecialDeleter);
    
    std::vector<std::shared_ptr<Widget>> vsp;  // vector of shared pointers not care about deleters
    vsp.push_back(sp1);                        // of each element       
    vsp.push_back(sp2);
    return 0;
}

Start

void makeSpecialLogEntry(Widget*)0
void makeLogEntry(Widget*)0
void makeLogEntry(Widget*)0

0

Finish
```
As a result, we can have 
* A vector of shared pointers with different deleters. 
* Shared pointers can be reassigned, reset its deleter. 
* Passing a share pointer to a function, with different deleters.

# Control Block
There is a control block for each object managed by `std::shared_ptr`s. The control block contains:
* a reference count
* weak count (?)
* Other data: custom deleter, custom allocator, etc.

todo: ASCII_ize control block image.

When a shared pointer is created to the *same* object, no control block is created. So, when a control block is created ?
* `std::make_shared` (item 21) always creates a control block
* `std::shared_ptr` is created from `std::unique_ptr` and `std::auto_ptr` (`auto_ptr` in legacy code)
* `std::shared_ptr` is created from raw pointer

It makes sense since all three items mean that it is the first time a shared pointer is created.

# Creating `std::shared_ptr` from raw pointer
## Using `new` and `ctor`
Thing goes wrong when multiple `std::shared_ptr`s is created from a raw pointer.

Assuming we have a raw pointer as below.
```c++
#include <iostream>
#include <memory>

struct Widget
{
    std::string name = "";
    Widget(std::string n) : name{n} {}
    
    ~Widget()
    {
        std::cout << "Destroy.." << name << std::endl;
    }
};

int main()
{
    auto pw = new Widget("raw ptr");
    delete pw;   
    
    return 0;
}
Start

Destroy..raw ptr

0

Finish
```
But when we introduce a `shared_ptr` from this raw pointer. We double deleting it.
```c++
#include <iostream>
#include <memory>

struct Widget
{
    std::string name = "";
    Widget(std::string n) : name{n} {}
    
    ~Widget()
    {
        std::cout << "Destroy.." << name << std::endl;
    }
};

int main()
{
    auto pw = new Widget("ptr");
    std::shared_ptr<Widget> spw1 (pw);
    delete pw;   
    
    return 0;
}
Start

Destroy..ptr

Segmentation fault

Finish
```
Object `Widget` is deleted twice, thus causes undefined behavior. 

When we comment out `delete pw`, it is OK.
```c++
int main()
{
    auto pw = new Widget("ptr");
    std::shared_ptr<Widget> spw1 (pw);
    //delete pw;   
    
    return 0;
} 
Start

Destroy..ptr

0

Finish
```
But when we have two `shared_ptr`s created from same raw pointer, thing does go wrong. There are two *reference count* to the same object pointed by both `spw1` and `spw2`. Thus, causes undefined behavior.
```c++
#include <iostream>
#include <memory>

struct Widget
{
    std::string name = "";
    Widget(std::string n) : name{n} {}
    
    ~Widget()
    {
        std::cout << "Destroy.." << name << std::endl;
    }
};

int main()
{
    auto pw = new Widget("ptr");
    std::shared_ptr<Widget> spw1 (pw);
    std::shared_ptr<Widget> spw2 (pw);
    std::cout << "sp1's name: " << spw1->name << ". RC= " << spw1.use_count() << std::endl;
    std::cout << "sp2's name: " << spw2->name << ". RC= " << spw2.use_count() << std::endl;
    
    return 0;
}
Start

sp1's name: ptr. RC= 1
sp2's name: ptr. RC= 1
Destroy..ptr

Segmentation fault

Finish
```
There are two advises:
* first, don't use raw pointer at the first place. Use smart pointer
* second, don't make `shared_ptr` from a raw pointer. Instead, use `std::make_shared` (item 21) or use `new` and ctor to make second `shared_ptr`. Don't pass variable `pw` as above.
```c++
int main()
{
    std::shared_ptr<Widget> spw1 (new Widget("ptr"));  //using new
    std::shared_ptr<Widget> spw2 (spw1);               //using copy ctor
    
    std::cout << "sp1's name: " << spw1->name << ". RC= " << spw1.use_count() << std::endl;
    std::cout << "sp2's name: " << spw2->name << ". RC= " << spw2.use_count() << std::endl;
}
Start

sp1's name: ptr. RC= 2
sp2's name: ptr. RC= 2
Destroy..ptr

0

Finish
```
## Using `std::enable_shared_from_this` 
```c++
#include <iostream>
#include <memory>

struct Widget : public std::enable_shared_from_this<Widget> { // CRTP
{
    std::vector<std::shared_ptr<Widget>> prossessedWidgets;
    
    void process()
    {
        prossesedWidgets.emplace_back(shared_from_this());  //converter from this ptr to shared_ptr
            
    }
    
    ~Widget()
    {
        std::cout << "Destroy.." ;
    } 
    
};

int main()
{  
   Widget w;

   return 0;
}
```

One more example where you can create a shared pointer from a raw pointer, and free resource when applicable.
```c++
#include <iostream>
#include <memory>
#include <string>

struct Foo : public std::enable_shared_from_this<Foo> { //CRTP
    std::string name= "";
    Foo() { std::cout << "Foo::Foo\n"; }
    ~Foo() { std::cout << "Foo::~Foo\n"; }
    std::shared_ptr<Foo> getFoo() { return shared_from_this(); }
};
 
int main() {
    Foo *f = new Foo;             //raw pointer
    std::shared_ptr<Foo> pf1;     //pf1 not pointing to any obj
    std::cout << "RC = "<< pf1.use_count() << std::endl;
    {//local scope for pf2
        std::shared_ptr<Foo> pf2(f);  //pf2 pointing to same obj by raw pointer
        pf1 = pf2->getFoo();  // shares ownership of object with pf2
        std::cout << "RC = "<< pf1.use_count() << std::endl;
    }
    std::cout << "RC = "<< pf1.use_count() << std::endl;
    std::cout << "pf2 is gone\n";   //pf2 is gone because out of scope
}//pf1 is gone, thus free obj
Start

Foo::Foo
RC = 0
RC = 2
RC = 1
pf2 is gone
Foo::~Foo

0

Finish
```

# Other notes
- `std::unique_ptr` can be converted to `shared_ptr` but not the other way around. This means, *control block* is created as mentioned in rules of control block
- no `std::shared_ptr` of an array `std::shared_ptr<T[]>.` Dont try to build it.

