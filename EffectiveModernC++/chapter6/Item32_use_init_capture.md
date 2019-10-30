
# Item 32- Use init captures to move objects into closeures

If you have a move-only object(i.e. `std::unique_ptr`) which needs to go into a closure, C++11 offers no way to accomplish this. C++14
offers mechanisms to move objects into closures. In order to overcome this limitation in C++11, the standardization commitee provided a new
capture mechanism known as init capture. Advantages of using init capture:

* The name of the data member generated in the closure class
* An expression for initializing the data member

Take for example, the following snippet of code:

```c++
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
data member in the closure that is generated and the right side indicates the object which is in the scope of the lambda. Output from cppinsight:
```c++
#include <iostream>
#include <cstdlib>
#include <utility>
#include <memory>
class Person
{
  
  public: 
  inline Person(std::basic_string<char> name)
  : _name{std::basic_string<char>(name)}
  {
  }
  
  inline void printName()
  {
    std::operator<<(std::cout, this->_name).operator<<(std::endl);
  }
  
  inline void setName(std::basic_string<char> name)
  {
    this->_name.operator=(name);
  }
  
  
  private: 
  std::basic_string<char> _name;
  int _age;
  public: 
  // inline ~Person() = default;
};
int main()
{
  std::unique_ptr<Person, std::default_delete<Person> > pw = std::unique_ptr<Person, std::default_delete<Person> >(std::make_unique<Person>("Name1"));
  pw.operator->()->printName();
    
  class __lambda_28_17
  {
    public: 
    inline void operator()() const
    {
      std::operator<<(std::cout, "Entering Lambda").operator<<(std::endl);
      pw.operator->()->setName(std::basic_string<char>(std::basic_string<char>("Name2", std::allocator<char>())));
      pw.operator->()->printName();
    }
    
    private: 
    std::unique_ptr<Person, std::default_delete<Person> > pw;
    public: 
    // inline __lambda_28_17(const __lambda_28_17 &) = delete;
    // inline __lambda_28_17(__lambda_28_17 &&) noexcept = default;
    __lambda_28_17(std::unique_ptr<Person, std::default_delete<Person> > && _pw)
    : pw{std::move(_pw)}
    {}
    
  };
  
  __lambda_28_17 func = __lambda_28_17(__lambda_28_17{std::unique_ptr<Person, std::default_delete<Person> >(std::move(pw))});
  func.operator()();
  pw.operator->()->printName();
}
```

In C++11, you can either write your own class or you can accomplish this by the use of `std::bind`:
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
