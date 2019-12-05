# Item 41: Consider pass-by-value for copyable parameters that are cheap to move and always copied

For efficiency, a function should copy  lvalue arguments, but move rvalue arguments. Example: `push_back()` function of `std::vector` takes both lvalue and rvalue.

!!!Debug me!!! OR TRY WITH VECTOR OF INTEGER FOR SIMPLIFICATION.

```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
class Widget
{
    public:        
        void addName(std::string newName) 
        { 
            names.push_back(std::move(newName)); 
        }        
        void printNames()
        { 
            for (auto name: names) 
                std::cout << name; 
        }    
        void printAddresses() { 
            for ( auto i : names ) 
                std::cout << &(i) << std::endl; //something is wrong here
                                                //cant get address of each element in vector
        } 
    private:
        std::vector<std::string> names;
};
int main()
{
    Widget w;
    std::string s = "hello ";
    std::cout << "hello's adress " << &s << std::endl;
    w.addName(s);
    w.addName("world!");
    
    w.printNames();    
    std::cout << "\n";
    
    w.printAddresses();
}
```

## Two functions do the same thing

```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
class Widget
{
    public:
        void addName(const std::string& newName) { names.push_back(newName); }         //perform a copy
        void addName(std::string&& newName) { names.push_back(std::move(newName)); }   //perform a move
    
        void printNames(){ for (auto name: names) std::cout << name; }
        void printAddresses() { for (int i = 0; i< names.size(); i++) std::cout << &names[i] << std::endl;}  //todo: re-write this with for range loop
    private:
        std::vector<std::string> names;
};
int main()
{
    Widget w;
    std::string s = "hello ";
    w.addName(s);
    w.addName("world!");
    
    w.printNames();    
    std::cout << "\n";
    std::cout << &s << std::endl;
    w.printAddresses();
}
/*Output:
hello world!
0x7fff8b7c6d30
0x135b180
0x135b1a0
*/
```
We see that two functions doing the same thing but we need to maintain both of them. When two functions are not inline, then we have two functions in the object code.

## Two functions are replaced with one function template taking universal reference

Let us see how does universal reference help in this case:

```c++
template<typename T>
void addName(T&& newName) 
{ 
    names.push_back(std::forward<T>(newName)); 
}        
```
Disavantages:
* As a template, `addName`'s implementation needs to be included in header file - *bloated header file*. It may yields several functions in object code. Example: one for lvalue, one for rvalue, one for `std::string` and compatible types to `std::string` - Item 25
* There are few argument types that can not be passed by universal reference -Item 30
* Improper argument types lead to scary compiler error messages

Let us consider the fact, several instantiations are generated from one function template taking universal reference. Thus, several functions in object code.
I don't know compatible types of `std::string so let's simplify function with integers.
```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
class Widget
{
    public:
        template<typename T>
        void addValue(T&& i) { v.push_back(std::forward<T>(i)); }          
   
    private:
        std::vector<int> v;
};
int main()
{
    Widget w;
    int i = 1;
    long li = 2;  
  	
    w.addValue(i);
    w.addValue(2);
  	w.addValue(li);
}
```
C++ Insight shows three instantiations
```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
class Widget
{
  
  public: 
  template<typename T>
  inline void addValue(T && i)
  {
    push_back(std::forward<T>(i));
  }
  
  /* First instantiated from: insights.cpp:20 */
  #ifdef INSIGHTS_USE_TEMPLATE
  template<>
  inline void addValue<int &>(int & i)
  {
    this->v.push_back(std::forward<int &>(i));
  }
  #endif
  
  
  /* First instantiated from: insights.cpp:21 */
  #ifdef INSIGHTS_USE_TEMPLATE
  template<>
  inline void addValue<int>(int && i)
  {
    this->v.push_back(std::forward<int>(i));
  }
  #endif
  
  
  /* First instantiated from: insights.cpp:22 */
  #ifdef INSIGHTS_USE_TEMPLATE
  template<>
  inline void addValue<long &>(long & i)
  {
    this->v.push_back(static_cast<int>(std::forward<long &>(i)));
  }
  #endif
  
  
  private: 
  std::vector<int, std::allocator<int> > v;
  public: 
  // inline ~Widget() noexcept = default;
  // inline Widget() noexcept = default;
};


int main()
{
  Widget w = Widget();
  int i = 1;
  long li = static_cast<long>(2);
  w.addValue<int &>(i);
  w.addValue<int>(2);
  w.addValue<long &>(li);
}
```
## Pass-by-value function is a good fit in this case

Advantages:
* no *bloated* header file.
* In C++11, it copies on lvalue and moves on rvalue

