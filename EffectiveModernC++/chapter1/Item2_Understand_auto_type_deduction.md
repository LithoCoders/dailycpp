# auto
Template deduction is about deducting types for functions parameters involving templates. `auto` is a bit different because you do not use auto with templates. Interestingly the difference is almost null because auto works more or less the same way template deduction does.


when you delcare `auto a = 33` the keyworkd `auto` is like `T` in a template. 

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
	int x = 1;
  	const int cx = 2;
  	const int& rx = 3;
	int&& ux = 2;
  
  	auto&& a3 = 3;          //what is type of a3 ?
   	auto&& ax = x;		//what is type of ax ?
	auto&& acx = cx; 	//what is type of acx ?
  	auto&& arx = rx;  	//what is type of arx ?
	auto&& aux = ux;        //what is type of aux ?
```
Output on C++ Insights shows that rules from Item1.Case2 work as expected

```c+++
	int x = 1;
  	const int cx = 2;
  	const int & rx = 3;
  	int && ux = 2;
	
	int && a3 = 3;          //3 is a rvalue thus a3 is also a3 rvalue
  	int & ax = x;		//x is a lvalue thus ax is also ax lvalue
  	const int & acx = cx;	//cx is a lvalue thus acx is a lvalue, `const` from patern matching
  	const int & arx = rx;	//rx is a lvalue, arx is a lvalue
	int & aux = ux;         //why ? ux is a rvalue ?
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

be careful when you use the `{}` ways with auto because the type can end up being `std::initializer_list<T>`. For examples (from ISLA):
example 1: `auto` deduced to `std::initializer_list` of integer. 

```c++
#include <initializer_list>

template<typename T> 
void f(T param); 

template<typename T>
void f_list(std::initializer_list<T> initList);

int main()
{
	auto x = { 11, 23, 9 };
  	f(x);
   f_list(x);  	
}

```
Output on C++ Insight shows

```c++
#include <initializer_list>

template<typename T> 
void f(T param); 

template<typename T>
void f_list(std::initializer_list<T> initList);

int main()
{
  std::initializer_list<int> x = std::initializer_list<int>{11, 23, 9};   // auto is deduced to std::initializer_list<int>
  f(std::initializer_list<int>(x));                                       // ParamType and T also std::initializer_list<int>     
  f_list(std::initializer_list<int>(x));                                  // ParamType: std::initializer_list and T: int 
}
```
example 2: Passing initializer_list to function template does not work

```c++
#include <initializer_list>

template<typename T> 
void f(T param); 

template<typename T>
void f_list(std::initializer_list<T> initList);

int main()
{
  	f({ 11, 23, 9 }); 
}

/home/insights/insights.cpp:4:6: note: candidate template ignored: couldn't infer template argument 'T'
void f(T param); 
     ^
1 error generated.
Error while processing /home/insights/insights.cpp.
```
example 3: f_list() just works.


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


