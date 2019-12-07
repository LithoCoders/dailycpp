There is no strict rules in this chapter. It is depend on your use case. 

# Item 41: Consider pass-by-value for copyable parameters that are cheap to move and always copied

## Pass by value
Nice to re-read item 25. Use std::move on rvalue references and std::forward on universal references.

Let us consider a member function `Widget::addName()` copies its parameter into class's private container `names`. For efficiency, this function should copy lvalue arguments, but move rvalue arguments. Example: `push_back()` function of `std::vector` takes both lvalue and rvalue.

### Two functions do the same thing
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
    
        void printNames()
        { 
            for (auto name: names) 
                std::cout << name; 
        }    
        void printAddresses() { 
            for (unsigned int i = 0; i < names.size(); i++ ) 
                std::cout << &(names[i]) << std::endl;                                                
        } 
    private:
        std::vector<std::string> names;
};
int main()
{
    Widget w;
    std::string s = "hello ";
    std::cout << "local variable's adress " << &s << std::endl;
    w.addName(s);
    w.addName("world!");
    
    w.printNames();    
    std::cout << "\n";
    
    w.printAddresses();
}
/*Output:
local variable's adress 0x7ffed9b12cb0
hello world!
0xafb190
0xafb1b0 */
```
We see that two functions doing the same thing but we need to maintain both of them. When two functions are not inline, then we have two functions in the object code.

### Two functions are replaced with one function template taking universal reference

Let us see how does universal reference can help in this case:

```c++
template<typename T>
void addName(T&& newName) 
{ 
    names.push_back(std::forward<T>(newName)); 
}        
```
Disavantages:
* As a template, `addName`'s implementation needs to be included in header file - *bloated header file*. It may yields several functions in object code. Example: one for lvalue, one for rvalue, one for `std::string` and compatible types to `std::string` - Item 25. See code example below.
* There are few argument types that can not be passed by universal reference -Item 30
* Improper argument types lead to scary compiler error messages

Let us consider the fact, several instantiations are generated from one function template taking universal reference. Thus, several functions in object code.
I don't know compatible types of `std::string` so let's simplify function with integers.
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
  	
    w.addValue(i);  //lvalue
    w.addValue(2);  //rvalue
  	w.addValue(li); //`long`, compatible to `int`
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
### Pass-by-value function is a good fit in this case
As we know from Item 25, applying `std::move` to `rvalue` references but in this case it is fine because:
* (1) modifying `newName` has no impact on `newName` passed by the caller, i.e., `s` variable
* (2) apply `std::move` to `newName` at the end of function `addName()` is safe.

```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
class Widget
{
    public:        
        void addName(std::string newName)         // (1) pass-by-value
        { 
            names.push_back(std::move(newName));  // (2)after this, `newName` has no longer value
        }        
        void printNames()
        { 
            for (auto name: names) 
                std::cout << name; 
        }    
        void printAddresses() { 
            for (unsigned int i = 0; i < names.size(); i++ ) 
                std::cout << &(names[i]) << std::endl;                                                
        } 
    private:
        std::vector<std::string> names;
};
int main()
{
    Widget w;
    std::string s = "hello ";
    std::cout << "local variable's adress " << &s << std::endl;
    w.addName(s);
    w.addName("world!");
    
    w.printNames();    
    std::cout << "\n";
    
    w.printAddresses();
}
/*Output:
local variable's adress 0x7fffeb7dc720
hello world!
0x11c9190
0x11c91b0
*/
```
Advantages:
* no *bloated* header file.
* It copies on lvalue and moves on rvalue (As `std::move` exists in C++11)

### Recap on three approaches:
```c++
//Approach 1: Overloading. Cost = one copy for lvalues & one move for rvalues
        void addName(const std::string& newName) { names.push_back(newName); }         //perform a copy
        void addName(std::string&& newName) { names.push_back(std::move(newName)); }   //perform a move
        
//Approach 2: Universal reference. Cost = one copy for lvalues & one move for rvalues (same to Approach 1)
        template<typename T>
        void addValue(T&& i) { v.push_back(std::forward<T>(i)); }       
        
//Approach 3: Pass-by-value. Cost = copy ctor + move for lvalues & move ctor + move for rvalue
        void addName(std::string newName)         
        { 
            names.push_back(std::move(newName));  
        }        
```
Let us consider the title of this item: "*Consider* pass-by-value for *copyable parameters* that are *cheap to move* and *always copied*"
1. It contains word *consider* so it is not a rule. 
* Advantages: one function, it generates one function in object code, avoid issues with universal reference. 
* Disadvantages: higher cost.
2. Consider pass-by value for only *copyable parameters*. Unlike this, `std::unique_ptr` is not copyable.
Pass-by value is a bad approach. Cost = 2 moves = move ctor + `std::move`
```c++
#include <string>
#include <memory>
#include <utility>
#include <iostream>
class Widget
{
    public:        
        void setPtr(std::unique_ptr<std::string> ptr) //pass by value
            { p = std::move(ptr);}
    private:
        std::unique_ptr<std::string> p;
};
int main()
{
    Widget w;
    w.setPtr(std::make_unique<std::string>("C++"));
}
```
Overloading is a good approach. Cost = one move = `std::move`
```c++
#include <string>
#include <memory>
#include <utility>
#include <iostream>
class Widget
{
    public:        
        void setPtr(std::unique_ptr<std::string>&& ptr) //pass by reference
            { p = std::move(ptr);}
    private:
        std::unique_ptr<std::string> p;
};
int main()
{
    Widget w;
    w.setPtr(std::make_unique<std::string>("C++"));
}
```
3. *cheap to move*. When it is not cheap, a move can cost like a copy.
4. *always copied*. Some use case, some logic can prevent 100% copying, e.g, conditional copying.
This is bad because it costs one copy ctor but then destroy this object and nothing is added to `names`. 
Better approach is pass by references.
```c++
#include <string>
#include <vector>
#include <utility>
#include <iostream>
const int MIN =  1;
const int MAX = 10;
class Widget
{
    public:        
        void addName(std::string newName) //pass by value
        {
            if ( (newName.length() >= MIN) && (newName.length() <= MAX) ) //Bad: conditional copy
                names.push_back(std::move(newName)); 
        }        
        
    private:
        std::vector<std::string> names;
};
int main()
{
    Widget w;
    std::string s = "hello ";    
    w.addName(s);
    w.addName("world!");    
}
```
## lvalue auguments
## Slicing problem

