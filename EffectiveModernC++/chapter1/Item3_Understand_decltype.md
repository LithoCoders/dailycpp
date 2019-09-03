# `decltype` almost always yields the type of a variable or expression without any modifications
Given a name or an expression, decltype tells you the name’s or the expression’s type.

decltype(name) = *decl*ared *type* of name

Given this:
```c++
#include<vector>
#include<deque>

struct Tuna
{
	int a, b;
};

int main()
{
  const int ci = 0; 
  decltype(ci) dci = 0; 	 // decltype(ci) is const int
  				 // require initialization because of deducing dci as a const  
  Tuna t; 
  decltype(t) dt;  
  bool f(const Tuna & ob);  	 // decltype(ob) const Tuna &                            
                            	 // decltype(f) bool(const Tuna &)  
  decltype(Tuna::a) dta = 10;
  
  std::vector<int> v = {1};
  decltype(v)     dv = {1};	  // decltype(v) is vector<int>
  decltype(v[0]) dv0 = t.a;       // decltype(v[0]) is int&
  
  std::deque<int> d;  		  
  decltype(d) dd;
}
```

Output on C++ Insight shows:
```c++
  const int ci = 0;
  const int dci = 0;
  
  Tuna t = Tuna();
  Tuna dt = Tuna();
  bool f(const Tuna & ob);
  int dta = 10;
  
  std::vector<int> v = 
  	std::vector<int, std::allocator<int> >{std::initializer_list<int>{1}, std::allocator<int>()};
  decltype(v)     dv = 
  	std::vector<int, std::allocator<int> >{std::initializer_list<int>{1}, std::allocator<int>()};
  __gnu_cxx::__alloc_traits<std::allocator<int> >::value_type & dv0 = t.a;
 
  std::deque<int> d = std::deque<int, std::allocator<int> >();
  decltype(d)    dd = std::deque<int, std::allocator<int> >();
```
But exception with vector of bool:
```c++       
  std::vector<bool> vb = {true, false}; 		  //decltype(vb[0]) is bool (!not bool&)
  decltype(vb[0]) dvb = false;
  
  
  error: no viable conversion from 'bool' to 'decltype(vb[0])' (aka 'std::_Bit_reference')
  decltype(vb[0]) dvb = false;
                  ^     ~~~~~
```
This is because decltype of  `vb[0]` is a brand new object `bool` not a reference to `bool` so that you can not assign `dvb` to false as above. This leads to one problem we try to solve a bit later.

**Problem**: write a function that takes a container and an index and return the result of the indexing operation. The return type of the function should be the same as the type returned by the indexing operation, can be T or T&
This is where decltype takes place. In C++11, the primary use for decltype is declaring function templates where the functions’ return type depends on its parameter types.


The use of auto before the function name has nothing to do with type deduction. Rather, it indicates that *C++11’s trailing return type syntax* is being used (function return type will be declared after ->)

```c++
template<typename Container, typename Index>
decltype(auto) f(Container & c, Index i)
{
	retrun c[i];
}

#include <iostream>
#include <utility>
#include <vector>

//C++14
template<typename Container, typename Index>
decltype(auto) doMagic(Container&& c, Index i)
{
	//do something magic then returning indexing operation
  	return std::forward<Container>(c)[i];
}
//C++11
template<typename Container, typename Index>
auto doMoreMagic(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i]) //trailing return type
{
	//do something magic
  	return std::forward<Container>(c)[i];
}

int main()
{
    std::vector<int> v = {1, 2, 3};
    std::vector<bool> vb = {false, false, true};
    std::cout << doMagic(v, 1)  << std::endl;
    std::cout << doMoreMagic(v, 2) <<std::endl;
    std::cout << doMagic(vb, 0) << std::endl;      
}

Start

2
3
0

0

Finish

```
Output on C++ Insight:
```c++
#include <iostream>
#include <utility>
#include <vector>

//C++14
template<typename Container, typename Index>
decltype(auto) doMagic(Container&& c, Index i)
{
	//do something magic
  	return std::forward<Container>(c)[i];
}

/* First instantiated from: insights.cpp:24 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
int & 
	doMagic<std::vector<int, std::allocator<int> > &, int>
		(std::vector<int, std::allocator<int> > & c, int i)
{
  return std::forward<std::vector<int, std::allocator<int> > &>(c)
  			.operator[](static_cast<unsigned long>(i));
}
#endif


/* First instantiated from: insights.cpp:26 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
std::_Bit_reference 
	doMagic<std::vector<bool, std::allocator<bool> > &, int>
		(std::vector<bool, std::allocator<bool> > & c, int i)
{
  return std::forward<std::vector<bool, std::allocator<bool> > &>(c)
  			.operator[](static_cast<unsigned long>(i));
}
#endif

//C++11
template<typename Container, typename Index>
auto doMoreMagic(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i])
{
	//do something magic
  	return std::forward<Container>(c)[i];
}

/* First instantiated from: insights.cpp:25 */
#ifdef INSIGHTS_USE_TEMPLATE
template<>
__gnu_cxx::__alloc_traits<std::allocator<int> >::value_type & 
             doMoreMagic<std::vector<int, std::allocator<int> > &, int>
	              (std::vector<int, std::allocator<int> > & c, int i)
{
  return std::forward<std::vector<int, std::allocator<int> > &>(c)
  			.operator[](static_cast<unsigned long>(i));
}
#endif


int main()
{
  std::vector<int> v = 
  	std::vector<int, std::allocator<int> >
		{std::initializer_list<int>{1, 2, 3}, std::allocator<int>()};
  std::vector<bool> vb = 
  	std::vector<bool, std::allocator<bool> >
		{std::initializer_list<bool>{false, false, true}, std::allocator<bool>()};
		
  std::cout.operator<<(doMagic(v, 1)).operator<<(std::endl);
  std::cout.operator<<(doMoreMagic(v, 2)).operator<<(std::endl);
  std::cout.operator<<(static_cast<bool>(doMagic(vb, 0).operator bool())).operator<<(std::endl);
}

```
It shows doMagic return sometimes `int&` sometimes `bool` or `std::Bit_reference`. This solves the problem above, i.e., returning the same type as indexing operation on container. 

**Things to Remmember:**

• decltype almost always yields the type of a variable or expression without any modifications. 

• ???? For lvalue expressions of type T other than names, decltype always reports a type of T&  ????

• C++14 supports decltype(auto), which, like auto, deduces a type from its initializer, but it performs the type deduction using the decltype rules
