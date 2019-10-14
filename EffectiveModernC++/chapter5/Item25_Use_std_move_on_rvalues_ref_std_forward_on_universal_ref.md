**Rules**: Use `std::move` on rvalue references and `std::forward` on universal references.

"Coding, many quotes from book Effective Modern C++"

# Try to not follow the rules
Ravlue references bind to object to be moved whereas universal references might bind to object that eligible to be moved.

That is the rule, let us try to not follow the rule. Let us apply `std::forward` with rvalue reference and `std::move` with universal reference.

bad code: std::forward on rvalue reference - Make it up.

bad code 1: `std::move` on universal reference
```c++
#include <utility>
#include <string>
#include <iostream>

class Widget {
    public:  
        template<typename T>  void setNameByMove(T&& newName)         // universal reference  
        { 
            name = std::move(newName);                                // Compile, but it is BAD!!!
        }
        template<typename T>  void setNameByForward(T&& newName)      // universal reference  
        { 
            name = std::forward<T>(newName);                          // Correct
        }
        std::string getName()
        {  
            return name;
        }
    private:  
        std::string name;          
};

std::string getWidgetName() 
{
    return "foo";
}
int main()
{
    Widget w;

    auto name1 = getWidgetName();  //name1 is lvalue   
    w.setNameByForward(name1);     //name1 still there
    std::cout << w.getName() << " -- " << name1 << std::endl;
    
    auto name2 = getWidgetName();  //name2 is lvalue
    w.setNameByMove(name2);        //name2 is not there anymore           
    std::cout << w.getName() << " -- " << name2 << std::endl;
}
Start

foo -- foo
foo -- 

0

Finish
```
In both cases, we have universal references deduced to lvalue references because its parameter is an lvalue `name1` and `name2`. 
But after `setNameByMove`, `name2` is empty.
C++ Insights shows universal references deduced to lvalue references.

```c++
template<>
inline void setNameByMove<
                std::basic_string<char, std::char_traits<char>, std::allocator<char> > &>
                   (std::basic_string<char, std::char_traits<char>, std::allocator<char> > & newName)  //lvalue reference
{
  this->name.operator=(std::move(newName));
}

template<>
inline void setNameByForward<
                std::basic_string<char, std::char_traits<char>, std::allocator<char> > &>
                    (std::basic_string<char, std::char_traits<char>, std::allocator<char> > & newName)  //lvalue reference
{
  this->name.operator=
    (std::forward<std::basic_string<char, std::char_traits<char>, std::allocator<char> > &>(newName));
}  
```
If we don't use function template, we need to replace this template function with two overloading functions, one taking lvalues and the other one taking rvalue.

```c++
class Widget { 
    public:  
        void setName(const std::string& newName)      // const lvalue
        { name = newName; }
        void setName(std::string&& newName)           // rvalue
        { name = std::move(newName); } 
  };
```
Bad 1: With no function template, the code is lengthy and not efficient. Suppose, you need to call `w.setName("ABC")`. This creates a constructor for a temporary string `"ABC"`, a move, and a destructor. Not efficient.

Bad 2: Imagine you have a function taking n parameters, each of them can be an lvalue or an rvalue. Applying the above approach would produce you a 2^n combinations. You can not have 2^n overloading functions.

Thus, universal reference is the only way to go. As a result, you need `std::forward`.

# Follow the rules

There are more rules...

## Apply `std::move` and `std::forward` at the end of usage
Because you don't want to have unspecified value when you are not done with rvalue references and universal references
```c++
template<typename T>                       // text is 
void setSignText(T&& text)                 // univ. reference 
{  
    sign.setText(text);                              // use text, but don't modify it
    auto now = std::chrono::system_clock::now();     // get current time    

    signHistory.add(now, std::forward<T>(text));  // conditionally cast text to rvalue 
}                                      
```  

## With function returning a value
Object bound to rvalue reference or universal reference, apply `std::move` or `std::forward` correspondingly.

Example 1: Matrix adding. Efficient code shows moving instead of copying
```c++
Matrix operator+(Matrix&& lhs, const Matrix&& rhs) //return by-value
{
  lhs += rhs;
  return std::move(lhs);    //move lhs into return value
}

//less efficient
Matrix operator+(Matrix&& lhs, const Matrix&& rhs) //return by-value
{
  lhs += rhs;
  return lhs;    //copy lhs into return value
}
```
Example 2: Fraction. 
```c++
template<typename T> Fraction reduceAndCopy(T&& frac) // return a Fraction object
{
    fraction.reduce;
    return std::forward<T>(frac);   // if frac is an lvalue -> copy
                                    // if frac is an rvalue -> move
}

```

## RVO - return value optimization.
No applying `std::move` and `std::forward` for local variables.

RVO: constructing `w` in the memory alloted for the function's return value.
Two condition to have RVO happens:
* type of local object is the same as that returned by the function
* local object is returned

```c++
Widget makeWidget()
{
    Widget w;
    //do something
    return w;
}
```
Don't introduce `std::move` in this code. Because it is no longer qualified for RVO
```c++
Widget makeWidget()
{
    Widget w;
    //do something
    return std::move(w); //don't do this
}
```

The same with by-value function parameters
```c++
Widget makeWidget(Widget w)
{
    //do something
    return w;
}
```
Don't introduce `std::move` in this code.
```c++
Widget makeWidget(Widget w)
{
    //do something
    return std::move(w); //don't
}
```
