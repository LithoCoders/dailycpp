# auto
Template deduction is about deducting types for functions parameters involving templates. `auto` is a bit different because you do not use auto with templates. Interestingly the difference is almost null becuase auto works more or less the same way template deduction does.


when you delcare `auto a = 33` they keyworkd `auto` is like `T` in a template. 

```c++
auto a = 3;
const auto ca = 3;
const auto& ra = &ca;
```
is equivalent to:

```c++
template<class T>
void f(T a);

template<class T>
void f(const T a);

template<class T>
void f(const T& a);
```



## `auto&&`
```c++
int x = 3;
const int cx = 5;
auto&& a = x //=> what type is a?
auto&& a = 3 //=> what type is a?
auto&& a = cx //=> what type is a?
```

## arrays:

```c++
const char [] name = "blablabla";
auto s = name; //type of s?
auto& rs = name; //type of rs?
```

## functions
```c++
int func(const int a, const double b, ...){}

auto func1 = func; //what type is func1?
auto& func2 = func; //what type is func2?

```

There is one difference when you use initializer lists:
in C++ you can initialize stuff in 4 ways
```c++
auto a = 3;
auto b(22);
auto c = {3};
auto b{22};
```

be careful when you use the `{}` ways with auto because the type can end up being `std::initializer_list<T>`.

################ from ISLA std::initializer_list ###################
#include <cstdio>
#include <initializer_list>

template<typename T> // template with parameter
void f(T param); // declaration equivalent to x's declaration


/*template<typename T>
void f_list(std::initializer_list<T> initList);
*

/*
auto createInitList()
{
return { 1, 2, 3 }; // error: can't deduce type for { 1, 2, 3 }
}

std::vector<int> v;
auto resetV =
[&v](const auto& newValue) { v = newValue; }; // C++14 type for { 1, 2, 3 }
resetV({ 1, 2, 3 });
*/
int main()
{
   //Initialization list with different types
//auto x5 = { 1, 2, 3.0 }; 


//Using Explicit Initialization list for template function call
/*
auto x = { 11, 23, 9 }; // x's type is std::initializer_list<int>
f({ 11, 23, 9 }); 
*/

//f_list({ 11, 23, 9 }); // T deduced as int, and initList's type is std::initializer_list<int>
}


