
#Item 32- Use init captures to move objects into closeures

If you have a move-only object(i.e. `std::unique_ptr`) which needs to go into a closure, C++11 offers no way to accomplish this. C++14
offers mechanisms to move objects into closures. In order to overcome this limitation in C++11, the standardization commitee provided a new
capture mechanism known as init capture. Advantages of using init capture:

* The name of the data member generated in the closure class
* An expression for initializing the data member

Take for example, the following snippet of code:

```c++
// This file is a "Hello, world!" in C++ language by GCC for wandbox.
#include <iostream>
#include <cstdlib>
#include <utility>
#include <memory>
class Person
{
    public:
      Person(std::string name):_name(name)
      { };
      void printName()
      {
         std::cout<<_name<<std::endl;
      }
      void setName(std::string name)
      {
          _name = name;
      }
    private:
        std::string _name;
        int _age;
        
};

int main()
{
    auto pw = std::make_unique<Person>("Name1");
    pw->printName();
    auto func = [pw = std::move(pw)]
    {
        std::cout<<"Entering Lambda"<<std::endl;
        pw->setName("Name2");
        pw->printName();
    };
    func();
    pw->printName(); //Seg Fault as object is already moved
}

```

In the above case, the expression `pw = std::move(pw)` refers to init capture. The left side of the expression indicates the name of
data member in the closure and the right side indicates the object which is in the scope of the lambda. In C++11, you can either write your 
own class or you can accomplish this by the use of `std::bind`.

```c++
std::vector<double> data; // object to be moved
// into closure
… // populate data
auto func = [data = std::move(data)] {}//C++14
```

The first argument to `std::bind` is a callable object. The subsequent arguments are the values that will be passed to the object.
```c++
auto func =
std::bind( // C++11 emulation
[](const std::vector<double>& data) // of init capture
{ /* uses of data */ },
std::move(data)
);
```

Things to remember from this item:
* In C++14, use init captures to move objects in closure.
* In C++11, use `std::bind` to emulate init capture or write your own class which emulates the behavior of the closure class.
