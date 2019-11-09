# Item 32- Use init captures to move objects into closures
## C++14 init capture
If you have a move-only objects (i.e. `std::unique_ptr`, `std::future`) which need to go into a closure, C++11 offers no way to accomplish this. C++14 offers mechanisms to move objects into closures. In order to overcome this limitation in C++11, the standardization commitee provided a new capture mechanism known as *init capture*. Using init capture can do almost everything except default capture modes `[=]` or `[&]`. However, item 31 advise us to stay away from default capture modes.

Syntax of init capture contains:

* The name of a data member generated in the closure class
* An expression for initializing for that data member

Take for example, the following snippet of code:

```c++
#include <iostream>
#include <cstdlib>
#include <utility>
#include <memory>

class Person
{
    public:
      Person(std::string name):_name(name){};
      ~Person(){ std::cout << __PRETTY_FUNCTION__;};
      void printName()
      {
         std::cout << _name << std::endl;
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
    auto pw = std::make_unique<Person>("ABC");
    pw->printName();
    auto func = [lpw = std::move(pw)]                     //init capture. pw is data member. std::move inits pw
    {
        std::cout<<"Entering Lambda"<<std::endl;
        lpw->setName("CBA");
        lpw->printName();
    };
    func();    
    //pw->printName(); //Seg Fault as object is already moved
}
//Output:
ABC
Entering Lambda
CBA
Person::~Person()
```
In the above case, the expression `lpw = std::move(pw)` refers to init capture. The left side of the expression indicates the name of
data member in the closure `lpw` that is generated and the right side indicates initialization of lambda data member `lpw`. Output from cppinsight:
```c++
#include <iostream>
#include <cstdlib>
#include <utility>
#include <memory>

class Person
{ 
  public: 
      inline Person(std::basic_string<char, std::char_traits<char>, std::allocator<char> > name)
              : _name{std::basic_string<char, std::char_traits<char>, std::allocator<char> >(name)}
                  {}
  
      inline ~Person() 
      {
        std::operator<<(std::cout, "Person::~Person()");
      }

      inline void printName()
      {
        std::operator<<(std::cout, this->_name).operator<<(std::endl);
      }

      inline void setName(std::basic_string<char, std::char_traits<char>, std::allocator<char> > name)
      {
        this->_name.operator=(name);
      }

  private: 
      std::basic_string<char, std::char_traits<char>, std::allocator<char> > _name;
      int _age;
  
};

int main()
{
  std::unique_ptr<Person, std::default_delete<Person> > pw = std::make_unique<Person>("ABC");
  pw.operator->()->printName();
    
  class __lambda_28_17
  {
    public: 
        inline /*constexpr */ void operator()() const
        {
          std::operator<<(std::cout, "Entering Lambda").operator<<(std::endl);
          lpw.operator->()->setName(
                                    std::basic_string<char, std::char_traits<char>, std::allocator<char> >
                                        ("CBA", std::allocator<char>()));
          lpw.operator->()->printName();
        }
    
    private: 
        std::unique_ptr<Person, std::default_delete<Person> > lpw;
    public: 
        // inline __lambda_28_17(const __lambda_28_17 &) = delete;
        __lambda_28_17(std::unique_ptr<Person, std::default_delete<Person> > && _lpw) : lpw{std::move(_lpw)} {}    
  };
  
  __lambda_28_17 func = __lambda_28_17{std::unique_ptr<Person, std::default_delete<Person> >(std::move(pw))};
  func.operator()();
}
```
As we see that `lpw` is also an unique pointer, it takes over resource pointed by `pw`. At the end of `main()`, `lpw` is destroyed, its resource destructor is called; whereas `pw` is cleaned up since it is on stack. So, only one destructor of `Person` is called.

## C++11 init capture with `std::bind`
In C++11, you can either write your own class or you can accomplish this by the use of `std::bind`:

Let us try to write the following lambda with C++11
```c++
std::vector<double> data;                       // object to be moved
                                                // into closure    
auto func = [data = std::move(data)] {}   //only C++14
```
The first argument to `std::bind` is a callable object. The subsequent arguments are the values that will be passed to the object.
```c++
auto func = std::bind(                                      // C++11 emulation of init capture
                        [](const std::vector<double>& data) { /* uses of data */ },
                        std::move(data)
                     );
```
So, the whole code looks like this
```c++
#include <iostream>
#include <utility>
#include <vector>
#include <functional>

int main()
{
    std::vector<double> data = {2.0, 2.0};
    auto func = std::bind([](const std::vector<double>& data) { std::cout << data[0]; }, std::move(data));
    func();    
}
\\Output
2
```
Output of cppinsight (C++11):
```c++
#include <iostream>
#include <utility>
#include <vector>
#include <functional>

int main()
{
  std::vector<double> data = std::vector<double, std::allocator<double> >
                                {std::initializer_list<double>{2.0, 2.0}, std::allocator<double>()};
      
  class __lambda_9_27
  {
    public: 
        inline /*constexpr */ void operator()(const std::vector<double, std::allocator<double> > & data) const
        {
          std::cout.operator<<(data.operator[](static_cast<unsigned long>(0)));
        }
    
        using retType_9_27 = void (*)(const std::vector<double> &);
        inline /*constexpr */ operator retType_9_27 () const noexcept
        {
          return __invoke;
        };
    
    private: 
        static inline void __invoke(const std::vector<double, std::allocator<double> > & data)
        {
          std::cout.operator<<(data.operator[](static_cast<unsigned long>(0)));
        }
    
    public: 
        // inline /*constexpr */ __lambda_9_27(__lambda_9_27 &&) noexcept = default;
        // /*constexpr */ __lambda_9_27() = default;
    
  };
  
  std::_Bind<__lambda_9_27> func = std::bind(__lambda_9_27{}, std::move(data));
  func.operator()();
}
```
How does`std::bind` behave? If the argument is an lvalue, then the corresponding argument is copy constructed and if it's an rvalue then
it is move constructed.

Things to remember from this item:
* In C++14, use init captures to move objects in closure.
* In C++11, use `std::bind` to emulate init capture or write your own class which emulates the behavior of the closure class.
